# Journey customer list sheet: name link, Status filter, footer message

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T2 · edits shared list components of the Journey feature, no new API (re-check before testing: the name link opens a page that shows customer personal data, so keep it behind the existing permission)
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-journey-list-fixes/` (create when claimed) · **Branch:** `feat/journey-list-fixes`. The journey list code exists only on `feat/journey-cms`, so branch from it (or wait until it merges). Do not work in the `bEasy-portal/` checkout.
- **Docs:** `Docs/tracking/v2/requirements/Journey_Tracking_UI_Flow_v2.1.md` (§ customer list, footer), `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` (§ people)
- **Depends on:** `Portal-journey-cms` (branch `feat/journey-cms`) for the code. Item 2 option B needs backend 4b.4
- **Touches:** `app/components/journey/list/{people-row,row-shell,list-search,use-stage-list,list-footer}.tsx`, `app/lib/journey/footer-state.ts`, `app/locales/journey/{en,km}/list.json`. Overlaps with `Portal-journey-cms` and `Portal-journey-list-converted-filter` (same files; do not run in parallel with the latter)

## Goal
Fix three things found in review of the customer list sheet (Customer Journey drawer).

## Scope

**1. Customer name links to the customer's information.** In a customer row (booking funnel, `kind: 'customer'`) the name becomes a link to `customer/:id/profile` (the registered customer page; `routes.ts:76`, guarded by `MARKETING_CUSTOMER:VIEW`). Clicking the rest of the row still opens the overview sheet. The link stops row selection and row open (`stopPropagation`), opens in the same tab, and shows only when the user has that permission (`usePermission`). Check what the profile page shows: the request says "like phone number", and the list itself shows a masked phone. If the profile page does not show the full phone, say so in the task notes and ask before changing it (personal data). Guests, devices and registration rows have no link.

**2. Status filter does nothing on a Reached list (bug).** Cause found: `use-stage-list.ts` sends `abandoned` only when `sheet.state === 'dropped'` (`abandoned: sheet.state === 'dropped' ? abandoned : 'all'`), while `stage-list-content.tsx` shows the dropdown whenever the response has `abandonStrip`, so on Reached it is shown but ignored. Fix, pick one:
- **A (recommended, no backend):** show the Status dropdown only on Dropped lists. On Reached, Status is meaningless (everyone moved on), and item 3 replaces it with Converted.
- **B:** send the filter on Reached too; needs backend answer 4b.4.
Add a test for the param building (`buildStagePeopleQuery` / `useStageList`): Dropped sends it, Reached does not. Reset the filter when the sheet switches Reached/Dropped so a hidden filter never stays active.

**3 is a separate task** (`Portal-journey-list-converted-filter`).

**4. Footer message.** Cause found: the hint `reached` prints `breakdown.moved_on` as "{{count}} of these already bought". `moved_on` is everyone who went past this step (the screenshot: 149 of 162), not buyers (the funnel shows 28), so the sentence is wrong. Do now: drop the number and use the existing `reachedGeneric` text for all Reached lists ("These people already reached this step. Switch to Dropped here to send push or CS."), remove `reached` from `footer-state.ts` and the locales (en and km), and update the footer-state unit tests. Do **not** invent a "bought" number from existing fields. Later, when backend sends `converted`/`everPurchased` (backend request 4b), show "X converted in this period · Y bought before" in the converted-filter task.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
- G6 security review skipped unless item 1 shows more personal data than today; write `Skipped G6: reason`.

## Done when
- [ ] Name link opens the right customer page; row click, checkbox and link do not interfere; hidden without `MARKETING_CUSTOMER:VIEW`
- [ ] Status dropdown is absent on Reached (or works, if option B); a filter set on Dropped does not leak into Reached
- [ ] Footer never shows a "bought" number on Reached; unit tests updated
- [ ] Khmer strings checked for the changed text
- [ ] Tested on dev backend with a Reached and a Dropped list (e.g. Service viewed, Details completed)
- [ ] UI Flow doc updated for the footer rule (G7)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
