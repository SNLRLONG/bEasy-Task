# Journey definitions, tooltips and next-step links

- **ID:** T-0011
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-definitions/` · **Branch:** `feat/journey-definitions`, from `origin/feat/journey-cms` after the denominators, registration and acquisition tasks merge
- **Tier:** T2 · edits shared journey components (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (UX-13, UX-14, DQ-11) · the metric-definition sheet from `Cross-journey-review-signoff`
- **Depends on:** `Portal-journey-funnel-denominators`, the metric-definition sheet
- **Touches:** `app/components/journey/shared/interest-badge.tsx`, `app/components/journey/list/{reach-tiles,row-chips}.tsx`, `app/components/journey/funnel/{insight-line,kpi-card,journey-page-shell}.tsx`, `app/locales/journey/{en,km}/*.json`

## Goal
Labels explain themselves, and the suggested next step opens the right filtered list or an existing module. No new action system.

## Scope
1. **Tooltips (UX-13).** Interest badges (what High and Very High mean); reach tiles Push / CS Contact / Ads only: say they are mutually exclusive and add up to the list, and that "suppressed" overlaps them; Abandoned vs Not abandoned yet vs Converted are different things.
2. **Definitions (DQ-11).** A "Definitions" popover per tab from the sign-off sheet: customers vs journeys, what counts as abandoned (rule minutes), conversion, attribution window, timezone (Asia/Phnom_Penh, +07).
3. **Next step (UX-14).** The insight line becomes a link: booking and registration open the dropped list of that stage; recoverable opens the dropped list of its stages (until `Portal-journey-recoverable-card`); wording stays advice, not a task.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G6: copy and links only.

## Done when
- [ ] Every badge and tile named above has a tooltip in English and Khmer
- [ ] Insight line click lands on the right list with the filters kept
- [ ] Definitions text equals the sign-off sheet (one source)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
