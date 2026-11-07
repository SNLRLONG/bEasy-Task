# Portal: CS Action task panel and workflow

- **ID:** T-0105
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Worktree:** `bEasy/bEasy-portal-cs-action/` (shared by T-0104, T-0105, T-0106) · **Branch:** `feat/cs-action`
- **Tier:** T2 · new API writes (contacts, transitions, reconcile), management-only actions (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-05
- **Started / updated:** 2026-11-05 / 2026-11-05
- **Docs:** Plan `Tasks/_plans/Portal-cs-action-performance.md` · `Docs/tracking/v2/backend-response/CS_Admin_Portal_Handoff.md` (contract and response examples) · `Docs/tracking/v2/requirements/bEasy_CS_Action_and_Performance_Developer_Brief_V1.md`
- **Depends on:** T-0104 merged; backend branch on staging (R4)
- **Touches:** `app/components/cs-action/task-panel/*`, `app/hooks/mutations/use-cs-*`, `app/locales/*.json`, `tests/cs-action-panel*` (new files only; shared files belong to T-0104)

## Goal
Side panel for one task: detail, scope checklist, contact log, history, and every transition CS and management need, plus management Sync new orders.

## Progress (2026-11-05): UI phase on sample data
Built in one worktree for all three tasks so shared files do not conflict: `bEasy/bEasy-portal-cs-action/`, branch `feat/cs-action`, from `origin/main` (e8438cc3). Uncommitted (no commit until the user says).
- Done: modules `CS_ACTION` / `CS_PERFORMANCE`, nav, routes, header titles, endpoints, query keys, types, API layer with a sample-data switch (`VITE_CS_MOCK`, default on, `false` calls the real API), query/mutation hooks, CS Action page (counters, category tabs, status/due/Mine filters, queue, task panel with contact form, schedule, escalate, complete, release/reopen, history, Sync new orders), CS Performance page (period, employee, follow-up cards, activity table, records sheet, Excel export, not-available list, definitions).
- Checks: `yarn build` passes, eslint and tsc clean for new files, `node --test tests/cs-action.test.mjs` 5/5. Only remaining tsc error is the existing `vite-plugin-eslint` typing in `vite.config.ts`.
- NOT done: G4 browser run (needs a login whose role has the new modules; the backend does not list them yet, so the sidebar item and route stay hidden for everyone), real-API run, G5 review, G7 docs sync, mock-vs-real contract check on staging.
- Backend availability checked 5 Nov: see `pending-request-backend.md` item 18.

## Gates (T2 · new API writes (contacts, transitions, reconcile), management-only actions, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security (if auth/data/input)  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Take, Log attempt, Schedule, Escalate, Complete work with `expectedVersion` taken from the last server response
- [ ] Contact attempt uses a stable request ID across retries; new ID for a new attempt; follow-up Yes requires a future time with +07:00
- [ ] Scope checklist per service line; Complete `scope_verified` only after a phone + reached contact (server rule shown as message, not guessed)
- [ ] 409 refreshes the task and keeps typed input
- [ ] Release / Reopen / Not eligible only for management with CS_PERFORMANCE UPDATE and CS_ACTION UPDATE; report-only users see no edit buttons
- [ ] Contact history Next disabled when fewer than limit rows; history tab from `/history`
- [ ] Sync new orders loops `nextAfterId` while `mayHaveMore`, shows created/duplicate/ineligible reasons, management only
- [ ] Footer note: new contacts do not update the old CRM Last Contact
- [ ] After each mutation: refetch task, queue and counters
- [ ] Tested on staging backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
