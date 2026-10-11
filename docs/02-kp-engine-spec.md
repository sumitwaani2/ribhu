# 02 — KP Engine Spec

The engine is the product. It is a pure, deterministic library: **same input + same engine version → byte-identical output**. It has no AI, no network, and no randomness.

## 1. Inputs

| Input | Notes |
|---|---|
| Birth date & time | Local civil time, to the minute (seconds optional) |
| Birth place | Latitude, longitude, IANA time zone (from offline place DB) |
| UTC offset used | Resolved by the engine (§3), stored with the profile, confirmed by the user when ambiguous |
| "Now" context | Moment + user's current location — for ruling planets, transits, panchang |

## 2. Ephemeris & core astronomy

- **Swiss Ephemeris** (C library, linked into the Zig binary) with its compressed JPL-based data files, covering roughly 1800–2400.
- Bodies: Sun, Moon, Mars, Mercury, Jupiter, Venus, Saturn, Rahu, Ketu (= Rahu + 180°). No outer planets in readings for alpha.
- **Node type: mean vs true** — configurable; default fixed after comparing with the chosen reference software (open item).
- Geocentric, apparent positions; ΔT handled by Swiss Ephemeris.
- Retrograde status and speed recorded for every planet.

### Ayanamsa: KP New

Swiss Ephemeris has no exact built-in "KP New", so the engine computes it itself and subtracts it from tropical longitudes:

```
NKPA(year) = 22°22'30" + (T × 50.2388475" + T² × 0.000111") / 3600,  T = year − 1900
reference moment: 15 April, 05:30 IST each year
```

(Formula by Prof. K. Balachandran, *KP & Astrology Year Book 2003*, as [discussed on the Swiss Ephemeris list](https://groups.io/g/swisseph/topic/new_kp_ayanamsa/34114003).) The engine interpolates continuously between yearly values. **This must be verified against the reference software to within 1″ before anything else is built on it.**

### Houses

- **Placidus**, sidereal (cusp longitude − ayanamsa), 12 cusps.
- A planet belongs to a house from that house's cusp up to the next cusp (KP rule; no 5° orb).
- High latitudes where Placidus fails are out of scope (not relevant for India); engine returns an explicit error.

## 3. Time & place correctness

- **Place DB:** GeoNames (cities ≥ 1000 population, worldwide, CC-BY) stored in SQLite, searchable offline; gives lat/long + IANA zone.
- **Time zones:** IANA tzdata (historical rules included, e.g. India's 1942–45 war-time offset).
- **Pre-1956 India:** tzdata uses one zone for all of India, but Bombay Time (≈ UTC+4:51) and Calcutta Time (≈ UTC+5:54) were used locally until 1955 / 1948. For births before 1956 in those regions, the app asks which standard the time was recorded in and stores the answer.
- Final UTC moment and offset are stored with the profile, so a later tzdata update never silently changes a chart.

## 4. KP calculations (core)

### 4.1 Star / sub / sub-sub lords

- 27 nakshatras × 13°20'; each split into 9 subs proportional to Vimshottari years (Ke 7, Ve 20, Su 6, Mo 10, Ma 7, Ra 18, Ju 16, Sa 19, Me 17 = 120), starting from the nakshatra lord.
- Subs crossing sign boundaries are split → the standard **249** KP divisions.
- Sub-subs: same proportional split inside each sub.
- For **every planet and every cusp**: sign, sign lord, nakshatra, star lord, sub lord, sub-sub lord, degree in sign (DMS).

### 4.2 Significators (4-level, KSK order of strength)

For each house H:
1. Planets in the star of occupants of H (strongest)
2. Occupants of H
3. Planets in the star of the owner of H
4. Owner of H

Additional rules:
- A planet with **no planet in its star** gives results strongly of its own house-ownership / occupation.
- **Rahu / Ketu** act as agents: they also signify what their sign lord, star lord, and conjoined planets signify.
- Planet-wise and house-wise significator tables are both stored.
- Optional strength modifiers (configurable, off by default until validated): retrograde star lord, combustion, a planet in the star of a retrograde planet.

### 4.3 Cuspal sub-lord (CSL) promise

For each life topic, the sub-lord of the primary cusp decides whether the matter is promised:
- The CSL's **star lord** shows *what* the CSL signifies; the CSL itself shows whether it is favourable.
- **Promised** if it signifies the topic's houses; **denied / obstructed** if it mainly signifies the negating houses (12th from each topic house); **mixed** otherwise.
- Output: `promised | mixed | denied` + strength score + the exact chain of factors used.

### 4.4 Topic table (initial, to be confirmed with the KP astrologer)

| Topic | Primary cusp | Favourable houses |
|---|---|---|
| Marriage | 7 | 2, 7, 11 |
| Love relationship | 5 | 5, 11 (+7) |
| Job / service | 10 | 2, 6, 10, 11 |
| Promotion | 10 | 10, 11 (+2, 6) |
| Business | 7 / 10 | 7, 10, 11 (+2) |
| Money / gains | 11 | 2, 6, 11 |
| Education | 4 / 9 | 4, 9, 11 |
| Childbirth | 5 | 2, 5, 11 |
| Property / house | 4 | 4, 11, 12 |
| Vehicle | 4 | 4, 11 |
| Foreign travel / settlement | 12 / 9 | 3, 9, 12 |
| General health caution | 1 / 6 | (caution only, see AI spec §5) |

Negating houses are computed by the 12th-from rule. The table is data (stored in SQLite / config), not hard-coded, so the astrologer can tune it without code changes.

### 4.5 Vimshottari dasha

- Balance at birth from the Moon's exact position in its nakshatra.
- Levels: Mahadasha → Bhukti → Antara → Sookshma → Prana.
- Year length: configurable (365.25 days default); confirm against reference software.

### 4.6 Ruling planets (RP)

At any moment & location: day lord (Vedic day, sunrise to sunrise), Moon sign lord, Moon star lord, Lagna sign lord, Lagna star lord (+ Moon and Lagna sub lords as a second tier). Rahu/Ketu included when they represent an RP. Computed **at the time the user asks** and at their current location.

### 4.7 Timing

For a promised topic:
1. Take the significators of the topic houses.
2. Filter by current ruling planets (KP elimination).
3. Find upcoming Dasha-Bhukti-Antara periods ruled by the remaining significators → **time windows**.
4. Confirm with transits (Sun, Moon, Jupiter, Saturn through signs/stars/subs of the significators).

Output: ranked windows with start/end dates, a confidence level (`strong | moderate | weak`), and the factors used.

### 4.8 Transits

Daily positions of all planets with their star/sub lords; which natal houses they are transiting (by cusp); which natal significators they activate.

### 4.9 Birth-time sensitivity flag

Not rectification — we already require exact time. The engine computes how many minutes the birth time could move before the **Lagna, 7th, 10th, 11th cusp sub-lords** change. If under ±2 minutes, verdicts on those topics are marked `edge` and the AI softens its confidence wording.

## 5. Chart output (North Indian)

- **Rashi chart** (sign-based) and **Bhava chart** (KP cuspal, Placidus) in North Indian diamond layout, drawn as SVG by the server — identical on every device.
- Intercepted signs / two cusps in one sign are shown in the cusp table (the diamond cannot show them).
- Tables: planets, cusps, significators (house-wise and planet-wise), ruling planets, dasha tree.

## 6. Add-ons (outside classical KP, included because they add value)

| Add-on | Why | Used for |
|---|---|---|
| **Panchang** — tithi, vara, nakshatra, yoga, karana, sunrise/sunset, moonrise | Indian users plan their day by it | Daily guide |
| **Muhurat timings** — Rahu Kaal, Yamaganda, Gulika, Abhijit, Choghadiya, Hora | Common day-to-day questions ("kab shuru karun?") | Daily guide, "best time today" answers |
| **Sade Sati / Saturn transit flag** | Very commonly asked; users expect it | Informational; *judged through KP* (Saturn's significations), not Parashari rules |
| **Remedy catalog** — per planet: beej mantra, deity, day, daan items, vrat, colour, gemstone | KP has little remedy guidance; users want actionable steps | Suggested only for planets the engine marks as **favourable significators** of the asked topic. Never a gemstone for a planet that signifies the negating houses. |
| **Basic identity facts** — Lagna, Moon sign, nakshatra & pada | "Meri rashi kya hai?" | Profile summary |

All add-ons are pure calculations or data lookups; they follow the same determinism rules.

## 7. Engine API (internal)

The engine exposes plain functions; the web server and AI layer call them. All return JSON with an `engine_version` field.

| Function | Returns |
|---|---|
| `chart(birth)` | Positions, cusps, lords, significators, dasha tree, sensitivity flags |
| `judge_topic(birth, topic, now)` | Promise verdict, strength, timing windows, factors |
| `ruling_planets(moment, place)` | RP set |
| `transits(birth, date)` | Transit positions + activated significators |
| `daily(birth, date, place)` | Panchang, muhurat timings, personal day factors |
| `remedies(birth, topic)` | Eligible planets + catalog entries |

## 8. Accuracy targets & validation

### 8.1 Numerical (vs reference software, same settings)

| Item | Tolerance |
|---|---|
| Ayanamsa | ≤ 1″ |
| Planet longitudes | ≤ 2″ |
| Cusp longitudes | ≤ 5″ |
| Sign / star / sub / sub-sub lords (planets & cusps) | **identical** |
| Dasha dates (MD/AD/PD) | ≤ 1 day |
| Sunrise / sunset | ≤ 1 minute |

### 8.2 Golden test set

- **50 charts** (from the KP astrologer + inner circle, with consent), including edge cases: night births, pre-1956 births, births near sign/star/sub boundaries, births abroad.
- Reference outputs exported from the reference software and committed as test fixtures. Every engine change must pass them all.

### 8.3 Prediction back-testing

- 20–50 of those charts come with **known life events** (marriage, first job, childbirth, relocation, property) and dates.
- For each event: was it promised by the CSL? Did a top-ranked timing window contain the event date?
- Track hit rate per topic. Rule changes are accepted only if they don't lower it.

### 8.4 Versioning

`engine_version` is stored with every chart and answer. Changing rules or tables bumps the version, so any old answer can be reproduced exactly.
