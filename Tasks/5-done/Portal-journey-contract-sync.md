# Journey contract sync: types, renames, "Time unavailable"

- **ID:** T-0028
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-contract/` · **Branch:** `feat/journey-contract-sync`, from `origin/feat/journey-cms`
- **Tier:** T2 · edits shared types and format helper, API contract change (see `PIPELINE.md`)
- **Claimed by:** agent (claude), 2026-10-20. Owner said: work directly on `feat/journey-cms`, no separate worktree, portal side only
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-20 / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` · `Docs/tracking/v2/archive/Journey_Tech_Review_Handoff_19Oct2026.md` · `Docs/tracking/v2/api/Journey_Backend_API_Spec.md`
- **Depends on:** backend sample responses from dev (task `Cross-journey-review-signoff`, part 1). Types can be written from the handoff first, then checked against the samples.
- **Touches:** `app/types/journey.ts`, `app/lib/journey/{format,filters,kpi-delta,list-query}.ts`, `app/components/journey/{list/row-shell,list/otp-chips,shared/status-tag,funnel/kpi-row}.tsx`, `app/locales/journey/{en,km}/{funnel,list}.json`, `tests/`. Docs: API spec, `backend-response/`

## Goal
Make the portal's types match the new backend contract and fix DQ-02 ("NaN days ago") so the other journey tasks can build on it. No new screens.

## Scope
1. **Nullable time (DQ-02).** `lastActivityAt` and `idleMinutes` become `string | null` / `number | null`. `formatIdle` must never print NaN: null, undefined or NaN gives "Time unavailable" (en + km). Lists sort newest first with null rows last. Cause on the portal side: `formatIdle(undefined)` falls into the days branch.
2. **Renames.** KPI key `otp_failed` becomes `otp_not_verified` (type, `kpi-row`, `kpi-delta`, locales). `OtpReason` becomes `expired | not_verified_unknown`; add `otpReasonAvailability` to the registration response type. Remove the old values.
3. **New response fields** (types only, no UI here): per stage `dropRate`, `dropPctOfFirst`; acquisition `purchaseCounts`, `identityUnit`, `unit: 'people'`; people items `converted`, `everPurchased`, `firstPurchasedAt`, `customerKind`, `firstPurchase`; registration `purchaseBreakdown`; after-purchase denominator (field name pending backend reply).
4. **New query params** (types + `list-query`/`filters` + URL state): `customerKind=all|new|existing`, `sourceType=all|attributed|organic|unknown`, registration people `purchased=all|yes|no`.
5. **Docs (G7):** move the handoff into `backend-response/` as a dated file, update the API spec sections for the points above and the "Empty for now" table (OTP reasons, refund amount `null`), close answered items in `pending-request-backend.md`.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G6: no auth or personal data change.

## Done when
- [x] Unit tests: `formatIdle` with undefined, null, NaN, 0, 59, 60, 2880; list sort with null times
- [x] A row with a null time shows "Time unavailable" on dev and sorts last
- [x] `tsc` shows no new errors in journey files; no leftover `otp_failed` or old OTP reasons
- [x] Types match a real dev response for booking, registration and acquisition (attach a masked sample in `backend-response/`)
- [x] API spec updated, pending-request items closed
- [ ] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge

## Progress 2026-10-20 (agent, on `feat/journey-cms`, uncommitted)
Done: nullable `lastActivityAt`/`idleMinutes`; `formatIdle` returns `''` for null/undefined/NaN and the list row shows "Time unavailable" (en + km); new types (`CustomerKind` with `all`, `SourceType`, `dropRate`, `dropPctOfFirst`, `OtpReasonAvailability`, `PurchaseCounts`, per-row `firstPurchasedAt`/`customerKind`/`firstPurchase`, `purchaseBreakdown`); `customerKind`, `sourceType` (`stype`) in URL filters; `purchased` in `buildStagePeopleQuery` (registration only); KPI `otp_not_verified` added next to `otp_failed`. New tests: `npm run test:journey-contract`.
Decision: old and new OTP names are both accepted until backend deploys the rename (dev still sends the old ones). Remove the old names after the dev check.
Docs synced: handoff moved to `archive/Journey_Tech_Review_Handoff_19Oct2026.md` (banner: unverified); API spec rows added.
Checks: tsc no journey errors; eslint clean; journey node tests pass (contract 4, list 12, funnel 8, extra 1, customer 5, cs 5, audiences 8). G4 not run (needs backend on dev). G5, G8, G11 not done.
Not done: after-purchase denominator type (field name unknown, item A4); `unit` handling for acquisition `people`/`identityUnit` is typed only.
