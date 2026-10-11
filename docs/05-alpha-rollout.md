# 05 — Alpha Rollout

Build the engine first, prove it is correct, then add the app and the AI. Each phase has a gate; we don't move on until it passes.

## Phases

| Phase | Deliverable | Gate to move on |
|---|---|---|
| **0. Docs** | These specs agreed | Owner sign-off; licence + validation partner decided |
| **1. Engine** | KP engine + CLI, golden test set (50 charts) | 100% of charts within tolerance (engine spec §8.1); all lords identical |
| **2. App shell** | VPS, invite links, admin panel, profile form, place search, North Indian charts, KP tables | Owner + astrologer create profiles and confirm charts match reference software |
| **3. AI agent** | Question pipeline, Hinglish/English answers, guardrails, daily guide | Eval set passes: 0 blocked-topic leaks, validator 100%, astrologer approves sample |
| **4. Closed test** | ~10 users (owner, astrologer, close family) for 1 week | No critical bugs; ≥ 70% 👍 |
| **5. Alpha** | Invites in batches of ~25, up to ~100 users, over a few weeks | Success metrics below hold for 2 consecutive weeks |

## Invite flow

1. Owner creates an invite in `/admin` with a label (e.g. "Rahul — cousin").
2. Shares the link on WhatsApp with a short note (template in admin panel).
3. User opens link → language choice → consent screen → first profile → chart → "Ask anything".

## Feedback loop

- 👍 / 👎 + optional comment on every answer; "Ye galat hai" report button.
- Weekly review: owner + astrologer go through all 👎, reports and flagged answers.
- Each issue is tagged: `engine` (calculation/rule), `agent` (wording/interpretation), `input` (wrong birth data), `ux`.
- Fixes ship as new engine/prompt versions; golden tests and eval set must still pass.

## Metrics (tracked in admin panel)

| Metric | Target |
|---|---|
| Answers rated 👍 | ≥ 70% |
| Astrologer agreement on weekly sample | ≥ 80% |
| Hard-limit violations | 0 |
| Validator failures reaching users | 0 |
| Weekly active users / invited | ≥ 50% |
| Median answer time | < 10 s |

## Before any public release (later)

Real signup + abuse protection, DPDP compliance review, terms & disclaimers reviewed by a lawyer, cost model per user, load test, and a decision on adding further systems or KP horary.
