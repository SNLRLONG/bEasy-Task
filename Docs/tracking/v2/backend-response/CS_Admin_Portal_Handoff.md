# CS Action and Performance backend handoff

Date: 4 November 2026. Branch: `codex/cs-action-performance-v1`, based on local `origin/master` at `04649324`. Source: `bEasy_CS_Action_and_Performance_Developer_Brief_V1.docx` dated 3 November.

## Release status

This is the initial backend workflow, not the full brief's production release. It implements order scope tasks, linked support work, immutable contact attempts, temporary handling, audits, operational counters, bounded reconciliation and measured operational drill-down/export. It does not claim that financial attribution, automatic CRM/Chat opportunity ingestion or review-request automation is complete.

The feature is disabled by default. No migration, scheduler or feature flag was applied to any application environment. Database integration tests used temporary PostgreSQL fixtures only. The portal has not been modified.

## Implemented behavior

- Reuses `cs_cases` and `cs_contacts` and existing `orders`, `conversations`, `tickets` and audit infrastructure. Legacy CS rows have no category and are excluded from new queue counters; their historical contents are preserved.
- Confirmed paid App bookings and Direct Sale deposits can create one order-level scope task. Pending/failed payment cannot. Multiple service lines share that task.
- The event key is derived on the server from booking type and stable bulk order identity. A repeated confirmation returns the existing task, including after completion; it does not create another initial task.
- Each active service line must be verified in a reached phone call before scope completion. A fingerprint covers current service/schedule/access/add-on details; changed scope invalidates old verification. Explicit reconciliation of a changed booking reopens its existing task with audit history.
- Support tasks require an existing matching ticket. Closing Chat does not resolve the task. Explicit `issue_resolved` completion verifies that the linked ticket is resolved; it does not certify financial settlement or refund completion.
- An external Chat lead can have a support task without a customer ID. Customer association requires the existing Chat customer link, with conflicting identities rejected. This release does not perform account matching or merge customer histories.
- Taking a task marks temporary handling. Another employee cannot steal it; authorized management can release or reopen it. Version checks reject stale transitions.
- Every new contact is a separate immutable event. Multiple employees or multiple attempts by one employee are preserved. Retry keys prevent an HTTP retry from counting twice. Actor and contact time come from the server.
- A missing next follow-up does not finish a task. Follow-up Yes requires a future timezone-qualified timestamp. Scope verification requires explicit completion. Escalation remains open.
- Management sees due-cohort completion and new contact activity, with matching paginated records and Excel export. Missing financial policies and incomplete evidence return `available:false`, `value:null` and a reason.

## Existing flows left intact

No Customer/Registered Customer contact writer, order/payment handler, Chat send/close handler, review writer, segment editor or ticket transition was changed. In particular, the existing same-day last-contact merge remains compatible; its rows are not relabelled as complete attempt history.

New contacts are not automatically copied into the old last-contact read model in this release. The new dashboard and old CRM screen therefore expose different evidence until a reviewed compatibility integration is added. Do not tell the frontend that the old CRM Last Contact field automatically reflects every new event.

No outbound call, message or review request is sent by these endpoints. Reconciliation is explicit and bounded, rather than adding dashboard queries to checkout or Chat requests.

## Setup and permissions

1. Inspect staging table definitions, indexes, constraints and existing writers against `sql/cs-action-performance-v1.sql`. Install existing `sql/add-activity-audit.sql` first if its infrastructure is absent.
2. Review and manually run the additive CS migration in staging. It adds fields, source links, indexes, checks and audits, and allows a contact for an unregistered conversation. It does not rename/drop old columns or rewrite old contact evidence. It is transactional and uses a five-second lock timeout and thirty-second migration statement timeout. Separately review/run `sql/cs-action-performance-read-indexes.sql` outside a transaction for concurrent booking lookup/reconciliation indexes; inspect equivalent and invalid indexes before applying it. Staging metadata already showed the order ID, add-on order ID, legacy contact idempotency and CS closed-time indexes, which are reused rather than duplicated. No index was built by this review.
3. Confirm runtime table/sequence privileges and current role permissions. New modules follow the existing `RolePermission` lookup pattern: `CS_ACTION / READ` for action observation and `CS_ACTION / UPDATE` for non-CS staff workflow writes, `CS_PERFORMANCE / READ` for reports/definitions and management action observation, `CS_PERFORMANCE / UPDATE` for management release/reopen, ineligibility override and reconciliation. Built-in ADMIN/SUPER_ADMIN roles have management access; active CUSTOMER_SERVICE roles have action access. Role values are checked from the database, not the token or request body. Reconciliation and completion overrides also need action access because they create/change work. A report-only management grant does not allow taking or editing tasks.
4. Enable `CS_ACTION_DASHBOARD_ENABLED=true` only after staging verification. Unset/false leaves these routes unavailable without accessing new storage. There is no automatic migration or automatic scheduler activation.
5. Confirm operational settings before configuring a scheduler to call reconciliation. Each window uses updated-at times, so late payments and rescheduling can be picked up on a repeated window. There is no permanent cursor, historical backfill or durable event/outbox guarantee in this release.

Deploying the feature with a missing table/column returns the normal error wrapper with 503 on feature requests. It does not create tables at runtime or change existing endpoint responses. Do not treat that protection as a substitute for staging migration review.

## API contract

Base path: `/api/admin/cs`. Existing admin authentication and `res.success(data)` / `res.error(message,status)` wrappers apply. Success is `{code,status:true,message,data}`; error is `{code,status:false,message,data}`. Task/contact/source fields are camelCase on the wire; IDs from PostgreSQL bigint columns may be strings. Treat those IDs as opaque values.

| Method | Path | Purpose |
|---|---|---|
| GET | `/actions` | Paginated queue and counters |
| POST | `/actions` | Idempotent scope task creation or matching linked-ticket support task |
| POST | `/actions/reconcile` | Management-triggered bounded scope-task reconciliation |
| GET | `/actions/:id` | Task, contact page and up to 100 source links |
| POST | `/actions/:id/transitions` | Take, schedule, escalate, explicit complete, management release/reopen |
| POST | `/actions/:id/contacts` | Immutable contact attempt with retry key |
| GET | `/actions/:id/history` | Paginated existing audit records for task changes |
| GET | `/performance` | Operational measures and availability of other sections |
| GET | `/performance/definitions` | Review, revenue, attribution and operational policy explanations/readiness |
| GET | `/performance/records` | Paginated evidence behind available measures |
| POST | `/performance/records/export` | Excel evidence export, maximum 5000 rows |

Queue filters: `category=new_orders|registered|existing|after_service`, `status=todo|in_progress|waiting|completed`, `due=today|overdue`, `mine=true`, `page`, `limit`. Default page size 25, maximum 100. Queue page is capped at 400; narrow filters beyond that. Counters use the same supplied filters. Due Today and Overdue can overlap during the day. Category tabs do not imply that all category generators are enabled.

Performance filters: required ISO timestamps `from` inclusive and `to` exclusive, maximum 366 days; optional `employeeId`. Records additionally require `metric=attempts|reached|due|completed|overdue`, with page/limit. Export takes those same filters in its JSON body, excluding paging; it rejects more than 5000 matches instead of silently truncating. Unknown filters are rejected rather than ignored. Source/channel/service/first-repeat filters are not implemented yet; the frontend must not show them as functional.

Examples use illustrative IDs and dates, not deployment configuration.

Create scope task:

```json
{"category":"new_orders","purpose":"scope_confirmation","orderId":123}
```

Support task:

```json
{"category":"after_service","purpose":"support","customerId":456,"ticketId":"dc8024d4-410a-4f4b-bcb4-5f40b5d015ac"}
```

An unregistered lead supplies an existing `conversationId` instead of `customerId`, and its ticket must refer to the same conversation. A supplied customer and conversation must already be linked in Chat.

Take task (use the current returned version):

```json
{"action":"take","expectedVersion":1,"note":"Calling to confirm service details"}
```

Log unsuccessful attempt, keeping work open:

```json
{"requestId":"client-generated-stable-retry-id","channel":"phone","result":"no_answer","note":"No answer; retry scheduled","followUp":true,"nextFollowUpAt":"2026-11-05T10:00:00+07:00"}
```

Channels: `phone`, `telegram`, `in_app_chat`. Results: `reached`, `no_answer`, `busy`, `wrong_number`. Scope verification requires `phone` and `reached`; automated Chat responses are not logged as human calls. The public contact endpoint records the authenticated employee's deliberate manual action, not proof from a telephony provider. It does not accept a client-supplied employee or historical timestamp.

Log scope verification:

```json
{"requestId":"another-stable-retry-id","channel":"phone","result":"reached","note":"Scope, date, access, equipment, add-ons and restrictions confirmed for each line","checklist":[{"orderId":123,"verified":true},{"orderId":124,"verified":true}]}
```

Complete separately:

```json
{"action":"complete","expectedVersion":2,"resolution":"scope_verified","note":"All current service-line details confirmed"}
```

Use `schedule` with `dueAt` to change the next manual deadline. Use `escalate` to flag unresolved work. Use `release` / `reopen` only with management authorization. `not_eligible` requires a management override and explanatory note. Other permitted resolution integrations are gated until verified order/review/opportunity policies are implemented. A stale version or concurrent ownership conflict returns 409; refresh instead of overwriting.

Reconciliation:

```json
{"from":"2026-11-01T00:00:00+07:00","to":"2026-11-05T00:00:00+07:00","afterId":1,"limit":10}
```

Omit `afterId` for the start of a window; follow `nextAfterId` while `mayHaveMore=true`. Maximum batch 25; maximum window 31 days. Repeat overlapping windows from the beginning to pick up late commits. Individual ineligible bookings are returned with a reason; infrastructure failures do not get silently converted into success. A retry is safe because event keys deduplicate the resulting task.

## Definitions and financial readiness

`GET /performance/definitions` exposes the approved-rule gaps rather than inventing values. The frontend should render these explanations near the relevant cards or in a definitions panel:

- Review eligibility, preferred channel, first-request delay, retry interval/limit and opt-out treatment are `not_configured` with null values. Eligible customers must be selected fairly; explicit decline stops repeated prompting and negative feedback prioritizes recovery.
- Order value, earned/recognized revenue and cash received are different measures. Revenue recognition, VAT-inclusive/exclusive treatment, cancellations, refunds/partial refunds and primary Direct Sale owner governance need explicit decisions.
- The assistance window is null until approved. No implicit 30-day rule is enabled. A post-order scope/support/review contact must never retroactively assist the same order.
- Zero means a supported measured result has no matches. Null/unavailable requires its reason to be displayed. Pending work or observation windows are not finalized losses.

The new operational metrics count only version-2 contact events and categorized tasks. They are not the complete October report. Due completion uses tasks whose current deadline falls inside the selected period; completed-in-period is a separate concept. Staff filtering uses current task owner for task measures and event actor for contacts. Historical owner/deadline snapshots are audited but not reconstructed by this release's report. A customer may be counted under several employees; do not sum employee unique-customer counts into a team unique count.

## Work still required for the complete brief

1. Approve SLA/retry/escalation, per-service rebooking, suppression/frequency, attribution and post-purchase windows, review policy and revenue/owner rules.
2. Integrate CRM segment refresh with actual existing matchers, conversion/staleness/opt-out checks and a stable cross-source opportunity identity. Existing segment editors remain authoritative. Sales task ingestion intentionally returns unavailable while this contract is unresolved.
3. Integrate the separate Chat amendment for human response evidence, purpose, simplified outcome, lost reason, mandatory follow-up date and verified order/opportunity linking. Chat Booked alone must not recognize a sale. Implement audited, verified customer matching/merge and safe multi-source linkage.
4. Implement approved review-request generation and outcome lifecycle, receipt matching, decline/opt-out storage and negative-feedback issue linkage. Review task generation currently returns unavailable.
5. Implement Management sections A–C from reconciled booking/payment evidence, approved owner/assistance rules, cancellation/refund allocation and explicit VAT treatment. Add contact-without-sale cohorts, Chat closing rate and comprehensive scope/review coverage. Those measures are unavailable rather than estimated in the current response.
6. Add missing source/channel/service/customer-stage filters, richer action modal data and full source-link pagination as actual integrations become available. Corrected contact events/outcome-edit handling is not implemented; original attempts are immutable.
7. Complete staging and frontend end-to-end checks. Add an approved durable generation/scheduling mechanism before calling ingestion automatic or measuring all-order coverage.

## Performance and verification

All feature database work has a five-second statement timeout and 1.5-second lock timeout. This service admits at most two concurrent feature transactions per API process; further requests receive a retryable 503. List/record responses are paginated, exports capped, and reconciliation bounded. Read responses use a repeatable-read snapshot so their counts and matching rows do not observe different commits. There are no dashboard scans or outbound calls added to payment/Chat paths.

These safeguards reduce risk; they do not prove zero production impact. Production execution plans, cardinalities, indexes, migration lock behavior and sustained concurrent load have not been measured. Review those in staging before enabling this feature.

Tests cover migration repeatability on legacy-shaped CS tables, preserved legacy writes, pending payments, paid Direct Sale deposits, duplicate confirmations, multi-service checks, immutable/retried contacts, stale versions, ownership, management permissions, future follow-up, external-lead support resolution, material scope changes, unavailable policies and KPI/drill-down reconciliation. The optional PostgreSQL suite creates temporary tables/functions in a single connection and never installs the migration on application tables.

Verification on 4 November: 23 new CS checks passed, including PostgreSQL fixtures; 285 existing Direct Sale payment checks, 39 audit checks and 32 selected Telegram/last-contact checks passed. Existing suites skipped 453 other Direct Sale database checks and 19 audit database checks because their separate disposable database harnesses were not configured. The audit HTTP tests required rerunning outside the sandbox to permit their local test listener. Syntax checks and `git diff --check` passed. These results do not constitute production load testing or verification of the full remaining brief.

Run local unit checks:

```sh
node_modules/.bin/jest --config jest.cs-dashboard.config.js --runInBand
```

The PostgreSQL test requires explicit `BEASY_CS_TEMP_TEST=true`, reads test credentials from the local `.env`, and uses `BEASY_CS_TEST_HOST` (default existing staging private host). Confirm the intended staging connection before enabling it. Production must not be selected for these tests.

## Frontend response examples and screen behavior

The following are illustrative contract examples checked against the feature branch controller, serializers and query builders. They are not live staging or production responses. Examples show the fields the frontend should consume; deployments with older CS table extensions may return additional fields. Nested `details` remains JSON; only top-level database column names are converted to camelCase.

### Queue listing

`GET /api/admin/cs/actions?category=new_orders&page=1&limit=25` returns:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "items": [
      {
        "id": "42",
        "customerId": 456,
        "category": "new_orders",
        "purpose": "scope_confirmation",
        "eventKey": "scope_confirmation:ORDER:0000000123",
        "orderId": 123,
        "conversationId": null,
        "ticketId": null,
        "status": "in_progress",
        "ownerId": 7,
        "pickedAt": "2026-11-04T04:00:00.000Z",
        "dueAt": "2026-11-04T05:00:00.000Z",
        "priority": "normal",
        "note": "Calling to confirm service details",
        "closedAt": null,
        "resolution": null,
        "escalated": false,
        "version": 2,
        "createdBy": 9,
        "createdAt": "2026-11-04T03:00:00.000Z",
        "updatedAt": "2026-11-04T04:00:00.000Z",
        "details": {
          "checklist": [
            {
              "orderId": 123,
              "verified": false
            },
            {
              "orderId": 124,
              "verified": false
            }
          ],
          "scopeFingerprint": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          "dueAvailability": "Automatic deadline policy not approved"
        },
        "overdue": false
      }
    ],
    "page": 1,
    "limit": 25,
    "totalItems": 1,
    "counters": {
      "toDo": 0,
      "dueToday": 1,
      "overdue": 0,
      "completedToday": 0
    },
    "timeZone": "Asia/Phnom_Penh",
    "calculatedAt": "2026-11-04T04:00:00.000Z"
  }
}
```

Read rows from `response.data.items`; pagination is directly inside `response.data`, not `response.data.pagination`. Compute `totalPages = Math.ceil(totalItems / limit)`. For an empty result, expect `items: []`, `totalItems: 0` and zero counters. Keep filters visible and show an empty state.

Counters reflect the same filters as the list. Avoid sending `status=todo` when loading a set of all-status headline counters: that would also exclude completed rows from Completed Today. A separate unfiltered queue request can supply headline counters. Due Today and Overdue may overlap; do not add these cards together as unique tasks. `overdue` on a listing row can be null when `dueAt` is null; that does not mean an approved deadline was met.

### Task detail and contact history

`GET /api/admin/cs/actions/42?page=1&limit=25` returns:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "task": {
      "id": "42",
      "customerId": 456,
      "category": "new_orders",
      "purpose": "scope_confirmation",
      "eventKey": "scope_confirmation:ORDER:0000000123",
      "orderId": 123,
      "conversationId": null,
      "ticketId": null,
      "status": "in_progress",
      "ownerId": 7,
      "pickedAt": "2026-11-04T04:00:00.000Z",
      "dueAt": "2026-11-04T05:00:00.000Z",
      "priority": "normal",
      "note": "Calling to confirm service details",
      "closedAt": null,
      "resolution": null,
      "escalated": false,
      "version": 2,
      "createdBy": 9,
      "createdAt": "2026-11-04T03:00:00.000Z",
      "updatedAt": "2026-11-04T04:00:00.000Z",
      "details": {
        "checklist": [
          {
            "orderId": 123,
            "verified": false
          },
          {
            "orderId": 124,
            "verified": false
          }
        ],
        "scopeFingerprint": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        "dueAvailability": "Automatic deadline policy not approved"
      }
    },
    "contacts": [
      {
        "id": "81",
        "customerId": 456,
        "caseId": "42",
        "conversationId": null,
        "channel": "phone",
        "result": "no_answer",
        "purpose": "scope_confirmation",
        "said": null,
        "problem": "none",
        "nextFollowUpAt": null,
        "note": "No answer",
        "csId": 7,
        "contactedAt": "2026-11-04T04:01:00.000Z",
        "createdAt": "2026-11-04T04:01:00.000Z",
        "updatedAt": "2026-11-04T04:01:00.000Z",
        "idempotencyKey": "stable-retry-id",
        "requestHash": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
        "captureVersion": 2,
        "details": {
          "checklist": null,
          "scopeFingerprint": null
        }
      }
    ],
    "sources": [],
    "page": 1,
    "limit": 25
  }
}
```

`page`/`limit` paginate `contacts`. This response currently has no contact total or `hasMore`: disable Next when fewer than `limit` contacts return, otherwise allow the next page, which can be empty. `sources` returns at most 100 links and is not independently paginated yet. Do not label it a guaranteed complete source list. A source row exposes `id`, `caseId`, `sourceType`, `sourceId`, optional `segmentId`/`conversationId` and `createdAt`.

### Create and transition responses

`POST /actions` returns an object containing the task and duplicate indicator:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "task": {
      "id": "42",
      "customerId": 456,
      "category": "new_orders",
      "purpose": "scope_confirmation",
      "eventKey": "scope_confirmation:ORDER:0000000123",
      "orderId": 123,
      "conversationId": null,
      "ticketId": null,
      "status": "in_progress",
      "ownerId": 7,
      "pickedAt": "2026-11-04T04:00:00.000Z",
      "dueAt": "2026-11-04T05:00:00.000Z",
      "priority": "normal",
      "note": "Calling to confirm service details",
      "closedAt": null,
      "resolution": null,
      "escalated": false,
      "version": 2,
      "createdBy": 9,
      "createdAt": "2026-11-04T03:00:00.000Z",
      "updatedAt": "2026-11-04T04:00:00.000Z",
      "details": {
        "checklist": [
          {
            "orderId": 123,
            "verified": false
          },
          {
            "orderId": 124,
            "verified": false
          }
        ],
        "scopeFingerprint": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        "dueAvailability": "Automatic deadline policy not approved"
      }
    },
    "duplicate": true
  }
}
```

A duplicate is successful recovery of an existing event, not an error. Open/refresh the returned task.

`POST /actions/:id/transitions` returns the updated task **directly** as `response.data`; it does not return `{task: ...}`. For example, after scheduling:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "id": "42",
    "customerId": 456,
    "category": "new_orders",
    "purpose": "scope_confirmation",
    "eventKey": "scope_confirmation:ORDER:0000000123",
    "orderId": 123,
    "conversationId": null,
    "ticketId": null,
    "status": "waiting",
    "ownerId": 7,
    "pickedAt": "2026-11-04T04:00:00.000Z",
    "dueAt": "2026-11-05T03:00:00.000Z",
    "priority": "normal",
    "note": "Retry tomorrow",
    "closedAt": null,
    "resolution": null,
    "escalated": false,
    "version": 3,
    "createdBy": 9,
    "createdAt": "2026-11-04T03:00:00.000Z",
    "updatedAt": "2026-11-04T04:02:00.000Z",
    "details": {
      "checklist": [
        {
          "orderId": 123,
          "verified": false
        },
        {
          "orderId": 124,
          "verified": false
        }
      ],
      "scopeFingerprint": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "dueAvailability": "Automatic deadline policy not approved"
    }
  }
}
```

Always replace the displayed task/version with the returned value. The next transition must use that `version` as `expectedVersion`. Do not calculate a version locally. Logging a contact without a next follow-up may leave the task version unchanged; scheduling updates it.

### Contact save and retry response

`POST /actions/:id/contacts` returns:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "contact": {
      "id": "81",
      "customerId": 456,
      "caseId": "42",
      "conversationId": null,
      "channel": "phone",
      "result": "no_answer",
      "purpose": "scope_confirmation",
      "said": null,
      "problem": "none",
      "nextFollowUpAt": null,
      "note": "No answer",
      "csId": 7,
      "contactedAt": "2026-11-04T04:01:00.000Z",
      "createdAt": "2026-11-04T04:01:00.000Z",
      "updatedAt": "2026-11-04T04:01:00.000Z",
      "idempotencyKey": "stable-retry-id",
      "requestHash": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
      "captureVersion": 2,
      "details": {
        "checklist": null,
        "scopeFingerprint": null
      }
    },
    "replayed": false
  }
}
```

A retry with the same request ID and identical payload returns the same contact with `replayed: true`. Keep the request ID stable during network retries; create a new one for a genuinely new attempt. A reused ID with different contents returns 409. These example hash/provenance fields are server metadata; the frontend does not calculate or submit them. After saving, refresh task detail, queue counters and any open contact/performance view. A saved attempt alone does not mean scope is confirmed or a sale occurred.

### Management performance response

`GET /performance?from=2026-11-01T00%3A00%3A00%2B07%3A00&to=2026-12-01T00%3A00%3A00%2B07%3A00` uses an inclusive lower boundary and exclusive upper boundary. Build query strings with URLSearchParams/the existing request client so `+07:00` is encoded correctly. Representative `response.data` structure:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "period": {
      "from": "2026-10-31T17:00:00.000Z",
      "to": "2026-11-30T17:00:00.000Z",
      "toExclusive": true,
      "timeZone": "Asia/Phnom_Penh"
    },
    "calculatedAt": "2026-11-04T04:00:00.000Z",
    "sections": {
      "directSales": {
        "available": false,
        "value": null,
        "reason": "Primary accountable-owner governance, revenue recognition, refund allocation and VAT policy await approval"
      },
      "appNonAssisted": {
        "available": false,
        "value": null,
        "reason": "Approved sales-assistance window and complete qualifying human-contact evidence are required; absent legacy evidence is unknown"
      },
      "appAssisted": {
        "available": false,
        "value": null,
        "reason": "Approved sales-assistance window, qualifying human sales purpose and verified order linkage are required"
      },
      "followUp": {
        "dueTasks": 1,
        "completedDueTasks": 0,
        "overdueDueTasks": 0,
        "completionPercent": 0,
        "contactActivity": [
          {
            "employeeId": 7,
            "attempts": 1,
            "reached": 0,
            "uniqueCustomers": 1,
            "unregisteredConversations": 0
          }
        ],
        "chatClosingRate": {
          "available": false,
          "value": null,
          "reason": "Unique sales-opportunity identity, verified conversion and approved cohort/window not implemented yet"
        },
        "reviewCoverage": {
          "available": false,
          "value": null,
          "reason": "Review eligibility, request channel, cadence and opt-out rules await approval"
        },
        "scopeCoverage": {
          "available": false,
          "value": null,
          "reason": "Automatic confirmation ingestion and historical eligibility coverage are not established"
        },
        "outreachWithoutSale": {
          "available": false,
          "value": null,
          "reason": "Approved sales-opportunity identity, observation window and sales ingestion are required"
        }
      }
    },
    "definitions": {
      "contacts": "Only immutable capture_version=2 events; legacy daily contact rows are excluded from attempts",
      "dueTasks": "Current due_at in selected period; current owner filter. Historical deadline and owner snapshots are not reconstructed",
      "completion": "Explicitly resolved tasks among tasks whose current deadline is in the selected period; not completed-in-period count",
      "historicalCoverage": "Starts when this workflow is enabled. Missing earlier evidence is not zero."
    }
  }
}
```

The follow-up section contains measured fields directly; it does not have a section-level `available` flag. Its unsupported submetrics do have availability objects. Show `completionPercent: null` as not applicable when `dueTasks=0`; keep a real numeric zero as 0%. Use `contactActivity.employeeId` for employee grouping; individual contact rows expose the same actor as `csId`. Customer counts are distinct within each employee row and must not be added into a team unique count.

### Drill-down response and links

`GET /performance/records?metric=attempts&from=...&to=...&employeeId=7&page=1&limit=25` returns:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "items": [
      {
        "id": "81",
        "customerId": 456,
        "caseId": "42",
        "conversationId": null,
        "channel": "phone",
        "result": "no_answer",
        "purpose": "scope_confirmation",
        "said": null,
        "problem": "none",
        "nextFollowUpAt": null,
        "note": "No answer",
        "csId": 7,
        "contactedAt": "2026-11-04T04:01:00.000Z",
        "createdAt": "2026-11-04T04:01:00.000Z",
        "updatedAt": "2026-11-04T04:01:00.000Z",
        "idempotencyKey": "stable-retry-id",
        "requestHash": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
        "captureVersion": 2,
        "details": {
          "checklist": null,
          "scopeFingerprint": null
        }
      }
    ],
    "totalItems": 1,
    "limit": 25,
    "page": 1,
    "period": {
      "from": "2026-10-31T17:00:00.000Z",
      "to": "2026-11-30T17:00:00.000Z",
      "toExclusive": true
    }
  }
}
```

For `attempts`/`reached`, items are contact records. For `due`/`completed`/`overdue`, items are tasks. Use the exact same period and employee filter as the source card. There are no working financial drill-downs yet.

Use existing portal route helpers for links: `customerId` opens the customer profile, `orderId` is a service-line ID, `conversationId` opens Chat and `ticketId` opens the support ticket. The API does not supply resolved display names, customer phone numbers, hyperlinks or a top-level bulk booking number in these new task responses. Do not assume a bulk-order detail route accepts the service-line `orderId`; resolve the booking through the existing order API/route contract. `eventKey` is an internal deduplication key, not a public URL. Extra lookup/presentation integration may be needed for the full designer row.

### Reconciliation and history responses

`POST /actions/reconcile` returns:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "items": [
      {
        "orderId": 123,
        "taskId": "42",
        "duplicate": true,
        "available": true
      }
    ],
    "nextAfterId": 123,
    "mayHaveMore": false,
    "window": {
      "from": "2026-10-31T17:00:00.000Z",
      "to": "2026-11-04T17:00:00.000Z"
    },
    "warning": "Repeat the updated-at window from afterId=0 to reconcile late commits. No messages are sent. Scheduling and SLA policy must be approved before automatic activation."
  }
}
```

An ineligible reconciliation item instead contains `orderId`, `available:false` and `reason`. This is a management/operator workflow, not a request to run on every queue render.

`GET /actions/:id/history` returns `data: {items: [...], page, limit}`. Each item is an existing AuditChange record, with fields including `id`, `schemaName`, `tableName`, `recordId`, `operation`, `actorId`, `actorType`, `requestId`, `transactionId`, `changes`, `context` and `createdAt`. A changed field inside `changes` has `{before, after}` values. This history endpoint currently lists task changes, not a merged timeline of contacts and all source-link changes. There is no total count; use the same next-page convention as contact history.

### Definitions response

`GET /performance/definitions` returns `data.timeZone`, `data.reviewPolicy`, `data.revenuePolicy`, `data.salesAssistance`, `data.operationalPolicy` and `data.valueStates`. Review and revenue policies expose `status: "not_configured"`, `available:false`, explanations and null values, as described above. Fetch this response for the definitions/readiness panel; it is not a settings-save endpoint. The current release does not accept policy approval values through this endpoint.

### Errors, downloads and refresh behavior

An example 409 response is:

```json
{
  "code": "ERROR",
  "status": false,
  "message": "Task changed. Refresh before saving.",
  "data": null
}
```

HTTP status is authoritative. `code` is the project's response code (`SUCCESS` or `ERROR`), not the numeric HTTP status. Existing authentication middleware may return its older authentication error shape before this controller; use the portal's existing authentication handling.

| HTTP status | Frontend handling |
|---|---|
| 400 | Display validation/filter message; do not silently drop unsupported filters |
| 401 | Existing login/session-expiry behavior |
| 403 | Show insufficient permission; do not retry as another employee |
| 404 | Task/order/ticket missing; refresh navigation |
| 409 | Refresh current task; retain unsaved input. For a contact network retry, retain the same request ID and payload |
| 503 | Display the supplied unavailable/busy/setup/policy reason. Do not replace with zero or blindly retry indefinitely |
| 500 | Existing generic error behavior; avoid exposing server details |

`POST /performance/records/export` returns an XLSX binary download, **not** a success JSON wrapper. Request it with the existing authenticated download/blob helper. On failure it can return the normal JSON error wrapper; inspect Content-Type before saving a file. The file has CS records and Definitions sheets. An export exceeding 5000 matches fails with 400 and must be narrowed.

This backend does not provide WebSocket/SSE notifications or a polling interval. Refresh/revalidate relevant queries after mutations and use the application's existing refresh conventions. Do not promise live cross-tab updates or automatic refresh of the old CRM Last Contact field. Reuse existing tables, sheets, dialogs, pagination, selectors, loading states, permission handling and download utilities in the portal.
