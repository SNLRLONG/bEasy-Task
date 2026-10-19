# Journey customer list sheet: Converted / Not converted filter

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T2 · new filter on a live API call, needs a backend change
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-journey-converted/` (create when claimed) · **Branch:** `feat/journey-converted`. Branch from `feat/journey-cms` (the list code lives there) after `Portal-journey-list-sheet-fixes` merges.
- **Docs:** `Docs/tracking/v2/backend-request/Journey_List_Filters_Request.md` §2 · `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` (people endpoint)
- **Depends on:** backend request §2.1 to 2.3 · `Portal-journey-list-sheet-fixes`
- **Touches:** `app/components/journey/list/{list-search,use-stage-list,list-footer}.tsx`, `app/lib/journey/{list-query,footer-state}.ts`, `app/types/journey.ts`, `app/locales/journey/{en,km}/list.json`, the export dialog (the filter must be part of the export query). Overlaps with `Portal-journey-list-sheet-fixes`

## Goal
Add a **Converted | Not converted** filter to the customer list sheet, and make the "bought" numbers clear. Reviewer's point: "149 of these already bought" versus 28 purchases in the funnel confuses two things, *ever purchased* and *converted in this journey*.

## Scope
- Dropdown next to search (Reached and Dropped lists): `All`, `Converted`, `Not converted`. Combines with the Dropped-only Status filter. Sends `converted=true|false`, omitted for `All` (backend 2.1).
- Row chip: keep `Converted`; add a quieter `Bought before` chip when `everPurchased` and not `converted` (2.2).
- Footer: replace the removed message with two separate numbers from `convertedBreakdown` (2.3), worded so they cannot be mixed up, e.g. "28 converted in this period · 9 had bought before". Hide it when the backend sends nothing.
- The filter is part of the selection key and of `ListAction.query`, so Export, Push and CS use the filtered list. Check the export count matches.
- Add the param to `StagePeopleQuery` in `types/journey.ts` and to the API spec doc.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [x] Converted + Not converted counts add up to the list count on dev
- [x] The Converted count of a stage's Reached list agrees with the funnel's purchased number, or the difference is explained in the footer
- [x] Filter survives paging, resets on Reached/Dropped switch, is included in export
- [x] API spec updated; backend request file closed once answered
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
