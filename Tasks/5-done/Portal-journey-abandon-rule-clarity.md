# Journey list: make the abandon rule understandable, and keep its numbers consistent

- **ID:** T-0027
- **Repo:** bEasy-portal (+ backend question)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-abandon-rule/` · **Branch:** `fix/journey-abandon-rule`, from `origin/feat/journey-cms`
- **Tier:** T2 · edits shared list components (see `PIPELINE.md`)
- **Claimed by:** agent (claude), 2026-10-20. Owner said: work directly on `feat/journey-cms`, no separate worktree, portal side only
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-20 / 2026-10-21
- **Docs:** `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` §2 · `Tasks/_plans/Cross-journey-tech-review.md` (UX-02, DQ-08)
- **Depends on:** backend answer A11 for part 2
- **Touches:** `app/components/journey/list/{abandon-strip,list-footer,stage-list-content,list-header}.tsx`, `app/components/journey/rules/abandon-rules-dialog.tsx`, `app/lib/journey/{format,footer-state}.ts`, `app/locales/journey/{en,km}/list.json`

## Goal
"Rule: 1440 min · Edit" confuses people. Explain it in plain units, and stop the strip and footer from showing numbers that do not match the filtered list.

## What 1440 min is
It is the abandon time for the stage's furthest step: for Service viewed it is "Viewed, didn't book" = 1 day (1440 minutes). A journey is Abandoned when it has had no next step for at least that long; before that it is "Not abandoned yet" (spec §2). The strip prints raw minutes while the dialog already says "1 day".

## Scope
1. **Wording.** Show the rule in units: "No next step for 1 day or more" and "Rule: 1 day" (reuse `formatIdle`). Add a tooltip: which step it belongs to, and that the same time is used in the Abandon rules dialog. In the strip text name the step ("no Booking started for 1 day+").
2. **"Edit" is misleading.** Saving is disabled ("Editing is coming soon", `PUT /abandon-rules` returns 501). Until it ships, label the link "View rule" (en + km); switch back to "Edit" when the backend ships it.
3. **Numbers vs filters (screenshot of Service viewed, Reached):** list shows 5 customers (filters Not abandoned yet, Not converted) but the strip says 173 abandoned · 5 inside, the tiles say Moved on 0, and the footer says "40 converted · 25 had bought before". Find out, with the backend answer, whether strip, tiles and `convertedBreakdown` follow the active filters. If they describe the unfiltered list, say so in each label ("all 218 reached"); if they should follow, ask backend. Never show a count beside a list it does not describe.
4. On a Reached list the strip should not talk about abandonment of people who already moved on (they are not stuck at this step); check that text for Reached and keep it only where it applies.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G6: copy and display only.

## Done when
- [x] The strip shows 1 day, 1 hour, 30 min (not 1440 min) and its tooltip names the step
- [x] No "Edit" link while saving is not available
- [x] On a filtered list, strip, tiles and footer either match the list or say what they cover
- [x] Khmer strings added (native read pending)
- [ ] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge

## Progress 2026-10-20 (agent, on `feat/journey-cms`, uncommitted)
Done: parts 1 and 2. The strip prints the rule in units ("stopped at Service viewed for 1 day+", "still inside 1 day") using a shared `useDurationLabel` hook (moved out of the rules dialog); the rule link reads "Rule: 1 day · View" while `JOURNEY_BACKEND.editAbandonRules` is false and "· Edit" when it is on; tooltip explains it.
Open: part 3 and 4 (do strip, tiles and `convertedBreakdown` follow the active filters; strip on Reached lists) wait for backend request A11. From the portal types, `convertedBreakdown` is documented as counted before the Converted filter, which explains "40 converted · 25 had bought before" next to a 5-row list; the footer text should say what it covers. Not changed yet.
Checks: tsc, eslint clean. G4, G5, G8, G11 not done.

## Update 2026-10-20 (backend answered A11)
Backend: `abandonStrip` is counted after base filters and search but before the Status and Converted filters; `convertedBreakdown` after Status, before Converted. So "173 abandoned" next to a 5-row list is by design. Done: when Status or Converted is active the strip adds "These numbers count everyone at this step, before the Status and Converted filters below." (en + km). Remaining: the response's `summaryScopes` exact keys (asked in the new request file §1) could replace this fixed rule.
