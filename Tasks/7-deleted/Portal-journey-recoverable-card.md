# Journey booking: "Recoverable now" opens a scoped list

- **ID:** T-0005
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T2 · new list entry, depends on backend rules
- **Claimed by:** none
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-30
- **Dropped:** 2026-10-30 · backend has not written the eligibility rules yet. Reopen when it does.
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (UX-06) · handoff "Recoverable counts exclude anonymous devices and require an abandoned registered journey"
- **Depends on:** backend: written eligibility and exclusion rules (already paid, duplicates, active attempt, no permissible contact), value rule, and a way to list exactly the people in the count. Abandon-rule sweep (`pending-request-backend.md` 3.1) still `501` for edits.
- **Touches:** `app/components/journey/funnel/{kpi-row,kpi-card}.tsx`, `app/components/journey/list/*`

## Goal
The card number and the list behind it are the same people, with the rule and exclusions stated in a tooltip. Do not claim consent eligibility; backend cannot prove it yet.

## Done when
- [ ] Card count equals the scoped list count
- [ ] Tooltip lists the eligibility rule and the exclusions
- [ ] Root `CHANGELOG.md` entry added
