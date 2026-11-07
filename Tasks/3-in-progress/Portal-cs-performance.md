# Portal: CS Performance (management)

- **ID:** T-0106
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Worktree:** `bEasy/bEasy-portal-cs-action/` (shared by T-0104, T-0105, T-0106) · **Branch:** `feat/cs-action`
- **Tier:** T2 · new page, new API calls, XLSX export of staff activity (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-05
- **Started / updated:** 2026-11-05 / 2026-11-05
- **Docs:** Plan `Tasks/_plans/Portal-cs-action-performance.md` · `Docs/tracking/v2/backend-response/CS_Admin_Portal_Handoff.md` (contract and response examples) · `Docs/tracking/v2/requirements/bEasy_CS_Action_and_Performance_Developer_Brief_V1.md`
- **Depends on:** T-0104 merged (permissions, endpoints, keys); backend on staging (R4)
- **Touches:** `app/routes/customer-services/cs-performance.tsx`, `app/components/cs-performance/*`, `app/hooks/query/use-cs-performance*`, `app/locales/*.json`, `tests/cs-performance*`

## Goal
One management page: period + employee filter, follow-up cards, contact activity per employee, records drill-down, XLSX export, a 'Not available yet' list with backend reasons, and a read-only definitions panel.

## Progress (2026-11-05): UI phase on sample data
Built in one worktree for all three tasks so shared files do not conflict: `bEasy/bEasy-portal-cs-action/`, branch `feat/cs-action`, from `origin/main` (e8438cc3). Uncommitted (no commit until the user says).
- Done: modules `CS_ACTION` / `CS_PERFORMANCE`, nav, routes, header titles, endpoints, query keys, types, API layer with a sample-data switch (`VITE_CS_MOCK`, default on, `false` calls the real API), query/mutation hooks, CS Action page (counters, category tabs, status/due/Mine filters, queue, task panel with contact form, schedule, escalate, complete, release/reopen, history, Sync new orders), CS Performance page (period, employee, follow-up cards, activity table, records sheet, Excel export, not-available list, definitions).
- Checks: `yarn build` passes, eslint and tsc clean for new files, `node --test tests/cs-action.test.mjs` 5/5. Only remaining tsc error is the existing `vite-plugin-eslint` typing in `vite.config.ts`.
- NOT done: G4 browser run (needs a login whose role has the new modules; the backend does not list them yet, so the sidebar item and route stay hidden for everyone), real-API run, G5 review, G7 docs sync, mock-vs-real contract check on staging.
- Backend availability checked 5 Nov: see `pending-request-backend.md` item 18.

## Gates (T2 · new page, new API calls, XLSX export of staff activity, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security (if auth/data/input)  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Period: month default and custom range, ISO with +07:00 via URLSearchParams, max 366 days validated client-side
- [ ] Cards use `sections.followUp`; `completionPercent` null shows n/a, 0 shows 0%
- [ ] Employee rows not summed into a team unique-customer count
- [ ] Records sheet uses the same period and employee as the clicked card; pagination Next convention
- [ ] Export is a blob download; JSON Content-Type treated as error; 400 over 5000 says narrow the filter
- [ ] Unavailable measures (direct sales, app assisted/non-assisted, chat closing rate, review, scope coverage, outreach) show the backend reason, never 0 or an estimate
- [ ] No source/channel/service/first-repeat filter, no revenue figures
- [ ] Definitions panel from `/performance/definitions`
- [ ] Hidden without CS_PERFORMANCE READ
- [ ] Tested on staging backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
