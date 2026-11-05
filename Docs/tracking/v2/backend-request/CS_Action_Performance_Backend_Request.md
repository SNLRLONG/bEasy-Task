# CS Action and CS Performance: what we need from backend

**To:** Backend · **From:** Portal · **Updated:** 5 Nov 2026
**Portal task:** T-0104, T-0105, T-0106 (branch `feat/cs-action`) · **Plan:** `Tasks/_plans/Portal-cs-action-performance.md`
**Read against:** [CS_Admin_Portal_Handoff.md](../backend-response/CS_Admin_Portal_Handoff.md) (your 4 Nov handoff) and the [CS brief](../requirements/bEasy_CS_Action_and_Performance_Developer_Brief_V1.md)

## Summary

The portal has two pages built to your handoff: **CS Action** (`/cs-action`, one working queue) and **CS Performance** (`/cs-performance`, one management page with four tabs). They run on sample data today because the CS routes are not on any environment we can reach.

Earlier portal notes use R-numbers: R2 = C1 and C2, R3 = B2, R4 = B1, R5 = E1, R6 and R6b = C3 and C4, R7 and R8 = C5, R9 = D2, R10 = D1, R11 = D4, R12 = D3.

Please reply per item number with **yes / no / later**, any change to the route or fields, and a date. Items are in the order they block us.

| # | Item | Blocks | Type |
|---|---|---|---|
| A | Clean the saved segments, then seed them from production | Testing the Saved segments tab | Data |
| B1 | Switch the CS API on (it answers 503 "not enabled" today) | All real testing | Config |
| B2 | Scheduled task generation | Tasks appearing without a manager | Feature |
| C1 to C5 | Fields and filters missing on existing endpoints | A usable queue | Contract |
| D1 to D3 | New create and outcome endpoints | Segment follow-ups, support tasks, reviews | Feature |
| D4 | Response shape for Performance sections A to C and extras | Management numbers | Contract |
| D5 | Monthly targets per CS, set by management | Comparing each CS with a target | Feature |
| E1 | Policy values | Due times, revenue, reviews | Decision |

---

## A. Saved segment data (urgent)

Dev and staging are the same environment (`api-dev.beasy.info`), so this is one job on that environment: **clean, then seed from production**.

A saved segment is a row behind `GET/POST/PUT/DELETE /api/admin/customer-segment` (`audience=customer` or `registered`): a name plus the filters the Customer pages send to `customer/existed` or `customer/register-customer`. The CS Action page shows them as cards with a live match count and lets CS open the people in each one. The data is currently a mess, so the page is hard to test.

**State today (read through the admin portal on 5 Nov 2026):** 9 segments, all test leftovers: "Give this filter set a name so you can reapply it…", "Hello", "Test", "segments consumer journey", "test save 2" (customer audience) and "test 4", "test reg 1", "test reg customer segments", "testing 5" (registered audience).

### A1. Clean: remove all saved segments

Delete every saved segment, both audiences, so the list starts empty.

**Care:** `DELETE /customer-segment/:id` is protected when a coupon or banner uses the segment. If a test coupon or banner points at one of these rows, either detach it first or leave that one row and list it in your reply. Do not remove coupons or banners we did not ask about.

### A2. Seed: copy production's saved segments

Copy the **production saved-segment definitions** into the same environment, so the segment cards show real names and sensible counts. Production is read-only for this job; nothing is written there.

**Rules for the copy**

1. **Segment definitions only:** `audience`, `name`, `filters`. No customer rows, no customer ids, no contact data, nothing from a segment's member list.
2. **Do not clean or edit the names.** If you want a note on which are pending review, say so in your reply.
3. **Fresh ids.** The environment gets new ids; do not try to keep production ids.
4. **Created-by:** production user ids do not exist here. Use one named admin user for every copied row and tell us which one.
5. **Filters can hold environment ids** (for example a service id, category id or preferred-service id). For each copied segment, check those ids exist here with the same meaning. If one does not, **skip that segment and list it** rather than copying a filter that silently matches nothing.
6. **Idempotent:** match on `(audience, name)`, so running it twice does not duplicate rows.
7. **Safe to run:** take a backup of the segment table first, run in a transaction, and run the production read with a read-only role.

**Report back:** how many segments were copied per audience, which were skipped and why, and which admin user owns them.

**Done when**
- `GET /customer-segment?audience=customer` and `?audience=registered` return production's segments, minus the listed skips, and none of the old test names.
- Each copied segment's count works: `GET customer/existed?<filters>&countOnly=true` (and the registered equivalent) returns a number, not an error, for every row.
- Nothing changed in production.

---

## B. Needed before we can test at all

### B1. Switch the CS API on

The portal pages and the admin permissions are ready, but the data is still sample data, because the CS API is switched off. We checked on 5 Nov with a logged-in admin session (read-only GET calls):

| Call | Answer |
|---|---|
| `GET /api/admin/cs/actions` | 503 "CS Action and Performance backend is not enabled" |
| `GET /api/admin/cs/performance` | 503, same message |
| `GET /api/admin/cs/performance/definitions` | 503, same message |
| `GET /api/admin/customer-segment` | 200 |

So the code is deployed and the feature flag is off. Please set `CS_ACTION_DASHBOARD_ENABLED=true` on `api-dev.beasy.info`, and confirm the migration `sql/cs-action-performance-v1.sql` and the read-index script are applied (we cannot see that from outside). Tell us when it is done.

**Done when:** `GET /api/admin/cs/actions?page=1&limit=1` returns 200 with `items` and `counters`.

### B2. Scheduled generation of scope tasks

Tasks exist only after a manager presses **Sync new orders** (`POST /actions/reconcile`). That is our interim button. Please tell us the date for an approved scheduled generation, or confirm the button is the long-term path. We do not want to call reconcile from the browser on every page load.

---

## C. Gaps in the endpoints you already built

The portal is working around each of these today.

**C1. Display fields on task rows.** Rows carry ids only. Please add `customerName`, `customerPhone`, `ownerName`, the bulk booking number (or a route-safe order reference), and for order tasks the **service name and scheduled date of each service line**. Today the portal calls `customer/{id}` once per visible row (up to 25 per page, cached 5 minutes) for name, phone and usual service, and cannot show the actual service of an order because the order detail route needs the bulk order id, not the service-line id.

**C2. Last contact on queue rows.** Add `lastContact: {result, attempts, at}` (or null) so the queue can show the Figma "Last contact" column without opening each task.

**C3. Totals and order.** Contact history and `/history` return no total or `hasMore`; `sources` is not paginated; ordering of `contacts` is not stated (the portal sorts newest first itself). Please add a total or `hasMore`, state the order, and make history cover contacts too if you can.

**C4. "Open" filter.** `status` takes one value, so **My cases** cannot exclude completed tasks. Please add `open=true` (every status except `completed`), or accept several statuses.

**C5. Identity and names.** Confirm `ownerId` and `csId` equal the admin user `id` from the login response (the portal uses that to show Log call, Schedule and so on only on my own tasks). And give us `{id, name}` for staff, or add `employeeName` to `contactActivity` and the task, so the employee filter on Performance shows names instead of `Staff #7`.

---

## D. New endpoints and fields

### D1. Start a follow-up from a saved segment

The brief (section 4) says Registered and Existing tasks come from saved CRM segments. `POST /actions` creates only scope and support tasks. Please add an idempotent create:

```json
POST /api/admin/cs/actions
{"category":"existing","purpose":"sales_follow_up","customerId":3526,"segmentId":12}
```

- `category` is `existing` (segment audience `customer`) or `registered` (audience `registered`).
- Returns `{task, duplicate}` like the other create.
- One open task per customer: a second segment or a Chat save adds a **source** to the same task (`sources` keeps every segment and conversation); it does not create a second task.
- Skip, with a reason in the response, a customer who opted out, already converted, or is not eligible.
- The portal's "Start follow-up" button is disabled on the real API until this exists. It works on sample data so the flow can be reviewed.

### D2. Support tasks: who creates them?

Only `POST /actions` with an existing ticket creates a support task. Tell us the intended path: CS creates one from a ticket, or the system creates one on a complaint, low rating or negative review. The portal has no create screen until this is decided.

### D3. Review request tasks and outcomes

The After-Service review flow is built in the portal on sample data. Please add:

- Task purpose `review_request`: one per eligible completed order; none if a review already exists, or the customer declined.
- `reviewOutcome` on `POST /actions/:id/contacts`: `requested | received | declined | no_response | negative_feedback`.
- Rules: **received** (verified against a real review) and **declined** close the task; declined also stops further requests; **negative_feedback** creates or links a support case and pauses review prompts for that customer; **requested** and **no_response** keep the task open.

### D4. Performance: sections A to C and extra follow-up measures

The Performance page has a tab per section: A Direct Sales, B App non-assisted, C App assisted, D Follow-up. A to C and three follow-up measures are built to the shape below. **Please confirm or change it before you build.** Each stays `{available:false, value:null, reason}` until its policies (E1) are approved. Rules: orders and customers are separate counts; every revenue field carries both `orderValue` and `netRecognised` and a VAT label; an order is in exactly one of assisted or non-assisted; one accountable owner per Direct Sale order; staff filter applies to follow-up measures only; rates are 0 to 1 and `null` means no eligible base (we show n/a, never 0%).

```json
"sections": {
  "directSales": {
    "available": true,
    "orders": 42,
    "revenue": { "orderValue": 5240.5, "netRecognised": 4980 },
    "vat": "inclusive",
    "byOwner": [
      { "owner": "sales", "orders": 30, "revenue": { "orderValue": 3800, "netRecognised": 3610 } },
      { "owner": "cs",    "orders": 12, "revenue": { "orderValue": 1440.5, "netRecognised": 1370 } }
    ]
  },
  "appNonAssisted": {
    "available": true,
    "uniqueCustomers": 118,
    "orders": { "first": 64, "repeat": 97 },
    "revenue": { "orderValue": 9120, "netRecognised": 8760 },
    "vat": "inclusive",
    "postPurchaseContact": { "yes": 71, "no": 90 }
  },
  "appAssisted": {
    "available": true,
    "firstTimeCustomers": 22,
    "returningCustomers": 31,
    "orders": 53,
    "revenue": { "orderValue": 3410, "netRecognised": null },
    "vat": "inclusive"
  },
  "followUp": {
    "chatClosingRate": {
      "available": true, "rate": 0.31, "won": 24, "eligible": 77, "pending": 18,
      "byChannel": [ { "channel": "Telegram", "eligible": 40, "won": 14 } ],
      "lostReasons": [ { "reason": "Price", "count": 21 } ]
    },
    "reviewCoverage": {
      "available": true, "eligible": 88, "requested": 70, "received": 29,
      "coverage": 0.795, "conversion": 0.414
    },
    "scopeCoverage": {
      "available": true, "required": 61, "confirmed": 52,
      "unanswered": 6, "overdue": 3, "percent": 0.852
    }
  }
}
```

Each `followUp.contactActivity` row also needs the employee's own due-task numbers, so a target can be compared without one request per person: add `dueTasks`, `completedDueTasks` and `completionPercent` (null when no task was due), using the same rules as the team figure.

`pending` chats are never counted as lost. `netRecognised: null` is shown as "Unavailable", not $0.

### D5. Monthly targets for each CS, set by management

Management wants to set a target for each CS employee and see it next to their actual numbers. The 3 Nov brief said to avoid employee targets in this phase; management has now asked for them. Keep it plain: numbers beside the actuals, **no weighted score and no ranking**.

The old CS Overview has team-level monthly targets (`/cs-overview/targets/:month`: first-sale %, repeat %, completed on time %). That is a different measure set and not per employee, so please do not reuse it. Per employee and per month, we propose two measures the backend can already count per employee:

| Target | Compared with |
|---|---|
| `reached` (whole number) | `contactActivity.reached` for that employee and month |
| `completionPercent` (0 to 100) | `contactActivity.completionPercent` for that employee and month |

Sales targets (first sales, repeat sales) can be added to the same record later, when sales attribution is approved.

```http
GET /api/admin/cs/targets?month=2026-11
-> { "month": "2026-11", "timeZone": "Asia/Phnom_Penh",
     "items": [ { "employeeId": 7, "reached": 120, "completionPercent": 90,
                  "updatedBy": 3, "updatedAt": "2026-11-01T02:10:00.000Z" } ] }

PUT /api/admin/cs/targets/2026-11
{ "items": [ { "employeeId": 7, "reached": 120, "completionPercent": 90 },
             { "employeeId": 9, "reached": null, "completionPercent": 85 } ] }
-> the saved items, same shape as GET
```

- **Who:** `GET` needs `CS_PERFORMANCE` `VIEW`; `PUT` needs `CS_PERFORMANCE` `UPDATE` (management). A CS employee does not see other people's targets.
- **Missing target:** an employee with no target for the month is returned with `null` values. The portal shows "No target", never 0.
- **Validation:** `reached` a whole number from 0; `completionPercent` 0 to 100; a field may be `null` to clear it; `employeeId` must be an active CS employee; an unknown field is rejected. Reject a month before the current month with 422 so past results stay comparable (a future month is allowed).
- **Meets target:** actual at or above target meets it (equal counts), and a target of `0` is valid and always met.
- **Audit:** record who changed which target, with the before and after values, in the same audit history as task changes.
- **Also return the target in the report:** in `contactActivity` rows add `target: {reached, completionPercent} | null` for the selected month, so the Performance page needs no second call. For a custom range or an overall period, return `target: null` (targets are monthly).

**Done when:** a manager can save targets for two employees, `GET` returns them, a non-manager gets 403 on `PUT`, and the Performance report shows the same values in `contactActivity[].target`.

---

## E. Decisions we need (no values are hardcoded in the portal)

- **E1. Policies:** scope-call SLA, retry timing and escalation threshold (until then due time is null and shows "No deadline set"); service-specific repeat-booking intervals; sales-assistance window and what counts as qualifying human contact; Chat opportunity deduplication and conversion window; review eligibility, channel, timing, retry and opt-out rules; revenue recognition basis, VAT treatment and the Direct Sale owner rule.

---

## What the portal does meanwhile

- Runs on sample data (`VITE_CS_MOCK=true`, off by default) and shows a "Sample data" banner.
- Shows `Unavailable` and the backend's reason, never 0 or an estimate, for anything marked unavailable.
- Shows Start follow-up and review tasks only on sample data; they are hidden or disabled on the real API.

## Not asked for in this release

Linking an unregistered Chat lead to a customer record, automatic review sending, per-service and per-channel filters on Performance, and a policy-editing screen.
