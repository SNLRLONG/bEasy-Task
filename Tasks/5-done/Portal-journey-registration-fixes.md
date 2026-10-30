# Journey registration: identity, OTP honesty, Purchased filter

- **ID:** T-0020
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-registration/` · **Branch:** `fix/journey-otp-reasons`, from `main` (created 2026-10-30; journey-cms is already merged)
- **Tier:** T2 · new filter on a live API call, shows personal data (name, masked phone) (see `PIPELINE.md`)
- **Claimed by:** agent (claude), 2026-10-30. Owner asked for a new branch to fix the failed QA check (OTP reasons card)
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-20 / 2026-10-30
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (DQ-01, UX-08, UX-09, UX-10) · handoff "Registration"
- **Depends on:** `Portal-journey-contract-sync`
- **Touches:** `app/components/journey/funnel/{registration-funnel-tab,otp-reasons-card,kpi-row}.tsx`, `app/components/journey/list/{otp-chips,row-shell,people-row,list-search,use-stage-list,list-footer}.tsx`, `app/lib/journey/{list-query,footer-state}.ts`, `app/locales/journey/{en,km}/{funnel,list}.json`. Overlaps with the acquisition task only in `list-search`/`use-stage-list`: merge one first

## Goal
The Registration tab shows real customer names, does not call abandonment an OTP failure, never shows uninstrumented OTP reasons as zero, and lets staff see who has no order yet.

## Scope
1. **Identity (DQ-01).** Completed rows show the saved name. When the name is null or missing, no name is shown and the masked phone is the title (owner, 2026-10-20: no fallback text). Click opens the customer page (reuse the name-link rule from the list-fixes work: customer rows, `MARKETING_CUSTOMER:VIEW`). Devices and guests have no link.
2. **OTP honesty (UX-09).** KPI label "OTP not verified" with a tooltip: includes expired and people who left; not a delivery or code failure. The reasons card shows only `expired` and `not_verified_unknown`. For delivery, gateway, wrong code and rate limit, show "Not tracked yet" using `otpReasonAvailability`, never 0 and never a bar.
3. **Purchased filter (UX-08).** Dropdown next to search on registration lists: All / Purchased / No order yet, with counts from `purchaseBreakdown`. Row shows "First order {date}" from `firstPurchasedAt` when present. No wording that suggests every new user needs a call. The filter is part of the selection key, export and list actions, resets on Reached/Dropped switch.
4. **Event order (UX-10).** Confirmed by the owner on 2026-10-20: the funnel does not match the app flow. Do not change stages here; the rework is `Tasks/1-backlog/Portal-journey-registration-flow-rework.md`.
5. **Name on every row.** Where a registration row has a `name`, show it as the title (phone below); the "OTP" avatar only for rows with no name. Required, not optional. The phone stays masked (full phone is postponed in `1-backlog/Portal-journey-phone-reveal.md`).

6. **Search by name and phone.** The search box is on every list and sends `q`. Portal: strip spaces, dashes and brackets from phone-like input before sending. The backend must match name and phone for completed and OTP-only rows (request A12), and must always send `name` (A9).

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [x] G3 tests  - [x] G4 run it
- [ ] G5 code review  - [ ] G6 security (names + phones in lists)  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [x] 5 sample completed registrations open the right customer profile and show the CRM name (sign-off demo 1)
- [x] Purchased + No order yet counts add up to the list count
- [x] No OTP category without data is rendered as 0
- [ ] Export with the filter returns the same count
- [ ] Searching a name and a phone (typed with spaces or dashes) in a registration list finds the person (needs A12)
- [x] Registration rows with a saved name show the name; no "OTP" avatar on them
- [ ] Khmer strings added (native read pending)
- [x] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge

## Progress 2026-10-20 (agent, on `feat/journey-cms`, uncommitted)
Done: item 5 (name): rows show `name` as title, masked phone below; no name (null or missing) shows the masked phone as the title, no fallback text (owner decision); registered users with no name get a "U" avatar, the "OTP" avatar stays only for unregistered OTP rows. Known customers in a registration list open their customer page (`customerId` required). Item 3: Purchased / No order yet dropdown with counts from `purchaseBreakdown`, reset on Reached/Dropped switch, part of the query key (so export and actions use it), "First order {date}" chip from `firstPurchasedAt`. Item 2: OTP card title "Why the OTP was not verified", new reason labels, one "Not tracked yet" line instead of zero bars; KPI definition no longer says failure.
Not done: the `purchased` param and `purchaseBreakdown` are not on dev yet (portal sends it only when not "All", so nothing changes until the user picks it); `otpReasonAvailability` shape unknown (asked in Request A1); name link on the name itself (the whole row opens the customer page; no separate link); G4, G5, G6, G8, G11.

## QA 2026-10-30 (dev portal, browser) - FAILED one check, moved back to in-progress
Passed: KPI label "OTP not verified"; completed rows show the saved name as title with masked phone and a letter avatar; Purchased filter is live on dev (All 27 = Purchased 2 + No order yet 25), filtering returns 2 people with "First order {date}" and "Bought before" chips.
**Failed:** the "Why the OTP was not verified" card renders SMS not delivered, API / gateway error, OTP rate limited and Blocked after 3 wrong tries as `0` while its own footer says SMS delivery is not tracked. Done-when says "No OTP category without data is rendered as 0". Fix: show "Not tracked" (or hide the row) unless `otpReasonAvailability` says the reason is measured. Also "1 people" should be "1 person".
Not checked: phone/name search in 3 formats (needs a full phone), export count with the filter, Khmer.

## Fix 2026-10-30 (agent, branch `fix/journey-otp-reasons`, uncommitted)
Done: the OTP reasons card lists a reason as "Not tracked yet" (no number, no bar, not clickable) unless it has a count above 0 or `otpReasonAvailability` says it is measured; `expired` and `not_verified_unknown` always show. Footer note shows when any row is untracked. New helper `app/lib/journey/otp-reasons.ts` with 4 tests (`npm run test:journey-otp-reasons`); journey tests pass; prettier and eslint clean; no type errors in the journey files (81 existing errors elsewhere in the repo). Added `notTrackedRow` in en and km (km native read pending).
Not done: run it in the browser on dev (needs a deploy of the branch), "1 people" wording (the unit label has no singular form), G5, G8, G11, phone-search and export checks.

## QA 2026-10-30, fix deployed to dev (browser)
Passed: "SMS not delivered" shows "Not tracked yet" with no number and no bar; clicking it opens nothing; the footer note shows; "Wrong code entered" still shows its real 1.
Backend answered 30 Oct (`Docs/tracking/v2/backend-response/Journey_OTP_Reasons_Backend_Response_30Oct2026.md`): gateway, wrong code, rate limit, blocked and expired-or-left are recorded; only SMS delivery is not. Dev matches, so the zeros are real "no one in this cohort" counts and the portal needs no further change.
Still unchecked: phone/name search in 3 formats, export count with the filter, Khmer read, G5, G8, G11.

## Status 2026-10-30
Fix `cf6eee7e` is on `origin/fix/journey-otp-reasons` and on `dev` (cherry-picked, deployed, tested). Not on `main` yet: open the PR from `fix/journey-otp-reasons` to `main`. After the merge: add the `[Portal]` entry to the root `CHANGELOG.md`, remove the worktree `bEasy/bEasy-portal-journey-registration/` and the branch, move this file to `5-done/`.
Carried over, not done: phone/name search in 3 formats, export count with the filter, Khmer native read, G5 code review, G8 PR message.

## Date picker 2026-10-30
Branch commit `91a1a1b5` swaps the journey date filter for the shared v2 `DateRangePicker` (92-day limit and no future dates kept via new optional props). Staged on `dev` but not committed, so the deployed dev still shows the old filter (Sunday-first calendar, old footer). Test after it is deployed: presets, custom range, future day, range over 92 days, URL and funnel numbers follow.

## Closed 2026-10-30
Merged to `main` as #531 (squash `c0c4d9d2`, includes the OTP card fix and the date picker). Changelog entry added; worktree `bEasy-portal-journey-registration` and local branch `fix/journey-otp-reasons` removed (remote branch left).
Carried over, not done: browser test of the new date picker (it was never deployed to dev before the merge), phone/name search in 3 formats, export count with the filter, Khmer native read of "Not tracked yet", G5 code review, G8 PR message.