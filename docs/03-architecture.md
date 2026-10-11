# 03 — Architecture

One VPS, one Zig binary, one SQLite file. Simple to run, simple to back up.

## 1. Overview

```
 Browser (PWA)
     │ HTTPS
     ▼
 Caddy (TLS for ribhu.co, auto Let's Encrypt)
     │ HTTP (localhost)
     ▼
 ribhu (single Zig binary)
  ├─ web: routes, sessions, invite links, static PWA assets (embedded)
  ├─ admin: invite management, conversation review, feedback
  ├─ engine: KP calculations (Swiss Ephemeris C lib linked in)
  ├─ agent: question pipeline, LLM calls, answer checks
  └─ db: SQLite (WAL mode)
     │
     ├─ ribhu.db          (all app data)
     ├─ ephe/*.se1        (ephemeris data files)
     └─ outbound HTTPS → LLM provider API
```

## 2. Tech choices

| Part | Choice | Why |
|---|---|---|
| Language | **Zig** (version pinned at project start) | Calls C libraries (Swiss Ephemeris, SQLite) directly, one small static binary, memory safety checks in debug builds |
| Database | **SQLite** (amalgamation compiled in), WAL mode | Zero ops; easily handles 100s of users |
| Ephemeris | Swiss Ephemeris (C) | Gold standard accuracy; licence: see README open items |
| TLS / proxy | Caddy | Automatic HTTPS; Zig's standard library has no TLS server |
| Frontend | Plain HTML + CSS + small vanilla JS, Web App Manifest + Service Worker | Installable PWA, no build tooling, fast on cheap phones |
| Charts | SVG rendered server-side by Zig | Same image on every device; no chart JS library |
| AI | Provider-agnostic adapter over HTTPS | Swap models without touching the engine |
| Backups | Litestream (continuous) to S3-compatible storage + nightly snapshot | Point-in-time restore |
| Hosting | VPS **in India** (e.g. Mumbai), 2 vCPU / 4 GB RAM | Low latency for users; data stays in India |

## 3. Codebase layout (planned)

```
ribhu/
├─ build.zig
├─ src/
│  ├─ main.zig
│  ├─ engine/      # pure KP library — no I/O, no allocation surprises
│  ├─ web/         # http routes, sessions, templates
│  ├─ admin/
│  ├─ agent/       # pipeline, prompts, LLM adapter, answer validator
│  └─ db/          # schema, migrations, queries
├─ web/            # PWA static files (embedded at build time)
├─ data/           # topic table, remedy catalog, prompt templates (versioned)
├─ vendor/         # swisseph, sqlite sources
├─ tests/golden/   # reference charts + expected outputs
└─ docs/
```

`engine/` also builds as a small CLI (`ribhu-engine chart --date ... --time ... --lat ... --lon ...`) for validation against reference software.

## 4. Access: invite links

No signup. The admin creates an invite; the user opens a link.

1. Admin creates invite → server generates a random 128-bit token.
2. Link: `https://ribhu.co/i/<token>` (shared via WhatsApp etc.).
3. Opening it creates a session (secure, http-only cookie, long-lived) and redirects to the app. The link keeps working on other devices.
4. Only a **hash** of the token is stored. If a link leaks, the admin can **revoke** it and issue a new one; all sessions for it end.
5. Admin panel shows each device/session that used the link.

### Admin panel (`/admin`)

- Login: password (Argon2-hashed) + TOTP code; owner only.
- Create / label / revoke / re-issue invites; set max profiles per invite.
- View usage, feedback, flagged answers, and conversations (users are told about this in the alpha consent screen).
- Export feedback as CSV.

## 5. Data model (SQLite)

| Table | Key columns |
|---|---|
| `invites` | id, token_hash, label, max_profiles, created_at, revoked_at |
| `sessions` | id, invite_id, token_hash, user_agent, created_at, last_seen_at |
| `users` | id, invite_id, language (`hinglish`/`en`), current_place_id, consent_at |
| `profiles` | id, user_id, relation, name, gender, birth_local, place_id, lat, lon, tz_id, utc_offset_used, birth_utc, time_confirmed, created_at |
| `charts` | profile_id, engine_version, chart_json, computed_at *(cache; can always be recomputed)* |
| `places` | GeoNames rows: id, name, admin1, country, lat, lon, tz_id, population |
| `conversations` | id, user_id, profile_id, created_at |
| `messages` | id, conversation_id, role, content, language, topic, engine_version, facts_json, model, tokens_in, tokens_out, latency_ms, created_at |
| `feedback` | message_id, rating, comment, reported_wrong, created_at |
| `daily_cache` | profile_id, date, place_id, payload_json |
| `audit_log` | actor, action, target, created_at |

Data deletion: "Delete my data" removes the user's profiles, charts, conversations, messages and feedback; the invite is marked used-and-deleted.

## 6. Request flows

**Create profile:** form → place search (`/api/places?q=`) → review screen → `POST /api/profiles` → engine `chart()` → stored → chart page.

**Ask a question:** `POST /api/ask` → agent pipeline (AI spec §3) → answer streamed back to the browser (Server-Sent Events) → stored with facts and engine version.

**Daily guide:** first open of the day → engine `daily()` for the user's current city → short AI summary → cached for the day.

## 7. Operations

- `systemd` service; deploy = copy new binary + restart (a few seconds downtime is fine for alpha).
- Structured logs to journald; no birth data or chat text in logs.
- Health endpoint `/healthz`; simple uptime check.
- Rate limits: per session on `/api/ask` (e.g. 20 questions/day, configurable) to control AI cost and abuse.
- Secrets (LLM API key, admin TOTP seed) in an env file readable only by the service user.

## 8. Privacy & security

- Birth data is personal data under India's **DPDP Act**: consent screen on first use, clear purpose, user-initiated deletion, data hosted in India.
- LLM requests contain chart facts and the question, **not** the user's name, exact birth date/time or place.
- HTTPS only, HSTS, secure cookies, CSRF protection on all POSTs, strict Content-Security-Policy.
