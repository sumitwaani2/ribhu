# Ribhu — ribhu.co

A KP (Krishnamurti Paddhati) astrology assistant for Indian users.
A **deterministic engine** computes the chart; an **AI agent** only explains the engine's output in plain Hinglish or English.

> Status: planning. Alpha will be invite-only (~100 users from the inner circle) for a few weeks before any public release.

## Docs

| # | Doc | What it covers |
|---|---|---|
| 1 | [Vision & PRD](docs/01-vision-prd.md) | What we build, for whom, alpha scope |
| 2 | [KP Engine Spec](docs/02-kp-engine-spec.md) | Calculations, KP rules, add-ons, accuracy targets, validation |
| 3 | [Architecture](docs/03-architecture.md) | Zig + SQLite + single VPS, data model, invite links, PWA |
| 4 | [AI Agent Spec](docs/04-ai-agent-spec.md) | How the AI uses engine facts, language, guardrails |
| 5 | [Alpha Rollout](docs/05-alpha-rollout.md) | Phases, invite flow, feedback, go/no-go gates |

## Decision log

| Decision | Choice |
|---|---|
| Astrology system | **KP only** for alpha. No Parashari pipeline, no horary, no fallbacks. |
| Add-ons on top of KP | Panchang & daily muhurat timings, Sade Sati flag, remedy catalog (see engine spec §6) |
| Ayanamsa | **KP New** only |
| House system | Placidus (standard KP) |
| Chart style | **North Indian** only |
| Birth data | Exact data only. If the user is unsure of birth time, we do not generate a chart. |
| Languages | **Hinglish (default)** and English, with a selector. No Hindi script or regional languages. |
| Hard-blocked topics | Death / longevity, serious illness, legal outcomes |
| Allowed | General health caution, remedies incl. puja, mantra, daan, vrat, gemstones (platform sells nothing) |
| Access | Invite links created from an admin panel; no signup |
| Stack | Zig + SQLite + one VPS (monolith), PWA frontend |

## Open items (need owner input)

1. **Swiss Ephemeris licence** — recommended: buy the Professional licence (CHF 700, one-time) so the code can stay private. Needed before the first external user. ([price list](https://www.astro.com/swisseph/swephprice_e.htm))
2. **Validation partner** — a KP astrologer + reference software (e.g. KP Star One) to compare against.
3. **AI provider** — design is provider-agnostic; pick one before Phase 3.
4. **Timeline** — no fixed date yet; see phases in the rollout doc.
