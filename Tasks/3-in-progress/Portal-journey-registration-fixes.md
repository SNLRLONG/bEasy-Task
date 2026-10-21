# Journey registration: identity, OTP honesty, Purchased filter

- **ID:** T-0020
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-registration/` · **Branch:** `feat/journey-registration`, from `origin/feat/journey-cms` after `Portal-journey-contract-sync` merges
- **Tier:** T2 · new filter on a live API call, shows personal data (name, masked phone) (see `PIPELINE.md`)
- **Claimed by:** agent (claude), 2026-10-20. Owner said: work directly on `feat/journey-cms`, no separate worktree, portal side only
- **Started / updated:** 2026-10-20 / 2026-10-19
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
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security (names + phones in lists)  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] 5 sample completed registrations open the right customer profile and show the CRM name (sign-off demo 1)
- [ ] Purchased + No order yet counts add up to the list count
- [ ] No OTP category without data is rendered as 0
- [ ] Export with the filter returns the same count
- [ ] Searching a name and a phone (typed with spaces or dashes) in a registration list finds the person (needs A12)
- [ ] Registration rows with a saved name show the name; no "OTP" avatar on them
- [ ] Khmer strings added (native read pending)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress 2026-10-20 (agent, on `feat/journey-cms`, uncommitted)
Done: item 5 (name): rows show `name` as title, masked phone below; no name (null or missing) shows the masked phone as the title, no fallback text (owner decision); registered users with no name get a "U" avatar, the "OTP" avatar stays only for unregistered OTP rows. Known customers in a registration list open their customer page (`customerId` required). Item 3: Purchased / No order yet dropdown with counts from `purchaseBreakdown`, reset on Reached/Dropped switch, part of the query key (so export and actions use it), "First order {date}" chip from `firstPurchasedAt`. Item 2: OTP card title "Why the OTP was not verified", new reason labels, one "Not tracked yet" line instead of zero bars; KPI definition no longer says failure.
Not done: the `purchased` param and `purchaseBreakdown` are not on dev yet (portal sends it only when not "All", so nothing changes until the user picks it); `otpReasonAvailability` shape unknown (asked in Request A1); name link on the name itself (the whole row opens the customer page; no separate link); G4, G5, G6, G8, G11.
