# Journey lists: advanced filters, filter chips, search labels

- **ID:** T-0003
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T2 · new filters on live API calls, needs a backend change
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (UX-03, UX-04, UX-05)
- **Depends on:** backend request: people-list filters `serviceIds` (exists), last activity range, furthest stage, customer type, estimated value range, CS contact status. Not in the handoff.
- **Touches:** `app/components/journey/list/*`, `app/lib/journey/{list-query,list-filters}.ts`, `app/types/journey.ts`

## Goal
Make lists usable for CS and Marketing: filters under an "Advanced" section, visible active-filter chips with clear, filters applied consistently to counts and drill-down, name/phone search with "Guest / Device" for anonymous rows.

## Scope
- Advanced section: service, last activity, customer type, furthest stage, estimated value; CS contact status only when the CS module is live (UX-07 stays parked).
- Active chips with "Clear all"; filters also change stage counts and summary cards, or the list says it is a filtered subset.
- Search stays inside the list; show the real backend name for registered users.

## Done when
- [ ] Backend request answered and API spec updated
- [ ] Counts, cards and list agree for every filter (QA T12)
- [ ] Root `CHANGELOG.md` entry added
