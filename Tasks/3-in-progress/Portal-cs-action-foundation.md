# Portal: CS Action foundation and task queue

- **ID:** T-0104
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Worktree:** `bEasy/bEasy-portal-cs-action/` (shared by T-0104, T-0105, T-0106) · **Branch:** `feat/cs-action`
- **Tier:** T3 · permissions and role modules, new route, new API client, shared nav/endpoint/query-key files (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-05
- **Started / updated:** 2026-11-05 / 2026-11-05
- **Docs:** Plan `Tasks/_plans/Portal-cs-action-performance.md` · `Docs/tracking/v2/backend-response/CS_Admin_Portal_Handoff.md` (contract and response examples) · `Docs/tracking/v2/requirements/bEasy_CS_Action_and_Performance_Developer_Brief_V1.md`
- **Depends on:** backend requests R1 (permission strings) and R4 (staging with migration and flag on). Build against mocks until then.
- **Touches:** `app/routes/customer-services/cs-action.tsx`, `app/components/cs-action/*`, `app/hooks/query/use-cs-*`, `app/lib/cs-action.ts`, `app/types/cs-action.ts`, `app/lib/permission.ts`, `app/components/sidebar/nav-config.ts`, `app/routes.ts`, `app/api/endpoint.ts` (all CS endpoints incl. performance), `app/constants/query-key-enum.ts` (all CS keys), `app/locales/*.json`, role editor module list, `tests/cs-action*`. Shares nav/permission files with T-0058: sequence after it merges.

## Goal
One CS Action page: 4 counters, 4 category tabs, supported filters only (status, due, Mine), one paginated task list, empty/loading/error states, permission gating. Pre-adds endpoint constants and query keys for the whole CS feature so T-0105 and T-0106 do not collide.

## Progress (2026-11-05): UI phase on sample data
Built in one worktree for all three tasks so shared files do not conflict: `bEasy/bEasy-portal-cs-action/`, branch `feat/cs-action`, from `origin/main` (e8438cc3). Uncommitted (no commit until the user says).
- Done: modules `CS_ACTION` / `CS_PERFORMANCE`, nav, routes, header titles, endpoints, query keys, types, API layer with a sample-data switch (`VITE_CS_MOCK`, default on, `false` calls the real API), query/mutation hooks, CS Action page (counters, category tabs, status/due/Mine filters, queue, task panel with contact form, schedule, escalate, complete, release/reopen, history, Sync new orders), CS Performance page (period, employee, follow-up cards, activity table, records sheet, Excel export, not-available list, definitions).
- Checks: `yarn build` passes, eslint and tsc clean for new files, `node --test tests/cs-action.test.mjs` 5/5. Only remaining tsc error is the existing `vite-plugin-eslint` typing in `vite.config.ts`.
- Update 5 Nov (evening): UI aligned to the Figma Follow-ups frames (read through Chrome; Figma MCP was rate-limited): KPI cards, Needs follow-up / My cases pills, Due/Reason filters, Figma table columns, "How did the call go?" dialog, customer-sheet timeline. New **Saved segments** switch beside Today's work: cards from the live `customer-segment` + `countOnly` routes, member list sheet, "Start follow-up" (sample data only; real API needs backend R10).
- Checked in Chrome on sample data: pages render, Pick up, Log call, contact timeline, segments with real counts. Not checked: Complete/Escalate/Schedule clicks, records drill-down, export, 360/768 widths.
- Update 5 Nov (later): queue rows and the drawer now show customer name, phone (tap to call, copy), service interest, orders, last order, language and area, via the existing `customer/{id}` route (interim; backend ask R2). Drawer footer reworked (one primary button, equal secondary buttons, Release below). History reads in plain words. Records sheet shows names.
- Checked in Chrome on sample data: Schedule, Escalate, Complete (refused until every service line is verified, form stays open), History tab, performance records drill-down, export error message, queue as cards at ~800 px. Not checked: true 360 px (Chrome will not shrink below ~800), export with a real backend, anything against the real CS API.
- Temp code to remove before any commit: the `TEMP-VISUAL-CHECK` line in `app/hooks/use-permission.ts`.
- Update 5 Nov (remaining UI, simple version): drawer "Why it's here" (purpose chip plus saved-segment and other source chips); Review request tasks with a "How did the review request go?" outcome dialog (sample data only, backend R12); CS Performance split into tabs A Direct Sales, B App non-assisted, C App assisted, D Follow-up, with an Order value / Net revenue switch and VAT label, built on a proposed response shape (backend R11). A tab whose section is unavailable shows the backend reason. Employee filter shows on tab D only. Review fixes applied before commit (pagination cap, drill-down employee, request ID follows the payload, Pick up race, customer lookup toast spam, Cambodia-time badges, dial numbers, blob export errors).
- Not built on purpose (backlog): linking an unregistered Chat lead to a customer, a create-support-task screen (R9), source/channel/service filters.
- NOT done: G4 browser run (needs a login whose role has the new modules; the backend does not list them yet, so the sidebar item and route stay hidden for everyone), real-API run, G5 review, G7 docs sync, mock-vs-real contract check on staging.
- Backend availability checked 5 Nov: see `pending-request-backend.md` item 18.
- Update 6 Nov (backend reply `CS_Admin_Portal_Implementation_Guide_5Nov2026.md`): types, API and sample data moved to the final contract. Queue and records read customer name, phone, owner name, bulk order number (link `/order?bulkOrderId=`) and service lines from the task (no per-row profile call; the open panel still reads the profile once). Contact and history paging use their own totals and server order. Segment "Start follow-up" calls `POST /actions` with `segmentId` and shows skipped results. Review dialog: reached rules, ticket UUID for negative feedback, no next date on terminal outcomes. My cases sends `open=true`. Scope checklist skips cancelled/refunded lines. Build, eslint and 10 tests pass. Still sample data.
- Update 6 Nov (backend 6 Nov replies): Complete is built from the task's `completeOptions` (no hard-coded list, management-only options hidden, note required, inline confirm before the permanent Sales decline, button hidden when empty). Resolution labels by purpose plus resolution in the panel and records. Sample data follows the same rules (409 when not taken, `converted` refused). Not built: P6 Chat `csAction` toast (waits for Chat branch merge), label catalog from `/performance/definitions` (local fallback map only), P7 switch-off (waits for api-dev 200).

## Gates (T3 · permissions and role modules, new route, new API client, shared nav/endpoint/query-key files, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Modules `CS_ACTION` and `CS_PERFORMANCE` (READ/UPDATE) in permission map, nav and role editor; `CS-OVERVIEW` unchanged
- [ ] Counters from an unfiltered request; Due Today and Overdue not summed
- [ ] List reads `data.items`, pagination from `data`, page cap 400 handled
- [ ] Null due shows "No deadline set"; unregistered lead shown; ids treated as opaque strings
- [ ] Registered/Existing tabs show the not-live info line; no unsupported filters shown
- [ ] 400/401/403/503 handled per handoff table; 503 shows backend reason, never zero
- [x] English only: the portal has no language switch (decided 6 Nov), so no Khmer keys
- [ ] Tested on staging backend (or mocks if R4 is not ready, noted in PR)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Update 6 Nov (browser check on sample data, Chrome, ADMIN login)
Checked at `localhost:5175` with `VITE_CS_MOCK=true`: queue shows name, phone, order number and service; Pick up and My cases; task panel; Complete form from `completeOptions` (note required, management-only option labelled); CS Performance activity table (Met, Under, No target, n/a, target 0 met); Set monthly targets sheet. Found and fixed one crash (sample data defined before use, white page). Not checked: Sales decline confirm click-through, Save targets click, 360 px width, any real API, a non-admin CS user. Sample flag lives only in the gitignored `.env` / `.env.testing`.

## Update 6 Nov (merge of main, review, Chat message)
- Merged `origin/main` into `feat/cs-action` (no conflicts). Chat `csAction` message added (`app/lib/cs-action-chat.ts`, intake panel, `/cs-action?task=<id>`), not checked in a browser.
- G5/G6 review (read-only) fixed: task panel and history no longer show another task's data while loading; duplicate error toasts removed (API client toasts once; export errors parsed); management actions need `CS_PERFORMANCE` UPDATE only; review "requested" on Telegram/chat is not counted as reached; Sync no longer says success when incomplete; contact save reloads on 409; ticket link encoded; sample data is now lazy-loaded and absent from a build with the flag off (checked).
- Not done: Sales decline and Save targets click-through, real API run, G7 to G11.

## Update 6 Nov (browser pass and api-dev probe)
- Sample-data browser pass on the merged branch: queue, Pick up, segments (members, Start follow-up), Complete with Sales decline confirm (completes as "Sales decline"), targets sheet. All worked.
- api-dev probe (read-only GET from the logged-in portal): CS routes answer 200, shapes match types, 4 staff, targets month shape, `resolutions` present, role editor lists the CS modules. Queue is empty (no tasks). Old `cs-overview` still 500 (`column "user_id" does not exist`), unrelated to this work.
- Not tested against real data: take, contact, complete, export, targets save (needs tasks; Sync new orders is a write on dev, not pressed).

## Update 6 Nov PM (real data, backend PM reply)
- Sample flag removed from local env; the pages run on the real api-dev. Queue is empty: a manager Sync (today only) created nothing, so Sync now has a window choice (Today, 3 days, 7 days).
- Built from the PM reply: Direct Sales counts with "Created by" and "Creator not recorded" (no money), scheduler status badge (hidden while the endpoint 404s), `valueStates` legend. Filters not sent (backend returns 400 until confirmed).
- Sync new orders is a write on shared dev data; the auto-mode check blocks Claude from pressing it, so the user presses it.
- Not tested on real tasks yet: take, contact, complete, export, targets save.
- Update 6 Nov (real data): api-dev answers 400 "note is required" on take. Decision: the user types the note. Pick up in the list now opens the task with a Take form (note required); Schedule and Escalate also ask for a note; Release/Reopen already did. No automatic notes. Real tasks exist after the user's Sync; scheduler badge shows "Automatic creation is off".
