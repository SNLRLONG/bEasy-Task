# Plan: CS Action dashboard and CS Performance (management)

Updated: 4 Nov 2026 · Planner: claude · Replaces the scope of T-0085 (`Portal-kpi-cs-performance`)

## Goal
Two simple portal modules, nothing else:
1. **CS Action** (`/cs-action`): one working queue for CS staff. Counters, one task list, one task panel.
2. **CS Performance** (`/cs-performance`): one management page with the follow-up KPIs backend can measure today.

The Figma (28 generic frames, section `1726:104530`) is too wide. This plan follows the requirement brief and the real backend contract in `backend-response/CS_Admin_Portal_Handoff.md`. Unified Inbox, CRM lists and the existing CS Overview (T-0058) are not rebuilt.

Sources: `requirements/bEasy_CS_Action_and_Performance_Developer_Brief_V1.md`, `backend-response/CS_Admin_Portal_Handoff.md` (branch `codex/cs-action-performance-v1`, feature flag off, migration not applied anywhere).

## Flow (exists / change / new)
| # | Step | Status |
|---|---|---|
| 1 | Paid App order or Direct Sale deposit becomes an order-level scope task | **new, backend only.** No scheduler: tasks appear only when management runs "Sync new orders" (`POST /actions/reconcile`) |
| 2 | CS opens CS Action, sees 4 counters and the task list; tabs by category | new |
| 3 | CS opens a task panel, presses Take (temporary handling, version check) | new |
| 4 | CS logs a call attempt (phone / telegram / in_app_chat; reached, no_answer, busy, wrong_number), optional next follow-up | new |
| 5 | Scope task: tick each service line verified (phone + reached only), then Complete with `scope_verified` | new |
| 6 | Unanswered: task stays open; Schedule a new due time; Escalate if stuck | new |
| 7 | Management: Release / Reopen a stuck task, see history | new |
| 8 | Management opens CS Performance: period + employee, follow-up cards, contact activity per employee, click a number to see records, export | new |
| 9 | Registered / Existing / Chat sales tasks and review tasks | **not available** (backend returns unavailable). Tabs show an empty state with the reason |

## Screens and states

### CS Action (`/cs-action`)
- Header: title, **Sync new orders** button (management only), filter bar.
- Counters (4): To Do, Due Today, Overdue, Completed Today. Loaded from an unfiltered queue request. Due Today and Overdue may overlap; no total card.
- Tabs: New Orders, Registered Users, Existing Customers, After-Service (`category`). Registered and Existing show an info line: "Task creation for this category is not live yet" until backend enables it.
- Filters (only the supported ones): status (todo, in_progress, waiting, completed), due (today, overdue), **Mine** toggle. Page size 25.
- Table columns: type, customer id or "Unregistered lead", order / ticket / conversation link, status, owner ("Handling: name" when taken), due (Cambodia time), escalated flag, last update. Rows sort by backend order; do not add client sort that hides pages.
- Task panel (side sheet): summary, scope checklist per service line (new orders), contact history (paged, Next disabled when fewer than limit), history tab (audit), actions (Take, Log attempt, Schedule, Escalate, Complete; management: Release, Reopen, Not eligible with note).
- States: loading skeleton, empty ("No tasks"), 400/403/404/409/503 per backend table. 409 refreshes the task and keeps typed input; contact retry keeps the same request ID. Feature off or setup missing returns 503: show the supplied reason, never zero.

### CS Performance (`/cs-performance`)
- Filters: month picker with custom range (from inclusive, to exclusive, max 366 days, Cambodia time), employee select.
- Follow-up cards: Due tasks, Completed due tasks, Overdue, Completion % (null when no due tasks shows "n/a", real 0 shows 0%).
- Contact activity table per employee: attempts, reached, unique customers, unregistered conversations. No team total of unique customers.
- Each number opens a records sheet (`metric=attempts|reached|due|completed|overdue`, same period and employee), with Export (XLSX, 5000 max, 400 means narrow the filter).
- "Not available yet" panel instead of sections A to C and Chat closing rate, review coverage, scope coverage, outreach without sale: one row each with the backend `reason`. Definitions panel from `/performance/definitions` (read-only, shows `not_configured`).
- No source/channel/service/first-repeat filters, no revenue, no charts.

## Data and API
Base `/api/admin/cs` (wrapper `{code,status,message,data}`; bigint ids may be strings, treat as opaque). Endpoints per the handoff table: `GET /actions`, `POST /actions/reconcile`, `GET /actions/:id`, `POST /actions/:id/transitions` (returns task directly), `POST /actions/:id/contacts`, `GET /actions/:id/history`, `GET /performance`, `/performance/definitions`, `/performance/records`, `POST /performance/records/export` (blob; check Content-Type).
- Always replace the task and `version` with the server response; never compute the version.
- After a mutation: refetch task detail, queue and counters.
- Pagination is in `data` directly (`page`, `limit`, `totalItems`); queue capped at page 400.
- Links: `customerId` to customer profile, `conversationId` to Chat, `ticketId` to ticket. `orderId` is a service-line id: no order link until the order-route contract is checked (gap 3).
- Permissions: new modules `CS_ACTION` (READ, UPDATE) and `CS_PERFORMANCE` (READ, UPDATE) checked server-side from the role table. Portal needs the same module keys in `app/lib/permission.ts`, the nav, and the role editor.
- Roles: report-only management cannot take or edit tasks (hide actions by `CS_ACTION UPDATE`).

## Tracking events
None. Admin portal; the brief defines no events.

## Gaps and decisions
1. **Module naming mismatch.** Portal uses `CS-OVERVIEW` / `MODULES` view-add-update-export; backend uses `CS_ACTION` / `CS_PERFORMANCE` with READ/UPDATE. Decision: add two new module entries mapped to those backend names; leave `CS-OVERVIEW` untouched. Needs backend confirmation of the exact permission strings and how the RBAC role editor lists them (request R1).
2. **No task display data.** Responses carry ids only: no customer name, phone, owner name, order number. Decision: show ids and resolve owner name from the already-loaded employee list; customer name by a lazy lookup only if an existing hook can do it cheaply. Request R2 for names and the bulk booking number on the task.
3. **Order link.** `orderId` is a service-line id, bulk-order route may not accept it. Decision: show text, no link, until verified (request R2).
4. **No scheduler.** Tasks exist only after reconcile. Decision: ship the management "Sync new orders" action (loops `nextAfterId` while `mayHaveMore`, window = since start of today, max 31 days) and flag it to backend as temporary (R3).
5. **Contacts do not update the old CRM Last Contact.** Decision: say so in the task panel footer; do not claim they sync.
6. **Due time is null** (policy not approved). Show "No deadline set" not "on time"; overdue null is not "met".
7. **T-0085 overlaps.** Decision: T-0085 is superseded by this plan; add a note there, do not start it.
8. **T-0058 CS Overview** keeps its page. Both pages sit under Customer Service in the nav.
9. **Policies not approved** (SLA, retry, review, revenue, VAT): no fields in the UI. Nothing hardcoded.
10. **Design.** No usable Figma frame for the simplified screens. Build with the RDS/shadcn primitives already in the portal; ask the designer to review the result, not to redraw first.
11. **Backend not deployed.** Branch only, flag off, migration unapplied. Tasks cannot be tested until staging has it (R4). Build against the response examples with mocks, run on staging when ready.

## Open questions (recommendation first)
1. Does the new CS Action replace the CS "My cases" idea from the Figma? **Yes**, one queue only.
2. Should CS (not management) see Sync new orders? **No**, management only, until a scheduler exists.
3. Retire `/customer-overview` (T-0058)? **No**, keep until management confirms CS Performance covers it.
4. Where in the nav? **Customer Service group**: "CS Action" (CS_ACTION READ) and "CS Performance" (CS_PERFORMANCE READ).

## Task breakdown
| Order | Task | Repo | Depends on | Tier |
|---|---|---|---|---|
| 1 | T-0104 `Portal-cs-action-foundation`: permissions, nav, routes, types, API hooks, queue list with counters, tabs and filters | portal | R1, R4 | T3 |
| 2 | T-0105 `Portal-cs-action-task-panel`: task panel, contacts, transitions, history, management release/reopen, Sync new orders | portal | T-0104 | T2 |
| 2 | T-0106 `Portal-cs-performance`: performance page, records sheet, export, definitions panel | portal | T-0104 (hooks, permissions) | T2 |

Parallel: T-0105 and T-0106 after T-0104 merges (T-0104 owns shared files; the other two touch separate folders, but both add query keys and endpoints, so T-0104 pre-adds all keys and endpoints to avoid conflicts).

## Out of scope
Management sections A to C (Direct Sales, App assisted/non-assisted), revenue and VAT, Chat closing rate, review requests, CRM segment ingestion, Chat outcome form, source/channel/service filters, targets, employee scores, Unified Inbox (T-0091), policy settings editor, live refresh/polling.
