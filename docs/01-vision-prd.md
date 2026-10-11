# 01 — Vision & PRD

## 1. Vision

Ribhu is a personal KP astrology guide. It gives readings that are **precise, traceable and consistent**: the same birth data always produces the same chart, and every answer can show *which chart factors* it is based on.

We are **not** building an "AI astrologer wrapper". The AI never calculates anything. It reads the engine's verdict and explains it.

## 2. Principles

1. **Engine decides, AI explains.** Positions, lords, significators, dasha dates and verdicts come from deterministic code. The AI may only phrase them.
2. **KP only, done properly.** One system, mastered, before adding anything else.
3. **No guessing on input.** Wrong birth time = wrong KP chart. We ask for exact data and refuse to proceed without it.
4. **Show the "kyun".** Every answer has an expandable "Why?" section with the KP factors behind it.
5. **Sacred space, safe space.** Respectful tone, no fear-mongering, hard limits on death, serious illness and legal outcomes.

## 3. Users (alpha)

- ~100 people from the owner's inner circle, invite only.
- Mostly Indian, comfortable with Hinglish, familiar with basic terms (rashi, nakshatra, dasha).
- Many will also check charts for family members.

## 4. Alpha scope

### In scope

| Area | What the user gets |
|---|---|
| **Access** | Opens a personal invite link — no signup, no password |
| **Birth profile** | Enters name, gender, date, exact time, birth place. Confirms on a review screen. |
| **Family profiles** | Up to **5 profiles** per invite (self + family). Default, adjustable per invite by admin. |
| **Birth chart** | North Indian chart (Rashi + KP Bhava/cuspal), KP tables (planets & cusps with sign/star/sub/sub-sub lords), significators, Vimshottari dasha timeline |
| **Profile summary** | Lagna, Moon sign, nakshatra, current dasha, key life-area highlights |
| **Ask a question** | Chat; answers based on the birth chart + current dasha + transits + ruling planets at the time of asking |
| **Aaj ka din (daily guide)** | Panchang, Rahu Kaal / Choghadiya / Hora for the user's current city, plus a short personal note for the day. In-app only (no push notifications in alpha). |
| **Remedies** | Mantra, puja, daan, vrat, gemstone suggestions — only where the engine marks them relevant |
| **Language** | Hinglish (default) or English, switchable anytime |
| **Feedback** | 👍 / 👎 + comment on every answer; "this is wrong" report |
| **Privacy** | Consent screen; user can delete all their data |

### Out of scope (alpha)

Parashari / other systems, KP horary (prashna), birth-time rectification, kundli matching, Hindi script / regional languages, payments, public signup, push notifications, native apps.

## 5. Birth data rules

- **Required:** name, gender, date of birth, time of birth (to the minute), birth place.
- The form asks: *"Is this time from a birth certificate / hospital record / family record you trust?"* If the user says they are not sure, we explain why KP needs an exact time and **do not create the chart**.
- Review screen repeats the input in plain words to catch common mistakes:
  - AM/PM: "Subah 6:15 baje (6:15 AM)"
  - Night births: "Raat 2:10 baje, yaani **6 March** ki subah" — because families often record the previous date for births after midnight.
- Births in India before 1956 trigger an extra question about which local time standard the record used (old Bombay / Calcutta time). See engine spec §3.

## 6. Hard limits

The assistant **never**:
- predicts death, longevity, or "maraka" timing for anyone;
- predicts or names serious illness (cancer, heart attack, etc.) or gives a diagnosis;
- predicts legal outcomes (court cases, arrests, police matters, jail);
- predicts the sex of an unborn child (illegal in India under the PCPNDT Act).

It **may** give general health caution ("is period mein health pe dhyan rakhein, routine checkup karwa lein") and always points to a doctor / lawyer for real concerns. Full policy: AI spec §5.

## 7. Success for alpha

- Engine matches reference software on every validation chart (see engine spec §8).
- ≥ 70% of rated answers get 👍.
- KP astrologer agrees with ≥ 80% of a sampled set of answers.
- Zero hard-limit violations.
