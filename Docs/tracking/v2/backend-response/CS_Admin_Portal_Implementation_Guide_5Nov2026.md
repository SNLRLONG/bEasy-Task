# CS Admin Portal Implementation Guide

Updated 5 November 2026. Audience: admin portal developers and QA. Backend branch: `codex/cs-action-performance-v1`. Portal branch in the request: `feat/cs-action`, tasks T-0104, T-0105 and T-0106.

Implement the existing **CS Action** (`/cs-action`) and **CS Performance** (`/cs-performance`) pages against the backend contract below. Reuse the application's existing navigation, permissions, API client, tables, cards, sheets, dialogs, forms, pagination, date controls and export helpers. Keep the Customer/Registered Customer segment editors and Unified Inbox as their existing sources of truth.

## Release gate before real API testing

The code changes are local on the backend feature branch; they have not been pushed or deployed by this work. Staging `beasy-dev` has the earlier schema and read indexes installed; run the updated manual setup to add database scheduler configuration. Updated backend code needs no extra CS environment flag. Deployment and live authenticated responses have not been verified. Scheduled generation remains disabled until configured in the database. Keep real mutation controls gated until the backend deployment is confirmed and an authenticated `GET /api/admin/cs/actions?page=1&limit=1` returns 200. A 503 does not mean there are zero tasks.

Sample mode may continue for design review with a visible Sample data banner. Do not silently fall back to sample numbers after a real API error. Financial sections stay Unavailable until approved policies/evidence exist. Do not remove the sample banner until the page is actually using verified live responses.

Staging segment cleanup removed seven unused test definitions after backup. Two protected definitions remain: Customer id 2, Test (two coupon links), and Registered id 8, test reg customer segments (one coupon link). No coupons/banners were changed. Production definitions have not been imported; the exact source export and environment-ID mappings are still pending. These remaining test rows must not be presented as the production segment seed.

## Frontend implementation checklist

| Area | What to implement |
|---|---|
| Navigation | Keep the two existing pages and existing role-aware navigation patterns; do not replace the old CS Overview route or rebuild Chat/CRM editors. |
| Shared API integration | Use the existing authenticated client and SUCCESS/ERROR wrapper. Match field names exactly. Treat task/contact bigint IDs as opaque strings or numbers without lossy conversion. |
| Queue | Read rows, counters and pagination from GET /actions. Use category tabs and open=true for open/My cases views. Show the server's customer/owner names, actual service lines, bulk booking reference and last-contact summary. |
| Task sheet | Load task detail and independently paginated contacts/sources. Use returned order metadata; do not request the customer profile once for every row. |
| Task actions | Take before contact or scheduling. Submit expectedVersion from the current task. Refresh the task on 409 instead of overwriting another employee. |
| Contact modal | Use a stable requestId for a single attempt and all its retries. New attempts use new IDs. A saved unanswered call leaves the task open. Follow-up Yes needs a future timezone-qualified date/time. |
| Scope verification | Show the current active service-line checklist. Log a reached phone call with verified lines, then explicitly complete as scope_verified. Do not classify a post-purchase scope call as assisted sales. |
| Saved segments | Use existing saved names, audiences and backend filters/count adapters. Start follow-up through POST /actions; handle duplicate and skipped results. Do not build another client-side segment matcher. |
| Support | Start from an existing ticket and link its UUID. Use the existing ticket/Chat workflow, rather than another ticket creation system. |
| Review workflow | Log reviewOutcome; respect verified receipt, decline, pause and support-ticket requirements below. This is manual logging, not automatic review sending. |
| Performance | Render four sections. Use available flags/reasons, actual measured operational counts and null handling. Populate staff names from the supplied fields/staff endpoint. |
| Targets | Implement management-only monthly GET/PUT and comparisons; preserve zero and null. No weighted score or ranking. |
| Exports | Use the existing authenticated blob download helper. Retain exact period/employee/metric filters. Handle JSON errors instead of downloading them as XLSX. |
| Refresh | Revalidate queue/counters, open detail/history and relevant performance queries after successful mutations. No WebSocket/SSE or automatic cross-tab update contract is provided. |

## Common response handling

In this guide, `body` means the parsed HTTP response body. For Axios, this is normally the HTTP response object's `.data`; do not accidentally unwrap twice.

- Success: `body.status === true`, domain payload at `body.data`.
- Queue/record rows: `body.data.items`. Pagination is directly in that payload, not a nested pagination object.
- Create: `body.data.task`; may be null for a skipped result. `duplicate:true` is successful reuse of the same task.
- Transition: the updated task is directly `body.data`, not `body.data.task`.
- Contact save: `body.data.contact`; `replayed:true` means the same attempt was recovered.
- Detail: `body.data.task`, `contacts`, `sources` and their paging fields.
- Target GET/PUT: `body.data.items`.
- Existing CRM segment-member counts keep their existing Customer-page response/adapter; they do not adopt the new CS queue shape.

Examples below are illustrative and show UI-relevant fields; the server may return additional metadata. They are not live staging responses. Never submit server-only actor IDs, hashes or timestamps from these examples as mutation fields.

## Full queue response example

`GET /api/admin/cs/actions?category=new_orders&open=true&page=1&limit=25`

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "items": [
      {
        "id": "42",
        "customerId": 3526,
        "customerName": "Example Customer",
        "customerPhone": "85512345678",
        "category": "new_orders",
        "purpose": "scope_confirmation",
        "status": "in_progress",
        "ownerId": 7,
        "ownerName": "Example CS",
        "orderId": 123,
        "bulkOrderId": "0000000123",
        "conversationId": null,
        "ticketId": null,
        "pickedAt": "2026-11-05T03:58:00.000Z",
        "dueAt": null,
        "closedAt": null,
        "resolution": null,
        "escalated": false,
        "reviewPaused": false,
        "version": 2,
        "serviceLines": [
          {
            "orderId": 123,
            "serviceName": "Instant Cleaning",
            "scheduledAt": "2026-11-06T03:00:00.000Z",
            "status": "ACCEPTED",
            "paymentStatus": "PAID"
          }
        ],
        "lastContact": {
          "result": "no_answer",
          "attempts": 1,
          "at": "2026-11-05T04:00:00.000Z"
        },
        "details": {
          "checklist": [
            {
              "orderId": 123,
              "verified": false
            }
          ],
          "dueAvailability": "Automatic deadline policy not approved"
        },
        "overdue": null
      }
    ],
    "page": 1,
    "limit": 25,
    "totalItems": 1,
    "counters": {
      "toDo": 0,
      "dueToday": 0,
      "overdue": 0,
      "completedToday": 0
    },
    "timeZone": "Asia/Phnom_Penh",
    "calculatedAt": "2026-11-05T04:01:00.000Z"
  }
}
```

Use `Math.ceil(totalItems / limit)` for total pages. Empty success has items:[], totalItems:0 and measured zero counters. When dueAt is null, display No deadline set; a null row overdue flag is not evidence of timely completion. The counter filters match the list: an open-filtered request excludes completed tasks, including from Completed Today. Request an appropriate all-state scope for all-state headline counters. Due Today and Overdue can overlap; do not add them as unique tasks.

## Full detail response example

`GET /api/admin/cs/actions/42?page=1&limit=25&sourcePage=1&sourceLimit=25`

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "task": {
      "id": "42",
      "customerId": 3526,
      "customerName": "Example Customer",
      "customerPhone": "85512345678",
      "category": "new_orders",
      "purpose": "scope_confirmation",
      "status": "in_progress",
      "ownerId": 7,
      "ownerName": "Example CS",
      "orderId": 123,
      "bulkOrderId": "0000000123",
      "conversationId": null,
      "ticketId": null,
      "pickedAt": "2026-11-05T03:58:00.000Z",
      "dueAt": null,
      "closedAt": null,
      "resolution": null,
      "escalated": false,
      "reviewPaused": false,
      "version": 2,
      "serviceLines": [
        {
          "orderId": 123,
          "serviceName": "Instant Cleaning",
          "scheduledAt": "2026-11-06T03:00:00.000Z",
          "status": "ACCEPTED",
          "paymentStatus": "PAID"
        }
      ],
      "lastContact": {
        "result": "no_answer",
        "attempts": 1,
        "at": "2026-11-05T04:00:00.000Z"
      },
      "details": {
        "checklist": [
          {
            "orderId": 123,
            "verified": false
          }
        ],
        "dueAvailability": "Automatic deadline policy not approved"
      }
    },
    "contacts": [
      {
        "id": "81",
        "customerId": 3526,
        "caseId": "42",
        "csId": 7,
        "employeeName": "Example CS",
        "channel": "phone",
        "result": "no_answer",
        "purpose": "scope_confirmation",
        "reviewOutcome": null,
        "note": "No answer; work remains open",
        "contactedAt": "2026-11-05T04:00:00.000Z",
        "nextFollowUpAt": null,
        "details": {
          "checklist": null,
          "scopeFingerprint": null,
          "supportCaseId": null
        }
      }
    ],
    "page": 1,
    "limit": 25,
    "totalItems": 1,
    "hasMore": false,
    "contactOrder": "contactedAt DESC, id DESC",
    "sources": [],
    "sourcePage": 1,
    "sourceLimit": 25,
    "sourceTotalItems": 0,
    "sourceHasMore": false,
    "sourceOrder": "id ASC"
  }
}
```

The common page/limit/totalItems/hasMore fields describe contacts. Sources have independent paging fields. History is a separate endpoint with its own total/hasMore and order. Keep server ordering. Use login user.id to compare ownerId/csId, not cleaner IDs or employee display names.

## Empty, unavailable and error states

These states are different and must remain different:

| State | UI behavior |
|---|---|
| Measured zero | Show 0, if the supported endpoint successfully measured no matching records. |
| available:false | Show Unavailable or Not configured and the server reason; disable unsupported drill-downs/actions. |
| completionPercent:null with no due base | Show n/a, not 0%. |
| target field:null | Show No target, not 0. A numeric target 0 is valid. |
| Missing display name | Use a neutral fallback and retain the ID; do not guess which similarly named employee owns it. |
| Loading | Reuse the existing skeleton/loading components; do not show temporary zero KPI values. |
| 503 | Show setup/busy/policy reason with controlled retry; do not turn the response into an empty queue. |

Example conflict error (HTTP 409):

```json
{
  "code": "ERROR",
  "status": false,
  "message": "Task changed. Refresh before saving.",
  "data": null
}
```

HTTP 400 is validation/unsupported filter input; 401 uses existing session handling; 403 is denied access; 404 means a missing record; 409 needs refresh/retry recovery; 422 is used for past-month target changes; 503 is unavailable/busy/setup/policy. Existing authentication middleware may return its older error shape before the CS controller, so retain the portal's existing authentication adapter.

## Route links and form behavior

Use the server's bulkOrderId with the portal's existing order-detail route helper. orderId is a service-line ID and must not be substituted for a bulk booking number. Use customerId, conversationId and ticketId with existing profile, Chat and ticket navigation. Do not parse eventKey into a link. Mark cancelled/refunded service lines appropriately and only verify active lines in the scope checklist.

Keep Schedule and Log contact actions available only when the authenticated user owns an open task. Management release/reopen also need server authorization. UI visibility is an aid, not a substitute for handling 403/409. Refresh after take/schedule/contact/complete/release/reopen; scheduling may update the task version even though logging an unscheduled contact may not.

For review eligibility use the task's **top-level reviewPaused** field, which reflects current linked issue state. details.reviewPaused is a historical marker and is not the authoritative current eligibility flag. Terminal or paused review outcomes must not schedule another review prompt.

Financial unavailable objects must remain unavailable. No sales-assistance window, VAT formula, partial-refund rule or revenue recognition basis should be invented in the portal. The review/policy definitions endpoint supplies explanations for the readiness panel; it is not a policy-save API.

## Manual synchronization and downloads

Sync new orders is a management/operator repair action, not a request on every queue render. Begin reconciliation without afterId (do not send 0); follow nextAfterId only while mayHaveMore is true. Keep a fixed from/to window while following batches. Show per-item skip reasons and refresh the queue when finished. Long-term scheduling belongs to backend deployment configuration, not a browser interval that invokes reconcile.

An XLSX export is a binary response, not the JSON success wrapper. Keep selected period/employee/metric filters unchanged. On non-XLSX Content-Type, handle the JSON error rather than saving it as a spreadsheet. The export cap is 5000 records; narrow the filters when exceeded. Export uses core evidence fields; paginated JSON records carry the richer display metadata.

## Frontend QA checklist

- No per-row customer profile fetch for queue names, phone numbers, service lines or bulk references.
- Empty lists and failed/unavailable requests render different states.
- My cases with open=true excludes completed work; unsupported filter combinations are rejected visibly.
- Contact/source/history pagination uses the correct independent totals and ordering.
- Two clients cannot silently overwrite handling or stale task versions.
- A network retry of one attempt does not create another contact.
- No answer leaves work open. Scope completion requires reached phone verification of all current lines.
- The same customer from two saved segments has one open sales task with both sources.
- Skipped/converted/opted-out results do not create fake queue rows.
- Review receipt cannot close without a real order-linked rating; decline prevents more requests.
- Negative feedback links an existing open ticket, pauses prompts and exposes supportCaseId without duplicate ticket creation.
- Target 0, cleared null, missing target and null actual completion are handled distinctly.
- CS VIEW without management UPDATE cannot read another employee's targets or operational report rows.
- Monthly reports include matching targets; allowed custom/multiple-month ranges have target:null.
- Existing completionPercent is 0–100; future D4 ratio fields are 0–1. Do not multiply the former by 100 again.
- A sample-data banner is never removed before real API readiness is confirmed.

The following backend response is included as the detailed route, field and rollout contract. Its yes/partial/later statuses are authoritative; a local implementation is not a live deployment.

## Response by item

| Item | Answer | Result and date |
|---|---|---|
| A1 Clean segments | Yes, with protected references retained | 5 Nov: backed up all nine staging definitions and deleted seven unused test rows in a transaction. Kept two coupon-linked rows exactly as allowed by the request; details below. No coupons/banners were changed. |
| A2 Copy production | No, needs exact source access | 5 Nov: production Chrome is signed in and shows ten Customer segments, but the UI does not expose their complete JSON definitions. Copied: customer 0, registered 0; no owner chosen; no skipped-definition list can be certified yet. Need a production read-only DB profile or definitions-only export and reference checks. Production was not modified. |
| B1 Enable API | Partial | 5 Nov: installed both additive CS schema migrations and concurrent read indexes on `beasy-dev`. Three read indexes are valid. The extra CS feature flag has now been removed from the updated code. The new code is not deployed and a live 200 was not verified; SQL alone does not update the running application. The repository deployment script targets production; the api-dev deployment/service profile is still needed. |
| B2 Scheduled generation | Yes, code; activation later | 5 Nov: added an authenticated scheduler route with a durable checkpoint and bounded batches. Sync new orders remains an interim/manual repair path. Activation date/cadence needs deployment access and approved operator/lookback settings; no date is invented. |
| C1 Display fields | Yes | 5 Nov: task queue/detail/create/transition and paginated task drill-down responses include customerName, customerPhone, ownerName, bulkOrderId and actual serviceLines. Batches avoid one customer request per row. |
| C2 Last contact | Yes | 5 Nov: task display includes lastContact or null, with result, attempts and at, from immutable new workflow events only. |
| C3 Pagination and history | Yes | 5 Nov: totals/hasMore, independent source pagination and explicit ordering. History includes task, contact and source-link audits. |
| C4 Open filter | Yes | 5 Nov: open=true excludes completed tasks. Combined status=completed returns 400 instead of a confusing empty result. |
| C5 Identity and names | Yes | 5 Nov: ownerId and contact csId are authenticated admin user IDs, matching login user.id. Added staff list and employeeName/ownerName. |
| D1 Segment follow-up | Yes, manual ingestion | 5 Nov: POST /actions supports existing/registered sales_follow_up with segmentId. One open sales task per customer across segment/conversation sources; ineligible, suppressed or converted customers return a skip reason. Automatic CRM refresh and the separate Chat save adapter are not activated by this change. |
| D2 Support creation | Yes, path confirmed | 5 Nov: CS starts support work from an existing ticket. A negative review outcome creates/links the corresponding support task using an existing open ticket. No generic automatic complaint/low-rating task generator is enabled. |
| D3 Review tasks/outcomes | Yes, manual workflow with contract change | 5 Nov: completed-order review tasks, verified receipt, decline suppression and negative-feedback pause/link are implemented. Negative feedback requires ticketId for a matching existing open ticket; use the existing ticket workflow first. No automatic review sending or policy-based generation is enabled. |
| D4 Performance shape | Yes, contract accepted with clarifications below | 5 Nov: A–C and unconfigured extras stay unavailable. Employee rows now carry due counts and completionPercent. Financial computation remains later, after E1. |
| D5 Monthly employee targets | Yes | 5 Nov: separate per-employee/month table, GET/PUT, validation, past-month protection, audit and report integration. Old team-level CS Overview targets are untouched. |
| E1 Policies | Later, business decision | No approved values were supplied. Explanations/readiness remain exposed; no default SLA, attribution window, review cadence or revenue/VAT/refund treatment was invented. |

Dates above identify code/setup work completed on 5 November, not a release promise. Live activation remains pending. Cleanup removed four Customer and three Registered definitions. Staging now has one protected Customer and one protected Registered definition; copied from production: zero for each audience. The backups and setup/cleanup receipts are local private operational files, not frontend data files.

Retained staging references: Customer segment **id 2, Test**, used by two coupons; Registered segment **id 8, test reg customer segments**, used by one coupon. Neither is used by a banner. They were left in place rather than detaching or changing another feature's configuration. The seven unused test names were removed; the source production definitions and environment reference mappings are still required before seeding.

## Routes and permissions

All routes below are under `/api/admin/cs` unless specified. Existing success/error wrappers remain unchanged. HTTP success body is `{code:"SUCCESS",status:true,message:"Success",data:...}`. IDs from bigint task/contact columns may be strings; retain them as opaque IDs.

| Method | Path | Data shape |
|---|---|---|
| GET | `/actions` | `{items,page,limit,totalItems,counters,timeZone,calculatedAt}` |
| POST | `/actions` | `{task,duplicate}` or `{task,duplicate,skipped:true,reason}` |
| GET | `/actions/:id` | Task plus independently paginated contacts/sources, described below |
| POST | `/actions/:id/transitions` | Updated task directly as data |
| POST | `/actions/:id/contacts` | `{contact,replayed,supportCaseId?}` |
| GET | `/actions/:id/history` | `{items,page,limit,totalItems,hasMore,order}` |
| GET | `/staff` | `{items:[{id,name}]}` for active CS users, up to 500 |
| GET | `/targets?month=2026-11` | Monthly active-CS target rows |
| PUT | `/targets/:month` | Same shape as target GET after atomic save |
| GET | `/performance` | Existing period/sections/definitions, with enriched employee activity |
| GET | `/performance/definitions` | Policy explanations/readiness; manual review workflow availability |
| GET | `/performance/records` | Existing paginated metric evidence, with display fields for JSON task/contact records |
| POST | `/performance/records/export` | Existing XLSX download; 5000-row cap |
| POST | `/actions/reconcile` | Existing management/operator repair batch |
| POST | `/scheduler/cs-actions` | Actual path is `/api/scheduler/cs-actions`; existing cloudScheduler secret authentication |

Management read accepts existing `CS_PERFORMANCE / READ` and portal `CS_PERFORMANCE / VIEW`. Management writes require `CS_PERFORMANCE / UPDATE`. CS action reads accept READ/VIEW/UPDATE or the active CS role; writes retain action authorization. Built-in ADMIN/SUPER_ADMIN behavior remains. A report-only grant does not allow task changes or target saves. CS users without management read cannot see other employees' targets.

If a CS-role user receives VIEW/READ but not management UPDATE, target GET and operational report/drill-down are restricted to that authenticated employee; requesting another employee returns 403. Such a grant does not expose other employees' targets. Existing ordinary CS users without VIEW/READ still receive 403 for management endpoints.

## C1 and C2 Task display

Representative fields added to a task; names/services can be null when their original records lack them:

```json
{
  "id": "42",
  "customerId": 3526,
  "customerName": "Example Customer",
  "customerPhone": "85512345678",
  "ownerId": 7,
  "ownerName": "Example CS",
  "orderId": 123,
  "bulkOrderId": "0000000123",
  "serviceLines": [
    {"orderId":123,"serviceName":"Instant Cleaning","scheduledAt":"2026-11-06T03:00:00.000Z","status":"ACCEPTED","paymentStatus":"PAID"}
  ],
  "lastContact": {"result":"no_answer","attempts":2,"at":"2026-11-05T03:00:00.000Z"}
}
```

`orderId` is still a service-line ID. Use `bulkOrderId` with the existing bulk-order route helper. Do not parse eventKey as a route. `serviceLines` includes each line's current status/payment state so cancelled/refunded work can be labelled correctly. Support/sales tasks without a linked order have null bulkOrderId and an empty serviceLines array. `lastContact.attempts` is an event count for this task, not a unique-customer count or verified telephony-provider count. Legacy same-day merged contacts are excluded.

Queue accepts the existing filters plus `open=true|false`. Example: `GET /actions?mine=true&open=true&page=1&limit=25`. Open excludes completed; it does not mean only todo. Counter filters still match the list: an open-filtered response has no Completed Today rows, so use an appropriate all-state request if the page needs all-state headline counters.

## C3 Detail pagination and history

`GET /actions/42?page=1&limit=25&sourcePage=1&sourceLimit=25` returns:

```json
{
  "task": {"id":"42","version":3},
  "contacts": [],
  "page": 1,
  "limit": 25,
  "totalItems": 0,
  "hasMore": false,
  "contactOrder": "contactedAt DESC, id DESC",
  "sources": [],
  "sourcePage": 1,
  "sourceLimit": 25,
  "sourceTotalItems": 0,
  "sourceHasMore": false,
  "sourceOrder": "id ASC"
}
```

The example task is abbreviated. `page/limit/totalItems/hasMore` describe contacts, independently of sources. Both page sizes default to 25 and cap at 100. Contact rows also expose employeeName and csId. Keep server ordering; no client sort is required. History uses createdAt DESC, id DESC and now includes existing audit records for cs_cases, cs_contacts and cs_case_sources associated with this task. Target changes use the same audit_changes infrastructure, with month/employee in audit context; they are not falsely attached to an unrelated task.

## D1 Saved-segment follow-up

```json
{"category":"existing","purpose":"sales_follow_up","customerId":3526,"segmentId":12}
```

Use registered with a registered segment; existing with a customer segment. The existing CRM audience/matching helpers verify the definition instead of creating a second filter engine. Every source is retained. A second matching segment attaches to the same open sales task. A verified conversationId can also be supplied; its existing customer link must match. The separate Chat-save implementation must call this ingestion after saving appropriate details; no unrelated Chat branch was merged here.

Skip example (inside the success wrapper's data):

```json
{"task":null,"duplicate":false,"skipped":true,"reason":"Customer no longer matches saved segment"}
```

A verified purchase after an open task began resolves that sales task as converted when ingestion checks it again, and skips new outreach for that opportunity. This is verified operational closure, not proof of sales assistance or revenue attribution. Registered customers who already purchased are skipped. Sales decline is an explicit no-further-sales-contact resolution and writes suppression; use lost for an ordinary qualified loss that is not a permanent opt-out. The frontend must label that distinction clearly. Automatic refresh cadence and broader contact-frequency policy remain pending E1.

## D2 and D3 Manual support and review flow

Support starts from an existing ticket using the existing POST /actions contract. Ticket IDs are UUIDs, not the displayed #TK number. No new support-create screen is required if the portal adds a contextual action on an existing ticket.

Manual review task:

```json
{"category":"after_service","purpose":"review_request","orderId":123}
```

The server verifies that all active booking lines are completed and that there is no existing order-linked review or active customer review suppression. One stable order-level review event key prevents repeated creation, including after completion. If an existing review is found, ingestion skips a new request and satisfies an existing open review task where applicable. It does not send a message. This staff-initiated workflow is distinct from automatically selecting review cohorts under an approved eligibility/timing policy.

Review contact example:

```json
{"requestId":"stable-review-attempt-id","channel":"phone","result":"reached","reviewOutcome":"requested","note":"Asked customer for an in-app review"}
```

| reviewOutcome | Behavior |
|---|---|
| requested | Append the contact; keep task open. A phone request requires a reached customer. |
| no_response | Append an unreached contact; keep work open, never silently mark lost. |
| received | Requires a real order-linked rating; then closes task as review_received. |
| declined | Closes task and suppresses future review requests for that customer. |
| negative_feedback | Requires matching open ticketId; creates/links its support task, flags the review task as paused/escalated and suppresses prompts while the issue remains unresolved. |

Negative-feedback example:

```json
{"requestId":"stable-feedback-id","channel":"phone","result":"reached","reviewOutcome":"negative_feedback","ticketId":"dc8024d4-410a-4f4b-bcb4-5f40b5d015ac","note":"Customer reports a service issue; linked existing ticket"}
```

Use the existing ticket creation workflow first if no matching open ticket exists. No duplicate ticket allocator was added. The response includes supportCaseId; the contact also retains the link in details for retry recovery. A duplicate request ID with identical payload returns the same event and support link. The original contact is never overwritten.

For current prompt eligibility, use the enriched task's top-level reviewPaused boolean. It rechecks the linked issue's current status; details.reviewPaused is the stored historical marker from the earlier event and may remain true until staff explicitly resume. Resolving the issue does not send another review prompt automatically.

Received/declined/negative_feedback require reached evidence; no_response cannot claim reached. Do not send a next review prompt date with a terminal or paused outcome. Requested/no_response may use the existing followUp/nextFollowUpAt fields, with a future timestamp required for Yes. Policy-driven automatic review generation, retry cadence and automatic sending are not enabled. An existing permanent decline remains suppressed even if a later complaint resolves. Negative-feedback suppression stops applying after its linked issue ticket is resolved, but resuming prompts remains an explicit staff/policy decision.

## D4 Performance response agreement

The proposed sections A–C and new follow-up availability objects are accepted as the future contract. They still return `{available:false,value:null,reason}` while the required policy/evidence is missing. No new financial number was guessed from legacy contact timing.

Clarifications before financial implementation:

- Direct Sales also needs uniqueCustomers to distinguish customers from orders. byOwner must support unconfirmed legacy ownership as well as sales/cs, unless management supplies an approved mapping. Do not drop unconfirmed creators to make two buckets add up.
- Future Chat/review/scope rate, coverage, conversion and percent fields in D4 are ratios from 0 to 1. Existing completionPercent remains 0–100, as explicitly required for D5 and by the original endpoint. Do not multiply completionPercent by 100 again.
- Revenue always separates orderValue and netRecognised and declares VAT treatment. A null netRecognised is unavailable, not zero. Refund/recognition policies must be approved before populating it.
- employeeId filters operational follow-up only; it will not filter the A–C business totals. The current A–C sections are unavailable regardless of employee selection. Performance still requires explicit timezone-qualified from/to bounds of at most 366 days; an unbounded all-time query is not implemented.

Each contactActivity row now includes:

```json
{
  "employeeId":7,
  "employeeName":"Example CS",
  "attempts":20,
  "reached":12,
  "uniqueCustomers":10,
  "unregisteredConversations":0,
  "dueTasks":10,
  "completedDueTasks":9,
  "completionPercent":90,
  "target":{"reached":120,"completionPercent":90}
}
```

Active CS users with zero activity are included, so a target can be displayed before their first attempt. Historical actors with measured activity also remain visible. Due-task staff attribution uses current owner and current deadline, exactly as the team figure; unassigned tasks can contribute to the team base without belonging to an employee row. Missing due base returns null completionPercent. No weighted score or ranking is calculated.

## D5 Monthly targets

GET `/targets?month=2026-11` returns, inside data:

```json
{"month":"2026-11","timeZone":"Asia/Phnom_Penh","items":[{"employeeId":7,"reached":120,"completionPercent":90,"updatedBy":3,"updatedAt":"2026-11-05T03:00:00.000Z"},{"employeeId":8,"reached":null,"completionPercent":null,"updatedBy":null,"updatedAt":null}]}
```

PUT `/targets/2026-11` accepts:

```json
{"items":[{"employeeId":7,"reached":120,"completionPercent":90},{"employeeId":8,"reached":null,"completionPercent":85}]}
```

Both target fields are required for each submitted employee; null clears a field. Unsubmitted employee rows are unchanged. Duplicate employee IDs, unknown fields, inactive/non-CS employees, negative/noninteger reached and percentages outside 0–100 return 400. Integer reached has the database upper bound 2147483647. Past-month changes return 422 using the server's Cambodia month; future months are allowed. GET of a past month remains allowed. Unauthorized writes return 403. The whole save is transactional and audited.

GET includes active CS users without saved targets, with null values. Report target is an object for a saved row, possibly containing cleared null fields; otherwise null. It is populated only for a complete Cambodia calendar month: from at local month start through exclusive next-month start. Custom/multiple-month ranges have target:null. A target of zero is valid and always met; never use a falsy fallback that turns zero into missing. Other null targets display No target. Null actual completion with a positive target is not a measured zero.

## B2 Scheduler setup

The server has `/api/scheduler/cs-actions`, protected by the existing cloudScheduler middleware and x-schedule-secret. Configuration is stored in the database, so no new CS environment variables are needed:

- cs_generation_config.name = scope
- enabled defaults to false
- actor_id must identify an active authorized operator account
- lookback_minutes must be an approved value from 1–43200
- updated_by records the manager/operator who configured it

Existing cloudScheduler authentication (x-schedule-secret / CLOUD_SCHEDULER_SECRET) is retained; this is the application's pre-existing scheduler security, not a new CS setting. Do not put a secret in SQL or the frontend. The manual schema file does not enable the scheduler or choose an operator/lookback on your behalf.

A database checkpoint retains the window/cursor between calls. Each invocation processes at most ten primary bookings. Duplicate events do not create extra tasks. An advisory lock prevents overlapping batches; overlapping windows cover late updates. A backlog beyond 31 days fails explicitly for operator repair. No SLA or outbound messaging default is introduced. Scope dueAt remains null until an explicit deadline/policy is supplied.

A cron job/service schedule still needs to be created/approved on the actual api-dev deployment. The manual Sync button is not intended to be the long-term sole generator.

## Database setup and rollout

Applied to staging `beasy-dev` on 5 November after a private backup:

1. `sql/cs-action-performance-v1.sql`
2. `sql/cs-action-frontend-contract.sql`
3. `sql/cs-action-performance-read-indexes.sql`, concurrent builds outside a transaction

The three read indexes were verified valid. Existing CS tables were empty. All nine staging segments were backed up before cleanup; seven unused definitions were then deleted and two coupon-linked definitions retained. No production database change, coupon/banner change, target insertion or runtime permission grant was performed. Re-running the additive scripts on legacy-shaped temporary fixtures was tested. New employee targets do not reuse the old team target table or endpoint. Historical target/author identifiers are retained without introducing user-deletion blockers; new target writes still require validated active staff.

The CS admin routes use existing authentication and permissions without an extra CS environment flag. Merge/deploy this code to the correct staging service, run the updated manual schema setup, verify database/runtime privileges and perform authenticated 200-response checks before disabling portal sample data. Do not run the production deployment script to accomplish a staging-only change.

## Verification

PostgreSQL temporary fixtures cover actual SQL joins, migration repeatability, names/service display, open filtering, contact/source totals, monthly target audits and report consistency, skipped/duplicate segment follow-ups, real review receipt checks, declines, negative-feedback pause/support links, combined history and durable scheduled checkpoints. Fixtures do not write application tables. Remaining live blockers are A source access/mapping, B1 deployment configuration and E1 business policy approval. This is not a claim of production load verification.

Verification result on 5 November: **32 CS checks passed**, including PostgreSQL temporary fixtures; **285 existing Direct Sale payment checks and 39 selected contact/Telegram/journey checks passed**. The separate legacy disposable Direct Sale DB harness was not configured, so 453 tests in that suite were skipped. Syntax checks, Markdown JSON example validation and git diff --check passed. The updated staging contract migration was refreshed after target/user-deletion compatibility checks; staging CS/contact/target rows remained empty and protected segment references were preserved.
