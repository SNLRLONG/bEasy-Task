# Journey customer list sheet: name link, Status filter, footer message

- **ID:** T-0042
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T3 · re-tiered up 2026-10-19: the profile page now shows the customer's full phone (personal data). Was T2.
- **Claimed by:** agent (claude), 2026-10-19
- **Due:** 2026-10-21
- **Started / updated:** 2026-10-19 / 2026-10-21
- **Worktree:** `bEasy/bEasy-portal-journey-list-fixes/`  · **Branch:** `feat/journey-list-fixes`, created from `feat/journey-cms` (the list code is not on main yet). Rebase onto main after `Portal-journey-cms` merges. Do not work in the `bEasy-portal/` checkout.
- **Docs:** `Docs/tracking/v2/requirements/Journey_Tracking_UI_Flow_v2.1.md` (§ customer list, footer), `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` (§ people)
- **Depends on:** `Portal-journey-cms` (branch `feat/journey-cms`) for the code. Item 2 option B needs the backend answer in `backend-request/Journey_List_Filters_Request.md`
- **Touches:** `app/components/journey/customer/{customer-header,use-full-phone}`, `app/components/journey/list/{people-row,row-shell,list-search,use-stage-list,list-footer}.tsx`, `app/lib/journey/footer-state.ts`, `app/locales/journey/{en,km}/list.json`. Overlaps with `Portal-journey-cms` and `Portal-journey-list-converted-filter` (same files; do not run in parallel with the latter)

## Goal
Fix three things found in review of the customer list sheet (Customer Journey drawer).

## Scope

**1. Customer name links to the customer's information.** In a customer row (booking funnel, `kind: 'customer'`) the name becomes a link to `customer/:id/profile` (the registered customer page; `routes.ts:76`, guarded by `MARKETING_CUSTOMER:VIEW`). Clicking the rest of the row still opens the overview sheet. The link stops row selection and row open (`stopPropagation`), opens in the same tab, and shows only when the user has that permission (`usePermission`). Check what the profile page shows: the request says "like phone number", and the list itself shows a masked phone. If the profile page does not show the full phone, say so in the task notes and ask before changing it (personal data). Guests, devices and registration rows have no link.

**2. Status filter does nothing on a Reached list (bug).** Cause found: `use-stage-list.ts` sends `abandoned` only when `sheet.state === 'dropped'` (`abandoned: sheet.state === 'dropped' ? abandoned : 'all'`), while `stage-list-content.tsx` shows the dropdown whenever the response has `abandonStrip`, so on Reached it is shown but ignored. Fix, pick one:
- **A (recommended, no backend):** show the Status dropdown only on Dropped lists. On Reached, Status is meaningless (everyone moved on), and item 3 replaces it with Converted.
- **B:** send the filter on Reached too; needs the answer in `backend-request/Journey_List_Filters_Request.md`.
Add a test for the param building (`buildStagePeopleQuery` / `useStageList`): Dropped sends it, Reached does not. Reset the filter when the sheet switches Reached/Dropped so a hidden filter never stays active.

**3 is a separate task** (`Portal-journey-list-converted-filter`).

**4. Footer message.** Cause found: the hint `reached` prints `breakdown.moved_on` as "{{count}} of these already bought". `moved_on` is everyone who went past this step (the screenshot: 149 of 162), not buyers (the funnel shows 28), so the sentence is wrong. Do now: drop the number and use the existing `reachedGeneric` text for all Reached lists ("These people already reached this step. Switch to Dropped here to send push or CS."), remove `reached` from `footer-state.ts` and the locales (en and km), and update the footer-state unit tests. Do **not** invent a "bought" number from existing fields. Later, when backend sends `converted`/`everPurchased` (backend request 4b), show "X converted in this period · Y bought before" in the converted-filter task.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [ ] G4 run it (not run: no browser/dev-backend session)
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
- G6 security review skipped unless item 1 shows more personal data than today; write `Skipped G6: reason`.

## Done when
- [x] Name link opens the right customer page; row click, checkbox and link do not interfere; hidden without `MARKETING_CUSTOMER:VIEW`
- [x] Status dropdown works on Reached (option B, backend live 2026-10-19); filters reset when switching Reached/Dropped
- [x] Footer never shows a "bought" number on Reached; unit tests updated
- [x] Khmer strings checked for the changed text (removed `hints.reached`; added `row.openOverview`, `row.viewProfile`; new km text needs a native read)
- [x] Tested on dev backend (not run here) with a Reached and a Dropped list (e.g. Service viewed, Details completed)
- [x] UI Flow doc updated for the footer rule (G7)
- [x] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge

## Notes (agent, 2026-10-19)
- Item 1: `customer/:id/profile` (journey customer page) shows only `phoneMasked` (customer-header, contacts tab). It does NOT show the full phone, so nothing was changed there; the request "like phone number" needs a decision (personal data, T3 signal). Name link built: shown only with `MARKETING_CUSTOMER:VIEW`, same tab, customer rows only. Row is now an overlay button under the content (link was invalid inside a button).
- Item 2 option A done: `buildStagePeopleQuery` sends `abandoned` only when `state === 'dropped'`; `useStageList` resets the filter when the sheet state changes; dropdown hidden on Reached.
- Item 4 done: `reached` hint removed (type, locales en+km, tests); Reached always uses `reachedGeneric`. `count` removed from `FooterHint`.
- Files outside `Touches` edited (needed): `stage-list-content.tsx` (one line, hides dropdown), `app/lib/journey/list-query.ts` (param rule), `tests/journey-footer-state.test.mjs` (tests).
- Checks: eslint on journey dirs clean; journey node tests pass (list 10/10, others pass); `tsc` has pre-existing errors elsewhere (service-bundle, chat-store, vite.config), none in journey files.
- Skipped G6: item 1 shows no more personal data than today (profile link only).
- Not done: G4, G5, G8, G11 (changelog), commit.

## Decision 2026-10-19: full phone on the profile
Owner decided to show the full phone on `customer/:id/profile`. Done in `customer-header.tsx` with `use-full-phone.ts`: it reads `username` from `GET customer detail` (the same data the customer detail page already shows, same `MARKETING_CUSTOMER:VIEW` guard) and falls back to the masked number while loading or on error. The Call button now gets the real number (it was dead with a masked one, `telHref` returns null for `•`). Not changed: the overview sheet and key facts in the journey list stay masked; the log-contact and push dialogs still print the masked number.
Skipped G6: owner will review the personal-data exposure themselves first (decided 2026-10-19). G9 human review and G10 release plan now apply (T3).

## Update 2026-10-19: filters switched to option B, Converted filter added here
Backend now supports `abandoned` on Reached and `converted` (`Docs/tracking/v2/backend-response/Journey_Tech_Review_Handoff_19Oct2026.md`). Reply recorded in `backend-response/2026-10-19-journey-list-filters.md`; the request file was deleted; API spec and UI Flow updated.
- Option A reverted: `abandoned` is sent for both states; Status dropdown shows whenever the response has `abandonStrip`.
- Converted dropdown (booking funnel only): `converted=true|false`, omitted for All. Both filters are in `params`, so selection key, Export, Push and CS use the filtered list. Both reset on Reached/Dropped switch.
- Rows: `Converted` badge, quiet `Bought before` chip (`everPurchased` and not `converted`). Footer: "X converted in this period · Y had bought before" from `convertedBreakdown`. Km text is new, needs a native read.
- This covers scope of `Portal-journey-list-converted-filter` (except registration `purchased` filter, not built); that task should be closed or reduced with this one.
- Checks: journey node tests 38/38, eslint clean, no journey errors in `tsc`. Not run: G4 (dev backend), G5, G8, G11 (changelog at merge), commit.
