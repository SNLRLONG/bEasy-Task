# CS Action and Performance: backend reply to the 6 November request

Date: 6 November 2026 · Branch: `codex/cs-action-performance-v1` · Replies to `CS_Action_Performance_Backend_Request.md` (6 Nov).
The 5 November guide stays authoritative for everything it answered. This file covers only the open items.

## Summary

| # | Status | Answer |
|---|---|---|
| N1 | **Done (code)** | Resolution names listed below. Each task now carries `completeOptions`; `/performance/definitions` has the full `resolutions` catalog. |
| B1 | Permissions **done (SQL)**, deploy **pending** | `sql/cs-action-permissions.sql` registers `CS_ACTION` and `CS_PERFORMANCE` (VIEW, UPDATE) so they appear in the role editor. Deploy to api-dev still needs the api-dev deployment profile. |
| N2 | **Adapter done (code)**, Chat call **after merge** | `services/csDashboard/chatIngestion.js`. Chat calls it after every successful details save, once `codex/telegram-customer-intake` merges. Unregistered leads stay out (see below). |
| B2 | Template **ready**, values **pending approval** | `sql/cs-action-scheduler-enable.sql` plus the cron command. Needs an approved operator account, lookback and cadence. |
| A2 | Pending | Needs a production read-only DB profile or a definitions-only export. Nothing changed. |
| E1 | Pending (business) | No policy values approved yet. Nothing invented; `/performance/definitions` keeps showing `not_configured`. |

## N1 Resolutions for `POST /actions/:id/transitions` `{action:"complete", resolution}`

| Purpose | `resolution` | Label to show | Manual? | Meaning |
|---|---|---|---|---|
| sales_follow_up | `lost` | Lost | Yes | Ordinary lost opportunity. Customer can be followed up again later. |
| sales_follow_up | `declined` | Sales decline (no further contact) | Yes | Customer asked for no more sales contact. **Permanent**: writes a suppression, future sales tasks for this customer are skipped with reason `declined`. Show a confirm dialog. |
| sales_follow_up | `not_eligible` | Not eligible | Yes, management only | Needs `CS_PERFORMANCE / UPDATE`. |
| sales_follow_up | `converted` | Converted | **No** | Set by the server when a verified paid/deposit purchase is found (on re-ingestion). Sending it returns **409** "Converted is set automatically…". |
| scope_confirmation | `scope_verified` | Scope verified | Yes | Needs a reached phone contact verifying every active line. |
| scope_confirmation | `not_eligible` | Not eligible | Management only | |
| review_request | `not_eligible` | Not eligible | Management only | |
| review_request | `review_received`, `declined` | Review received / Review declined | **No** | Come from a contact with `reviewOutcome` `received` / `declined`. Sending them in `complete` returns **409**. |
| support | `issue_resolved` | Issue resolved | Yes | Linked ticket must be RESOLVED first. |
| support | `not_eligible` | Not eligible | Management only | |

So the portal can **enable Complete for sales tasks** with `lost`, `declined` and (management) `not_eligible`.

Easiest: render the buttons from the task itself. Every task in list/detail/create/transition responses now has:

```json
"completeOptions": [
  {"resolution":"lost","label":"Lost","managementOnly":false,"meaning":"Ordinary lost opportunity. The customer can be followed up again later."},
  {"resolution":"declined","label":"Sales decline (no further contact)","managementOnly":false,"meaning":"Customer asked not to be contacted for sales. Permanent: future sales follow-ups are skipped."},
  {"resolution":"not_eligible","label":"Not eligible","managementOnly":true,"meaning":"Management closes work that should not be done."}
]
```

It is `[]` for completed tasks. Hide `managementOnly` options for users without `CS_PERFORMANCE / UPDATE` (the API returns 403 anyway). `GET /performance/definitions` → `data.resolutions` has the full catalog per purpose, including the `manual:false` ones, for labels in history/reports.

## B1 Permissions and deploy

- Role editor lists the `permissions` table. `CS_ACTION` / `CS_PERFORMANCE` were never registered there, so they did not appear. Run `sql/cs-action-permissions.sql` (safe twice; registers only, grants no role), or register them on the Roles & Permissions page with the same names.

| Module | VIEW | UPDATE |
|---|---|---|
| `CS_ACTION` | Open the queue and tasks | Work tasks: take, contact, schedule, complete |
| `CS_PERFORMANCE` | Report, records, export, all employees' targets | Management: release/reopen, `not_eligible`, save targets, reconcile |

  The API also accepts `READ` as a synonym of `VIEW`; the portal should use `VIEW`. ADMIN/SUPER_ADMIN always pass; `CUSTOMER_SERVICE` users can work tasks without a grant.
- Deploy to api-dev: still pending (needs the api-dev deployment profile; the repo script targets production). After deploy we will confirm `GET /api/admin/cs/actions?page=1&limit=1` returns 200 and tell you.

## N2 Chat save → CS task

**When:** after every successful `PATCH /api/admin/chat/conversations/:id/details`, once `codex/telegram-customer-intake` is merged with this branch. The call is added in `services/chat/conversationDetails.js` `saveDetails`, after the save commits:

```js
const { ingestChatLead } = require('../csDashboard/chatIngestion');
const csAction = await ingestChatLead(id, result, actorId); // never throws
return { ...result, csAction };
```

**Which chats become a task:** purpose `SALES` and (`followUpRequired = true` or outcome `INTERESTED_PENDING` or `BOOKED`). BOOKED re-checks an open task so it closes as `converted` when the purchase is verified. It goes through the normal `POST /actions` ingestion: one open sales task per customer (a second save attaches to the same task, `duplicate:true`), sales decline suppression, purchase check, test/blacklist skip. It tries `registered` first, then `existing`, using the CRM audience rules.

**Unregistered leads:** stay out of the queue on purpose. A sales task needs a verified customer for duplicate checks, suppression and purchase verification. As soon as Chat links the customer (`linkedUserId`), the next save creates the task. The PATCH response gets `csAction`:

```json
{"ingested":true,"taskId":"42","duplicate":false}
{"ingested":false,"skipped":true,"reason":"Chat lead is not linked to a bEasy customer yet"}
```

A permission or setup problem is returned as a skip reason; it never fails the Chat save. Showing unregistered leads in the queue would need a product decision (task without a customer) and is not built.

## B2 Scheduler

Route `/api/scheduler/cs-actions` is ready. To activate on api-dev (no code change):

1. The lead approves: **operator account** (ADMIN, or a role with `CS_PERFORMANCE / UPDATE`; we suggest a dedicated "CS Scheduler" user), **lookback** (suggested 1440 minutes) and **cadence** (suggested every 5 minutes; one run handles at most 10 bookings).
2. Run `sql/cs-action-scheduler-enable.sql` with those values on api-dev's database.
3. Create the cron job with the command in that file, using api-dev's host and its `CLOUD_SCHEDULER_SECRET`.

**Date:** right after the api-dev deploy (B1) and the approval above; we will confirm when the checkpoint (`cs_generation_state`) starts moving. Until then use "Sync new orders".

## A2 and E1

- **A2:** unchanged. Needs a production read-only DB profile or a definitions-only export plus environment ID mappings. Production is not touched.
- **E1:** unchanged. Waiting for management to approve scope-call SLA, retry timing, escalation threshold, sales-assistance window, review eligibility/timing, revenue and VAT treatment. Until then the related values stay `available:false` / `null`.

## Verification (6 Nov)

41 CS tests pass: 19 unit tests (including new resolution and Chat-adapter tests) and 22 PostgreSQL temporary-fixture tests run on a local PostgreSQL, including a new end-to-end test: Chat lead → one sales task → `completeOptions` → `converted` rejected → `lost` accepted → can be re-ingested. `sql/cs-action-permissions.sql` was run twice on both possible `permissions` table shapes with no duplicates. Nothing was run on beasy-dev or production.
