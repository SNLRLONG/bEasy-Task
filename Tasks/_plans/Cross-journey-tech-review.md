# Customer Journey tech review: fixes and improvements

- **Sources:** `Tasks/_inbox/bEasy_Customer_Journey_Tracking_Tech_Review_18Oct2026.docx` (requirements) · `Docs/tracking/v2/archive/Journey_Tech_Review_Handoff_19Oct2026.md` (backend, branch `fix/journey-data`)
- **Planned:** 2026-10-19 · **Owner:** SNLRLONG (Seavlong)
- **Code base:** `origin/feat/journey-cms` (PR #480 merged `feat/journey-list-fixes`; the local `bEasy-portal/` checkout is 2 commits behind, update it before creating worktrees)
- **Caveat:** `bEasy-api/` is not cloned, so the handoff could not be checked against backend code. Treat every "Implemented" as a claim until it is on dev and a real response is in `backend-response/`.

## Goal
Make the three funnels trustworthy (identity, time, cohort, purchase and denominator definitions) and the customer lists usable for CS and Marketing, as the 18 Oct review asks. Keep the current UI and three tabs. The Action Dashboard is out of scope.

## Already done (do not redo)
- Converted / Not converted filter, `everPurchased` chip, `convertedBreakdown` footer, `abandoned` sent on Reached too (`list-query.ts` on `origin/feat/journey-cms`). Task `Portal-journey-list-converted-filter` is in `5-done/`; it still needs its changelog line and worktree cleanup.
- Name link to the customer page; full phone on the profile page.
- `customerKind` exists as an optional acquisition filter; `unavailableReason` for Ad clicked is handled.

## Flow (exists / change / new)
1. Mobile sends events (`booking_details_completed`, `slot_selected`, …) with `user_id`, `journey_id`, `x-app-instance-id`. **Exists**; verify (App-journey-event-qa).
2. Backend builds funnels from events and orders; a purchase counts only for `PURCHASE_COMPLETED` on a `PAID` order. **Change** (backend, claimed done).
3. Portal reads `GET /funnels/{booking|registration|acquisition}` and `…/stages/{stage}/people`. **Change**: new fields, renamed keys, nullable times, new filters.
4. Portal shows cards, connectors, table, lists. **Change**: labelled denominators, honest empty values, cohort and source filters, purchased filters.
5. CS and Marketing act from lists (export, push, send to CS). **Exists**; CS fields (UX-07) parked.

## Coverage of the review

| Item | Where it is handled |
|---|---|
| DQ-01 identity | Registration task (name, fallback, profile link); sign-off task (5-sample demo) |
| DQ-02 NaN time | Contract-sync task |
| DQ-03 53 vs 40 | Acquisition task (cohort text) + sign-off task (reconciliation) |
| DQ-04 / DQ-10 purchases, denominators | Acquisition task |
| DQ-05 ad clicks | Already handled; real data stays in `Cross-acquisition-ad-data` |
| DQ-06 Organic vs Unknown | Acquisition task |
| DQ-07 details vs slot | App event-QA task; backend stage mapping question |
| DQ-08 denominators | Funnel-denominators task |
| DQ-09 ever purchased vs converted | Done (see above) |
| DQ-11 unit | Contract-sync (types) + Definitions task (Customers vs Journeys text) |
| UX-01, UX-02 | Done |
| UX-03, 04, 05 | Backlog: list-advanced-filters (needs backend) |
| UX-06 | Backlog: recoverable-card (needs backend rules) |
| UX-07 | Parked with the CS module (501) |
| UX-08, 09, 10 | Registration task (UX-10 is a backend/mobile confirmation) |
| UX-11, 12 | Acquisition task |
| UX-13, 14 | Definitions task |
| Tracking, finance, consent, assisted conversion (T09) | Backlog: Cross-journey-attribution-finance-consent |
| P2 reporting, Action Dashboard | Out of scope |
| Sign-off evidence, QA T01 to T12 | Cross-journey-review-signoff |

## Data and API
New or changed contract (from the handoff, unverified): `converted`/`everPurchased`/`convertedBreakdown` (booking, done); `purchased`, `purchaseBreakdown`, `everPurchased`, `firstPurchasedAt` (registration); `customerKind=all|new|existing` default `new`, `sourceType`, `unit: people` + `identityUnit`, `purchaseCounts`, per-row `customerKind`/`converted`/`firstPurchase` (acquisition); `dropRate`, `dropPctOfFirst` on every stage; `null` times; KPI `otp_not_verified`; `otpReasons` limited to `expired`, `not_verified_unknown` plus `otpReasonAvailability`; source key `other:Unknown`; refund amount `null` with `not_tracked`.

## Tracking events
No new events. App-journey-event-qa only verifies the existing ones and the identity contract. If a fix changes event names or params, re-tier to T3.

## Gaps and decisions
- **Decided:** use backend `dropRate`/`dropPctOfFirst` for display; do not recompute in the portal (one source).
- **Decided (20 Oct, owner):** the `name` field is required on registration rows (backend request A9). The portal shows the name; when it is null or missing it shows the masked phone, with no fallback text. The registration list is searchable by name and phone (A12).
- **Decided:** reach tiles are mutually exclusive (`push + cs + adsOnly = count`); `suppressed` overlaps. Tooltips say so.
- **Decided:** old OTP reason names and `otp_failed` are removed, not kept, because backend renames them.
- **Gap:** after-purchase denominator field name and shape are not given.
- **Gap:** the spec's stage mapping never says what maps to `details_completed`.
- **Gap:** which list filters exist for UX-03 (service, last activity, furthest stage, value, CS status). Only `serviceIds` and `customerType` are in the spec.

## Open questions (answered 20 Oct in `backend-response/Journey_Tech_Review_Backend_Response.md`; what is left is in `Journey_Tech_Review_Backend_Request.md`)
1. With `customerKind=new` as the default, does `ad_clicked` always return `filter_not_applicable`? Recommend: treat the default as "not a filter" and keep `meta_not_connected`.
2. For `countBy=customer`, does `converted` mean any journey of that customer in the period was paid? Recommend yes, and the Reached `converted` count should equal the funnel `purchased`.
3. Is the new-vs-returning split by "had a paid order before first app open" (as `existing` is defined today)? Recommend yes; the portal labels it that way.

## Task breakdown

| # | Task | Repo | Depends on | Tier | Group |
|---|---|---|---|---|---|
| 1 | `Portal-journey-contract-sync` | portal | backend sample responses (V1 in task 7) | T2 | first |
| 2 | `Portal-journey-funnel-denominators` | portal | 1 | T2 | A (parallel) |
| 3 | `Portal-journey-registration-fixes` | portal | 1 | T2 | A (parallel) |
| 4 | `Portal-journey-acquisition-cohort` | portal | 1 | T2 | A (parallel) |
| 5 | `Portal-journey-definitions-tooltips` | portal | 2 and the definition sheet from 7 | T2 | after A |
| 6 | `App-journey-event-qa` | bEasy-app | none | T2 | B (parallel with all) |
| 7 | `Cross-journey-review-signoff` | docs/QA | all of the above for final evidence; first part (dev deploy, samples) can start now | T0 | B, finishes last |
| 11 | `Portal-journey-order-status-alignment` | portal + backend | answered 20 Oct | T2 | C (added 19 Oct) |
| 12 | `Portal-journey-abandon-rule-clarity` | portal | answered 20 Oct | T2 | C (added 19 Oct) |
| 13 | `Cross-journey-duplicate-orders` | backend first | real-data check (request file §4) | T3 | C (added 20 Oct) |
| 14 | `Portal-journey-phone-reveal` | portal + backend | backend item (in `backlog.md`), permission decision | T3 | backlog (postponed 20 Oct; name display stays in task 3) |
| 15 | `Portal-journey-registration-flow-rework` | cross | after task 3 | T3 | backlog (added 20 Oct) |
| 8 | `Portal-journey-list-advanced-filters` | portal | backend filters | T2 | backlog |
| 9 | `Portal-journey-recoverable-card` | portal | backend rules | T2 | backlog |
| 10 | `Cross-journey-attribution-finance-consent` | cross | product + backend | T3 | backlog |

Overlap: tasks 2, 3, 4 touch different files but all start from task 1 (`types/journey.ts`). Task 4 and `Cross-acquisition-ad-data` item E both touch `funnel/stage-card.tsx`; do item E after task 4 or coordinate through `_inbox`. Task 5 touches `insight-line.tsx` (also task 2), so it runs after.

## Out of scope
Action Dashboard; P2 reporting (spend, CAC, ROAS, saved segments, form diagnostics); CS owner/next follow-up fields (UX-07); real OTP delivery reasons; ad data.
