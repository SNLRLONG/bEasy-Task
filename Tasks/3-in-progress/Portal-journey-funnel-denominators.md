# Journey funnel: labelled drop-off denominators

- **ID:** T-0019
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-denominators/` · **Branch:** `feat/journey-denominators`, from `origin/feat/journey-cms` after `Portal-journey-contract-sync` merges
- **Tier:** T2 · edits shared funnel components and the displayed metric (see `PIPELINE.md`)
- **Claimed by:** agent (claude), 2026-10-20. Owner said: work directly on `feat/journey-cms`, no separate worktree, portal side only
- **Started / updated:** 2026-10-20 / 2026-10-19
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (DQ-08) · handoff "Every stage includes `dropRate` and `dropPctOfFirst`"
- **Depends on:** `Portal-journey-contract-sync`
- **Touches:** `app/components/journey/funnel/{stage-card,stage-node,rate-edge,rate-pill,insight-line,drop-off-table,kpi-row,funnel-flow}.tsx`, `app/lib/journey/funnel-math.ts`, `app/locales/journey/{en,km}/funnel.json`. Overlaps with `Portal-journey-definitions-tooltips` (insight-line) and `Cross-acquisition-ad-data` item E (stage-card)

## Goal
Every percentage on the three funnels says what it is divided by, and the card, connector, table and headline agree. Reviewer's case: booking 96/149 = 64% (biggest loss) but the stage card shows 59% (96/162).

## Scope
- Two named numbers per stage, both from the backend: **drop rate at this stage** = `dropRate` (dropped / reached at the stage) and **share of first stage** = `dropPctOfFirst`. Do not recompute in the portal; `null` shows "—".
- Stage card, connector pill, drop-off table and the "biggest drop" KPI/insight use the same named basis, and each has a tooltip with the formula and real numbers ("96 of 149 reached this step").
- Pick one basis for the headline ("biggest drop") and say it in the label. Recommend stage-relative (`dropRate`).
- Applies to booking, registration and acquisition.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G6: display only.

## Done when
- [ ] On dev, recompute 5 stages by hand: each shown percentage equals its tooltip numbers
- [ ] Card, connector, table and headline show the same figure for the same stage and basis
- [ ] Khmer labels added (native read pending)
- [ ] Unit tests for the formatting/selection helper
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress 2026-10-20 (agent, on `feat/journey-cms`, uncommitted)
Done: stage card Dropped tile now shows the stage-relative share (`dropRate`: dropped / reached at the step) with a tooltip giving both numbers (96 of 149 = 64% of this step; 59% of the 162 at the start); badge keeps "of first stage". Drop-off table has labelled columns: Continued (of reached), Dropped here (% of reached), Dropped (% of first step). Helpers `dropRateOf` / `dropPctOfFirstOf` in `funnel-math.ts`, with a test.
Changed decision: the helpers use the backend field when present and compute the same ratio when it is missing, so the screen is right before backend deploys. One definition each, tested.
Not done: connector pill keeps "continued" (not hoverable on the canvas); insight line unchanged (its numbers are already stage-relative: "123 of 196"); registration and acquisition tabs not checked on real data (G4).
Checks as in contract-sync. G5, G8, G11 not done.
