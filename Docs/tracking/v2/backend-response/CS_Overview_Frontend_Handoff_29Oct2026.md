# CS Overview — Admin Portal Integration Handoff

| Document control     | Detail                                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------------- |
| Audience             | Admin portal developers, QA, technical lead and product reviewer                                  |
| Feature / repository | CS Overview / `bEasy-api`                                                                         |
| Branch               | `feature/cs-kpi`                                                                                  |
| Source review        | 29 October 2026; base commit `9e0e9a55` plus current working-tree changes                         |
| Requirements         | 1 October CS Overview brief, 26–27 October UX decisions and subsequent screenshots/clarifications |
| Delivery status      | Backend contract handoff; deployment and frontend integration still require verification          |

## 1. Purpose and delivery boundaries

CS Overview reports first sales, repeat sales and retained CS contact activity. It is related to Consumer Journey, but it is a separate reporting page. Journey and customer screens provide customer context and, eventually, sources for follow-up cases.

The frontend can integrate the reporting, number-to-list navigation, monthly Excel export and existing saved CRM segments. Monthly target read/write APIs are available in this branch after the target SQL is installed. The task queue and immutable contact workflow shown in the design are **not implemented yet**.

This document describes checked-out source, not confirmation of staging deployment or reconciled production totals. Examples use illustrative data, not customer records. No frontend source changes, database migration execution or deployment are included in this document delivery.

### Readiness matrix

| Area                                                               | Current backend support                                              | Frontend treatment                                                    |
| ------------------------------------------------------------------ | -------------------------------------------------------------------- | --------------------------------------------------------------------- |
| First-sale cards and employee table                                | Available, with legacy attribution limitations                       | Wire to summary and people APIs; show report definitions              |
| Repeat-sales employee table                                        | Available                                                            | Use deduplicated server totals; open qualifying order/customer lists  |
| Recorded contact activity                                          | Partial: retained legacy records only                                | Label as recorded contacts; preserve unknown outcomes                 |
| Edit monthly targets                                               | New GET/PUT APIs; SQL and permissions required                       | Enable after deployment checks; fetch separately from summary         |
| Saved segments from Customers and Registered Customers             | Existing CRUD and audience-list counts; registered `countOnly` added | Reuse saved definitions and matching audience endpoints               |
| Assigned / not contacted / due today / overdue / on-time task work | Not implemented                                                      | Show `—` and “Not implemented yet”; no fabricated totals              |
| Send to CS / pickup / reassignment / complete or reschedule        | Placeholder or absent                                                | Disable actions with a clear explanation                              |
| Segment follow-up progress                                         | Not implemented                                                      | Do not equate audience membership with sent/contacted/completed cases |
| Problems found by CS / weighted issue score                        | Out of scope                                                         | Keep marked out; do not include in KPI calculations                   |

Related technical inventories: [Endpoint and action inventory](cs-overview-endpoints.md) and [Backend UX review](cs-backend-ux-review.md).

## 2. Transport, authentication and response handling

### Base path and authentication

All endpoint paths in this document are relative to **`/api/admin`**. Resolve the host from the portal's existing environment configuration; do not hardcode a staging host in components.

```http
Authorization: Bearer <existing-admin-access-token>
Accept: application/json
```

Use `Content-Type: application/json` for JSON writes. Reuse the existing admin API client and token refresh handling.

### Standard JSON envelope

Reporting, targets, segments and legacy customer endpoints use the existing response wrapper:

```json
{
  "code": "SUCCESS",
  "status": true,
  "message": "Success",
  "data": {
    "month": "2026-10",
    "configured": false,
    "firstSaleConversionTarget": 30,
    "repeatConversionTarget": 30,
    "completedOnTimeTarget": 90,
    "updatedBy": null,
    "updatedAt": null
  }
}
```

The existing portal `app/api/api.ts` interceptor returns the unwrapped payload for these non-null responses. A caller using that client receives the target object above, not an Axios response with another `.data.data`. Do not add a second unwrapping interceptor. Native `fetch` or another raw client must read the wire envelope's `data` once.

Validation example: HTTP **422**:

```json
{
  "code": "ERROR",
  "status": false,
  "message": "month must be YYYY-MM",
  "data": { "field": "month" }
}
```

Use `error.response.status`, `error.response.data.message` and optional `error.response.data.data.field` with the existing Axios error flow. Some older customer/segment errors have `data: null`. Journey placeholder errors and Excel errors use different formats, documented below.

### Access boundaries

| Resource                               | Current server enforcement                                                          |
| -------------------------------------- | ----------------------------------------------------------------------------------- |
| Summary, people lists, CS Excel export | Existing `authAdmin`; these handlers do not add a CS module/action permission check |
| Target GET                             | `authAdmin` plus database role permission `CS-OVERVIEW / VIEW`                      |
| Target PUT                             | `authAdmin` plus database role permission `CS-OVERVIEW / UPDATE`                    |
| Saved-segment CRUD                     | Existing `authAdmin`; no new CS-specific permission check                           |

Do not describe a hidden navigation item as server-side authorization. The portal's existing Customer Overview module mapping and the target API's `CS-OVERVIEW` mapping need alignment by the project owner before release. The policies for viewing other employees, All open and lead-only reassignment are still unresolved. Do not invent role grants, pass an actor ID from the UI, or assume that `roleName` is populated by admin authentication.

## 3. Endpoint directory

### Working reporting and configuration APIs

| Method | Endpoint                      | Request                             | Successful result        |
| ------ | ----------------------------- | ----------------------------------- | ------------------------ |
| GET    | `/cs-overview`                | Shared report query                 | `CsOverviewSummary`      |
| GET    | `/cs-overview/people`         | Shared query + metric/pagination    | `CsPeoplePage`           |
| POST   | `/fileExport/cs-overview`     | Shared query; no JSON body required | XLSX binary, not JSON    |
| GET    | `/cs-overview/targets/:month` | Month in path                       | `CsMonthlyTargets`       |
| PUT    | `/cs-overview/targets/:month` | Month in path + three percentages   | Saved `CsMonthlyTargets` |

### Saved CRM segments

| Method | Endpoint                                | Purpose                                                |
| ------ | --------------------------------------- | ------------------------------------------------------ |
| GET    | `/customer-segment?audience=customer`   | Saved Customer-page segments                           |
| GET    | `/customer-segment?audience=registered` | Saved Registered Customer-page segments                |
| GET    | `/customer-segment/:id`                 | One definition                                         |
| POST   | `/customer-segment`                     | Create a definition                                    |
| PUT    | `/customer-segment/:id`                 | Update name and/or filters                             |
| DELETE | `/customer-segment/:id`                 | Delete, subject to coupon/banner usage protection      |
| GET    | `/customer/existed`                     | Apply Customer audience filters; optional count-only   |
| GET    | `/customer/register-customer`           | Apply Registered audience filters; optional count-only |

### Follow-up endpoints that must remain disabled

These routes exist, but return HTTP **501**, with no task/contact write:

| Method | Endpoint                | Intended action                                        |
| ------ | ----------------------- | ------------------------------------------------------ |
| POST   | `/cs/cases`             | Send customers to CS                                   |
| GET    | `/cs/cases`             | Follow-up list, including Mine / All open / Unassigned |
| POST   | `/cs/cases/:id/pick-up` | Pick up a case                                         |
| POST   | `/cs/contacts`          | Immutable contact attempt and case outcome             |
| GET    | `/cs/overview`          | Prepared Journey CS overview contract                  |

**`/cs-overview` works; `/cs/overview` is a placeholder. They are different paths.**

Example placeholder response for `/cs/cases`:

```json
{
  "error": {
    "code": "NOT_IMPLEMENTED",
    "message": "CS cases is not available yet. It needs data the backend does not collect yet.",
    "field": null
  }
}
```

There is no implemented reassignment endpoint, complete/reschedule endpoint, no-more-calls endpoint, CS metric-list export endpoint, or segment follow-up-progress endpoint. A future contract must be agreed before wiring these actions; do not guess request fields from a screenshot.

## 4. Report query contract

The summary, people lists and Excel export accept the same report filters. Keep one shared filter object in the page and pass it to all three.

| Field           | Type / example                                       | Default and behavior                                                                     |
| --------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `period`        | `month`, `range`, `all`                              | Default `month`                                                                          |
| `month`         | `2026-10`                                            | Current Phnom Penh month if omitted; used only for `period=month`; future month rejected |
| `startDate`     | `2026-08-01`                                         | Required for `period=range`                                                              |
| `endDate`       | `2026-10-31`                                         | Required for range; inclusive selected day; must be on/after start                       |
| `windowDays`    | Integer `1`–`90`                                     | Default `30`; approved monthly design uses a fixed 30-day window                         |
| `csId`          | Positive user ID, `unknown`, `all`, empty or omitted | Omitted/empty/`all` means all employees; `unknown` opens “Employee not recorded”         |
| `serviceIds`    | Comma-separated IDs, e.g. `2,7`                      | Omitted/empty means all services; duplicate IDs are removed                              |
| `followUpGroup` | Omitted, empty or `all`                              | Other values return 422; group filtering is not implemented                              |

Ranges use Phnom Penh calendar dates. Start cannot be in the future; an end in the future is allowed for existing date-picker compatibility. The response's `from` is inclusive and `to` is exclusive. `startDate`/`endDate` are the user-facing date labels. Overall uses all available history through calculation time and returns `from: null`, `startDate: null`, `month: null`.

Month and conversion window have different purposes:

- **Month/period** selects registrations for first sales, follow-up episode starts for repeat sales, and effective contact dates for contact activity.
- **30 days** is the observation time after each registration or episode start. It is not “the last 30 days” from today. A September registration can convert in October and remain in September's registration cohort.
- Current design: present month selection and explain the fixed 30-day observation rule. The range/overall modes and alternate window values remain supported for existing clients; they are not an instruction to add a second competing date selector.
- To offer multiple consecutive months, use `period=range`. There is no `months[]` parameter or per-month comparison-series response. Do not sum separate monthly ratios or repeat-customer counts to manufacture an overall result.

Examples:

```http
GET /api/admin/cs-overview?month=2026-10&windowDays=30
GET /api/admin/cs-overview?month=2026-10&windowDays=30&csId=7&serviceIds=2,7
GET /api/admin/cs-overview?period=range&startDate=2026-08-01&endDate=2026-10-31&windowDays=30
GET /api/admin/cs-overview?period=all&windowDays=30
```

Do not send range dates and assume they filter a request whose `period` is still `month`. Remove irrelevant query parameters when switching modes. Target editing remains month-specific: there is no aggregate target for range/overall.

## 5. Summary response: fields and units

`GET /cs-overview` returns the following payload inside the standard envelope. These TypeScript declarations describe the API contract; integrate them with the portal's existing type organization rather than duplicating domain types across components.

```ts
type IsoDateTime = string;
type Money = number | null; // USD; null means unavailable, not $0.
type Rate = number | null; // Ratio 0–1, rounded to 4 decimals; not percentage units.
type CsFilterId = number | 'unknown' | null;

interface CsEmployee {
  csId: number | null;
  name: string;
  ratingStatus?: 'not_rated';
}

interface FirstSaleEmployee extends CsEmployee {
  customers: number;
  converted: number;
  rate: Rate;
  revenue: Money;
}

interface RepeatTotals {
  followedUp: number;
  orderedAgain: number;
  rate: Rate;
  repeatBookings: number;
  completedBookings: number;
  revenue: Money;
}

interface RepeatEmployee extends CsEmployee, RepeatTotals {}

interface ActivityTotals {
  attempts: number;
  reached: number | null;
  knownOutcomes: number;
  unknownOutcomes: number;
  reachedRate: Rate;
  customersContacted: number;
}

interface ActivityEmployee extends CsEmployee, ActivityTotals {}

interface FirstSaleGroup {
  customers: number;
  converted: number;
  rate: Rate;
  revenue: Money;
  completedOrders: number;
}

type CsAvailabilityKey =
  | 'followUpCases'
  | 'dailyWork'
  | 'contactHistory'
  | 'salesPurposeAttribution'
  | 'chatContactCapture'
  | 'followUpGroup'
  | 'monthlyTargets'
  | 'savedSegments'
  | 'partialRefundRevenue'
  | 'repeatOrderStatusTiles'
  | 'problemsFound';

interface CsAvailabilityEntry {
  status: 'partial' | 'not_implemented' | 'out_of_scope';
  label: string;
  reason: string;
  value?: null;
}

interface CsOverviewSummary {
  period: {
    mode: 'month' | 'range' | 'all';
    month: string | null;
    from: IsoDateTime | null;
    to: IsoDateTime;
    startDate: string | null; // YYYY-MM-DD
    endDate: string; // YYYY-MM-DD, inclusive display date
    windowDays: number;
    provisional: boolean;
    finalAt: IsoDateTime | null;
  };
  filters: { csId: CsFilterId; serviceIds: number[] };
  firstSales: {
    total: number;
    provisional: number;
    contacted: FirstSaleGroup & {
      legacyContactCustomers: number;
      byEmployee: FirstSaleEmployee[];
    };
    noContact: FirstSaleGroup;
    otherChannels: {
      converted: number;
      rate: Rate;
      revenue: Money;
      completedOrders: number;
    };
  };
  repeatSales: {
    provisional: number;
    byEmployee: RepeatEmployee[];
    total: RepeatTotals;
  };
  activity: { byEmployee: ActivityEmployee[]; total: ActivityTotals };
  employees: { csId: number; name: string }[];
  dataNotes: { excludedTestOrBlacklistedCustomers: number };
  availability: Record<CsAvailabilityKey, CsAvailabilityEntry>;
  generatedAt: IsoDateTime;
}
```

### Display mapping

| UX element                       | Payload field / behavior                                                                                   |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Registration cohort size         | `firstSales.total`; remains the full selected registration cohort, not the selected employee's denominator |
| Contacted by CS card             | `firstSales.contacted.customers`, `.converted`, `.rate`, `.completedOrders`, `.revenue`                    |
| No CS contact card               | `firstSales.noContact`; `.converted` counts app first sales only                                           |
| Other sales without CS contact   | `firstSales.otherChannels`; includes unassisted direct sales; separate from app self-conversion            |
| First-sale employee rows         | `firstSales.contacted.byEmployee`                                                                          |
| First-sale team footer           | `firstSales.contacted` totals; do not sum overlapping employee denominators                                |
| Repeat rows and footer           | `repeatSales.byEmployee` / `.total`                                                                        |
| Recorded contact rows and footer | `activity.byEmployee` / `.total`                                                                           |
| Employee filter choices          | `employees`; this is report-derived, not a complete staff/workload directory                               |
| Unknown employee option          | Add `unknown` when needed; displayed as “Employee not recorded”; not a real user ID                        |
| Provisional banner               | `period.provisional`; use `period.finalAt`, not a client-estimated month-end-plus-30 date                  |
| First-sale provisional count     | `firstSales.provisional`: customers whose observation window is still open                                 |
| Repeat provisional count         | `repeatSales.provisional`: open observation episodes, not necessarily distinct customers                   |
| Calculated timestamp             | `generatedAt`, displayed in `Asia/Phnom_Penh`                                                              |
| Missing feature explanation      | Appropriate `availability` entry's `label` and `reason`                                                    |

`finalAt` reflects the latest actual observation-window end in the computed first/repeat cohorts. An empty month can return `provisional: false, finalAt: null`. Overall remains provisional with `finalAt: null`. Observation closure does not freeze later payment, completion or order-status changes.

### Formatting rules

| Value                                               | Correct display                                                      |
| --------------------------------------------------- | -------------------------------------------------------------------- |
| `rate: 0.3333`                                      | `33.33%` or the existing product percentage precision; never `0.33%` |
| `rate: 0`                                           | `0%`, a measured zero                                                |
| Rate null with no eligible denominator              | `N/A`                                                                |
| Employee `ratingStatus: not_rated`                  | `Not rated`, including target comparisons                            |
| `reached: null` with attempts and no known outcomes | `Unavailable`; do not substitute `0`                                 |
| Known outcomes plus missing outcomes                | Show known reached count/rate and separate unavailable count         |
| `revenue: null`                                     | `Unavailable`, not `$0.00`                                           |
| `revenue: 0`                                        | `$0.00`                                                              |
| Not implemented feature                             | `—` plus “Not implemented yet”; not `N/A` or `0`                     |

Reached rate is `reached / knownOutcomes`, not `reached / attempts`. For example, 1 reached, 0 known not-reached and 6 unavailable produces 100% **of 1 known outcome**, with 6 unavailable shown separately. It does not mean all 7 contacts reached the customer.

## 6. Number-to-list navigation

### Query and response

```http
GET /api/admin/cs-overview/people?month=2026-10&windowDays=30&csId=7&metric=first_sale_cs_converted&page=1&limit=50
```

Additional query fields:

| Field    | Contract                                                                  |
| -------- | ------------------------------------------------------------------------- |
| `metric` | Required; one key from the table below                                    |
| `page`   | One-based; default 1; values below 1 clamp to 1                           |
| `limit`  | Default 50; maximum 200; use existing page-size choices within this bound |
| `sortBy` | Omit for default order; `revenue` for first-sale/repeat lists only        |

Unsupported metrics/sort values return 422. Free-text `q`/search, extra date/status/purpose filters inside the sheet, and server-side employee-table pagination are not implemented. Do not display these controls as working. An unsupported unknown query key is not evidence that a filter was applied.

```ts
type CsMetric =
  | 'first_sale_contacted'
  | 'first_sale_cs_converted'
  | 'first_sale_cs_completed'
  | 'first_sale_no_contact'
  | 'first_sale_app_converted'
  | 'first_sale_app_completed'
  | 'first_sale_other_converted'
  | 'repeat_followed_up'
  | 'repeat_ordered_again'
  | 'repeat_orders'
  | 'contact_attempts'
  | 'contact_reached'
  | 'contact_not_reached'
  | 'contact_unknown'
  | 'contact_customers';

interface CsPeoplePage {
  metric: CsMetric;
  count: number; // Matching rows before pagination, not items.length.
  page: number;
  limit: number;
  hasMore: boolean;
  tiles?: { reached: number; notReached: number; unavailable: number };
  items: (FirstSaleItem | RepeatItem | ContactItem)[];
}
```

### Click mapping: use the same report filters as the displayed number

| Clickable number                            | Metric                       | Row unit                                               |
| ------------------------------------------- | ---------------------------- | ------------------------------------------------------ |
| Registered customers contacted              | `first_sale_contacted`       | Customer                                               |
| CS first-sale customers                     | `first_sale_cs_converted`    | Customer and first order                               |
| CS completed paid first orders              | `first_sale_cs_completed`    | Customer and first order                               |
| Registered customers without CS contact     | `first_sale_no_contact`      | Customer                                               |
| First app-sale customers without CS contact | `first_sale_app_converted`   | Customer and first order                               |
| Completed paid app first orders             | `first_sale_app_completed`   | Customer and first order                               |
| Other unassisted first sales                | `first_sale_other_converted` | Customer and first order                               |
| Existing customers followed up              | `repeat_followed_up`         | Unique customer                                        |
| Existing customers who ordered again        | `repeat_ordered_again`       | Unique customer                                        |
| Repeat bookings                             | `repeat_orders`              | Order; a customer can have multiple rows               |
| Retained contact records / attempts         | `contact_attempts`           | Retained contact record                                |
| Reached                                     | `contact_reached`            | Contact record with known reached result               |
| Not reached                                 | `contact_not_reached`        | Contact record with known not-reached result           |
| Outcome unavailable                         | `contact_unknown`            | Contact record without a known result                  |
| Unique customers contacted                  | `contact_customers`          | Unique customer, represented by latest matching record |

Revenue links use the applicable converted metric with `sortBy=revenue`; repeat revenue uses `repeat_orders&sortBy=revenue`. The list's **count is a count of rows, not the monetary value**. A converted list can include a deposit/scheduled order with zero completed revenue. For a completed-paid-only list, use the corresponding `first_sale_*_completed` metric.

For an employee-row click, override `csId` with that row's numeric ID. Use `csId=unknown` for “Employee not recorded”. For a team-footer click, omit `csId`. No-contact and other-unassisted groups remain global when an employee is selected; do not relabel them as that employee's own customers.

### Item fields

```ts
interface CsCustomer {
  id: number;
  name: string | null;
  phone: string | null; // Full recorded phone, not masked.
}

type ContactOutcome =
  | 'REACHED_INTERESTED'
  | 'REACHED_NOT_INTERESTED'
  | 'REACHED_TOO_EXPENSIVE'
  | 'REACHED_NO_CURRENT_NEED'
  | 'REACHED_SERVICE_ISSUE'
  | 'NO_ANSWER'
  | 'WRONG_NUMBER'
  | 'FOLLOW_UP_LATER';

interface CsContactFields {
  contactedAt: IsoDateTime | null; // Effective time used by the report.
  enteredContactedAt: IsoDateTime | null; // Only populated if different from effective time.
  outcome: ContactOutcome | null;
  outcomeLabel: string | null;
  source: 'log' | 'legacy' | null;
}

interface CsOrder {
  id: number;
  bulkOrderId: string | null;
  type: 'ORDER' | 'DIRECT_SALE';
  placedAt: IsoDateTime;
  status: string; // Existing order status, not a new CS status enum.
  paymentStatus: string;
  completed: boolean;
  completedPaid: boolean;
  revenue: Money;
}

interface FirstSaleItem extends CsContactFields {
  key: string;
  customer: CsCustomer;
  employee: CsEmployee | null;
  creditedEmployee: CsEmployee | null;
  registeredAt: IsoDateTime;
  windowEndsAt: IsoDateTime;
  provisional: boolean;
  group: 'cs' | 'no_contact';
  convertedVia: 'cs' | 'app' | 'other' | null;
  order: CsOrder | null;
  revenue: Money;
}

interface RepeatItem extends CsContactFields {
  key: string;
  customer: CsCustomer;
  employee: CsEmployee;
  followUpStartedAt: IsoDateTime;
  windowEndsAt: IsoDateTime;
  provisional: boolean;
  ordersInWindow: number;
  order: CsOrder | null;
  revenue: Money;
}

interface ContactItem extends CsContactFields {
  key: string;
  customer: CsCustomer;
  employee: CsEmployee;
  reached: boolean | null;
}
```

The API does not add a `kind` discriminator to items. The requested `metric` determines the item family. Do not expect fields from one family on another. An aggregated repeat-customer row can have `employee: { csId: null, name: "Multiple employees" }`; that is not the unknown-employee group and must not link to `csId=unknown`.

`source=legacy` represents inferred Registered Customer status data with no recorded employee; `source=log` represents a retained contact row. Neither guarantees a complete immutable attempt history. Display the supplied `outcomeLabel` rather than maintaining a competing label map. Current `FOLLOW_UP_LATER` is a known **not-reached** result in the legacy model; it is not the newer design's separate “busy” result plus response intent.

Illustrative people response, inside the standard envelope:

```json
{
  "metric": "first_sale_cs_converted",
  "count": 1,
  "page": 1,
  "limit": 50,
  "hasMore": false,
  "items": [
    {
      "key": "first-101",
      "customer": { "id": 101, "name": "Example Customer", "phone": "+85500000000" },
      "employee": { "csId": 7, "name": "Example Agent" },
      "creditedEmployee": { "csId": 7, "name": "Example Agent" },
      "registeredAt": "2026-10-01T09:00:00+07:00",
      "windowEndsAt": "2026-10-31T09:00:00+07:00",
      "provisional": true,
      "group": "cs",
      "convertedVia": "cs",
      "contactedAt": "2026-10-02T10:00:00+07:00",
      "enteredContactedAt": null,
      "outcome": "REACHED_INTERESTED",
      "outcomeLabel": "Reached — Interested",
      "source": "log",
      "order": {
        "id": 9001,
        "bulkOrderId": "EXAMPLE-9001",
        "type": "DIRECT_SALE",
        "placedAt": "2026-10-03T11:00:00+07:00",
        "status": "COMPLETED",
        "paymentStatus": "PAID",
        "completed": true,
        "completedPaid": true,
        "revenue": 18
      },
      "revenue": 18
    }
  ]
}
```

### Paging, sorting and tiles

- Use `key` for stable React row keys. Do not key order/contact rows only by customer ID.
- `count` is the server's pre-pagination total. Derive page count as `Math.ceil(count / limit)`; use `hasMore` for Next. An empty page returns `items: []`.
- Default first/repeat lists order by customer ID and stable item key. Contact lists order by effective contact time descending, then contact ID descending. Revenue sort is descending with unavailable values last.
- Contact-attempt-family metrics return `tiles` before pagination, except `contact_customers`. The tiles describe the **current metric's result set**. On `contact_reached`, the tiles describe the reached subset, not all original attempts. Fetch/use the `contact_attempts` result for all-outcome tiles; switch metric to open each subset.
- Repeat Completed/Scheduled/Refunded status tiles are not implemented. Do not derive a refunded total from a list that excludes fully refunded orders.
- There is no cross-request snapshot token. Records can change between summary, page and export calls; deterministic ordering does not freeze data.

## 7. Monthly targets: GET and PUT

**Deployment dependency:** apply and verify `sql/cs-overview-targets.sql` before enabling the editor. The script adds `cs_overview_targets`; it does not repurpose dashboard `AppConfig.saleTarget`, alter customer history, or grant permissions. The script was prepared but not executed during this work.

Read:

```http
GET /api/admin/cs-overview/targets/2026-10
```

Write:

```http
PUT /api/admin/cs-overview/targets/2026-10
Content-Type: application/json
```

```json
{
  "firstSaleConversionTarget": 30,
  "repeatConversionTarget": 30,
  "completedOnTimeTarget": 90
}
```

```ts
interface CsMonthlyTargets {
  month: string; // YYYY-MM; path is authoritative.
  configured: boolean;
  firstSaleConversionTarget: number; // Percent 0–100, up to 2 decimals.
  repeatConversionTarget: number;
  completedOnTimeTarget: number;
  updatedBy: number | null;
  updatedAt: IsoDateTime | null;
}
```

All three body fields are required on PUT. Send numbers; numeric strings are also accepted by the current implementation. Zero is valid. Null, empty, negative, over-100 and more-than-two-decimal values are rejected with 422 and the field name.

If storage exists but no row has been saved, GET returns defaults **30 / 30 / 90**, `configured: false`, `updatedBy: null`, `updatedAt: null`. Reading does not persist defaults. After saving, `configured: true`, actor and update time come from the server. Never send or trust a client-supplied `updatedBy`, `actorId` or employee ID for authorization.

Valid future target months are allowed for planning, although future reporting months are rejected. Saving one month does not change another month. GET requires VIEW; PUT requires UPDATE; permissions are checked against current database roles on each request.

| Target error                  | Frontend behavior                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------------- |
| 403 / `data.field=permission` | Explain access restriction; do not retry as an anonymous/default actor                |
| 422 / field name              | Show inline validation; preserve entered values                                       |
| 503 / `data.field=targets`    | Show target configuration unavailable; request storage deployment; keep report usable |

Missing storage message: `CS target storage is not installed yet. Apply sql/cs-overview-targets.sql.` Do not replace that failure with `configured: false`; a missing table is different from a legitimate unsaved month.

Targets are **not embedded** in the summary and no Above/Below badge fields are returned. Fetch targets for the selected month separately. Compare `rate * 100 >= target`; equality meets target. Keep target `0` using null checks, not a truthy fallback. Employee-not-recorded rows are Not rated; a null denominator is N/A. On-time target can be edited, but its actual task rate is unavailable until task history exists. Do not compare contact reached rate with on-time target.

## 8. Saved CRM segments on CS Overview

### Reuse the existing saved definitions

Request the two audiences separately, then display them with a source badge:

```http
GET /api/admin/customer-segment?audience=customer
GET /api/admin/customer-segment?audience=registered
```

Each returns an array of these objects inside the standard envelope, sorted by name then ID:

```ts
interface SavedCrmSegment {
  id: number;
  name: string;
  audience: 'customer' | 'registered';
  filters: Record<string, unknown>;
  createdBy: number;
  createdByName: string | null;
  createdByUser: {
    id: number;
    username: string;
    firstName: string | null;
    lastName: string | null;
  } | null;
  createdAt: IsoDateTime;
  updatedAt: IsoDateTime;
}
```

Omitted or unrecognized audience falls back to `customer`. **`audience=all` does not combine them.** A saved segment is a reusable query definition, not a static customer-ID snapshot. A segment's audience cannot be changed by PUT.

Create example:

```json
{
  "name": "Never contacted customers",
  "audience": "customer",
  "filters": { "lastContact": "never" }
}
```

POST and PUT return the saved object above. PUT accepts `name` and/or `filters`. Name is trimmed, must be nonempty and is case-insensitively unique within an audience. At least one recognized nonempty filter is required when creating/replacing filters. Unknown filters are removed server-side. Duplicate name is 409; invalid input is 400; missing ID is 404.

DELETE returns `{ "id": 12 }` inside the standard envelope. A segment used by coupons/banners is protected: show the server's explanation and leave it in the list. Existing dependency errors use the standard error envelope, normally HTTP 400. Do not unlink dependencies automatically.

### Matching counts and customer lists

For each saved segment, use its **stored filters**, not just its ID:

```http
GET /api/admin/customer/existed?lastContact=never&countOnly=true&page=1&limit=1
GET /api/admin/customer/register-customer?<stored-registered-filters>&countOnly=true&page=1&limit=1
```

`<stored-registered-filters>` above is a placeholder for the saved, serialized filter key/value pairs, not a literal parameter. Apply the stored filter object first, then enforce `page`, `limit` and `countOnly` from the caller. When opening the matching list, remove `countOnly` or send `false` and use that page's existing filter serialization and list component.

Both count-only endpoints return this payload; the example has 8 matches:

```json
{
  "data": [],
  "pagination": {
    "totalItems": 8,
    "totalPages": 8,
    "currentPage": 1,
    "pageSize": 1
  }
}
```

This inner `data` is the list of customer rows, inside the outer standard response envelope's `data`. With the existing unwrapping client, use `result.pagination.totalItems`; do not use `result.data.length` as the count. Count-only deliberately returns no customer rows. Zero matches produce `totalItems: 0, totalPages: 0`.

The exact count-only trigger is query string `countOnly=true`; a boolean `true` serialized by the API client is suitable. Audience-list paging defaults to 15 and caps at 100, unlike the KPI people API's 50/200 limits. Counts are live, computed before pagination, and use the same audience filter builder as the normal list.

Do not submit `segmentId` and assume the customer endpoint resolves its filters. Do not impose the CS reporting month/employee/service query on saved segments: `serviceId` in customer audience filters is not the report's `serviceIds`. Only combine audience filters through the existing screen's supported filter behavior.

### Recognized saved filter keys

Reuse the existing Customer / Registered Customer filter forms and serializers for values and enum choices. The backend stores only the following keys; this is a whitelist, not a new independent filter UI specification.

| Audience        | Recognized filter keys                                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Both            | `journey`, `startDate`, `endDate`, `resourceReferral`, `lastOrder`, `appActivity`                                                                                                                                                                                                                                                                                                                                                                 |
| Customer only   | `type`, `lastActive`, `referral`, `noOrderSince`, `lastOrderWithinDays`, `lastOrderMoreThanDays`, `lastOrderBetweenFromDays`, `lastOrderBetweenToDays`, `frequencyPeriod`, `frequencyOperator`, `frequencyCount`, `bookingOperator`, `bookingCount`, `minBookings`, `maxBookings`, `orderCount`, `timingStatus`, `serviceId`, `serviceBehaviour`, `lastContact`, `aovMin`, `aovMax`, `revenueMin`, `revenueMax`, `promoUsageMin`, `promoUsageMax` |
| Registered only | `contactDate`, `platform`, `loginType`, `preferredService`, `status`                                                                                                                                                                                                                                                                                                                                                                              |

`journey` is normalized and stored as a JSON string by the backend; preserve its existing serialization. Pagination, `searchText` and `segmentId` are not saved segment filters. Registered `contactDate` means the service/CRM record's contact date, not the CS report's effective contact time.

### What managers can track now

Display segment name, source audience, creator, updated time and current matching count, with an Open matching customers action. Use existing list navigation and saved-filter state rather than inventing an unimplemented segment URL parameter.

The following are **not available yet**: sent to CS, assigned, contacted within the case workflow, due/overdue, closed, converted by segment campaign, or immutable send-time segment membership. Do not label a matching audience count as CS workload, add up overlapping segment counts, or create a second CS segment table/count field. A current segment match is not proof that the customer was sent to CS. Live counts can change as customers book or contact records change.

## 9. Customer context and legacy contact support

| Existing endpoint                         | Reuse / limitation                                                                                 |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `GET /customer/:userId`                   | Existing customer profile and full recorded phone context; reuse its existing profile types        |
| `GET /customer/:userId/orders`            | General customer orders, not the filtered KPI `repeat_orders` cohort                               |
| `GET /customer/:userId/activity`          | Existing activity context, not case ownership audit history                                        |
| `GET /customer/last-contact/:userId`      | Retained legacy contact rows with employee and outcome labels                                      |
| `POST /customer/last-contact`             | Existing daily contact update, not immutable Log contact                                           |
| `POST /customer-service/create`           | Existing Registered CRM update; not a new CS case action                                           |
| `GET /customers/:id/summary`              | Journey summary has a masked-phone contract; do not use it for the CS full-phone cell              |
| `GET /customers/:id/journey-canvas`       | Existing Journey context                                                                           |
| `GET /customers/:id/activity` / `/orders` | Existing Journey activity/order context                                                            |
| `GET /customers/:id/contacts`             | Returns an empty array after customer validation; placeholder behavior, not proof of zero contacts |
| `GET /chat/conversations`                 | Existing chat context, not the CS follow-up queue                                                  |
| `PUT /chat/conversations/:id/resolve`     | Resolves a chat; does not complete a CS follow-up case                                             |

`GET /customer/last-contact/:userId` returns an array of `{ remark, lastContactDate, createdAt, csId, contactOutcome, customerService: { firstName, lastName } | null, contactOutcomeLabel }` inside the standard envelope. Historical values can be null. It orders by entered `lastContactDate` descending; the KPI report can order differently because it uses effective time.

Legacy contact write fields are `userId`, `lastContactDate`, `remark`, `isAutoFill`, `contactOutcome`. Actor comes from the token. Success returns `[]` inside the standard envelope. This endpoint can **overwrite an existing row for the same customer and Phnom Penh calendar day**, including the employee. New days can create new rows. Registered CRM logging can merge records within a 13-hour window. Neither supports a guaranteed immutable record of every call by every agent.

Do not wire the design's new Log contact modal to this legacy route and claim that `channel`, `purpose`, customer response, `nextFollowUp`, case closure and historical attribution are saved. Those fields and transitions are not supported. In particular, the existing contact outcome enum is not a drop-in replacement for the new separate connection-result and customer-intent controls.

## 10. KPI calculation rules the UI must communicate

1. **First sales:** customers registered in the selected period form the cohort. PENDING/test/blacklisted accounts are excluded under the report's existing rules. A later purchase does not remove a customer from that historical registration cohort. This population is different from the current Registered Customer screen's audience.
2. **Recorded CS attribution:** the last eligible recorded contact before the first qualifying purchase receives credit. Effective contact time is the later of entered contact date and row creation time. The report prevents earlier entered dates from backdating credit, but it cannot verify an actual call happened.
3. **Purpose limitation:** contacts after purchase do not receive first-sale credit. Contact purpose is not stored, so pre-purchase booking confirmations cannot reliably be separated from sales follow-ups. Display this as a recorded-contact comparison, not proof that CS caused the sale.
4. **Direct sales:** both `ORDER` and `DIRECT_SALE` bookings are considered. A qualifying direct sale with prior qualifying contact can be credited to CS. An unassisted direct first sale appears under other channels, not app conversion. Show `order.type` in details.
5. **Qualifying sale vs revenue:** paid or partially paid qualifying bookings can count as conversions. Cancelled/rejected/fully refunded bookings do not qualify. Revenue and completed paid counts require all active included service lines to be completed and fully paid. Deposits are not completed revenue.
6. **Revenue basis:** completed paid subtotal after discounts, excluding VAT, including the applicable primary service/transport fees. Missing required money data is unavailable. Partial-refund deductions are not implemented; do not label this as fully refund-adjusted net revenue.
7. **Repeat sales:** a completed service must exist before the follow-up. The first follow-up starts an observation episode; later contacts inside it do not restart overlapping windows. Each qualifying order is credited once. Customer metrics deduplicate people; order metrics count orders.
8. **Employee vs team:** a customer touched by multiple employees can appear in multiple employee denominators. Each sale is credited once. Use server team totals rather than adding employee rows. Unknown employees remain in team counts, with Not rated employee rows.
9. **Employee filtering:** the contacted denominator counts customers that employee touched; credited sales/revenue are only those credited to them. The no-contact comparison remains global. Therefore the selected employee's card count plus no-contact count need not equal the registration cohort total.
10. **Service filtering:** filters qualifying bookings by primary service/category. It does not narrow registration/contact denominators. Revenue includes the selected qualifying booking's included service lines; it is not a service-line-only revenue calculation.
11. **Contact activity:** counts retained records within the selected contact period. Unknown outcomes remain separate. A call-button click or legacy CRM change is not a verified call attempt. The same customer can have more than one retained record, but same-day merges mean history is incomplete.
12. **Current state, not frozen accounting:** recomputation uses current booking/payment/contact records. Provisional results can change while observation windows are open; later completion/payment/status edits can change revenue even after windows close.

## 11. Export contract

```http
POST /api/admin/fileExport/cs-overview?month=2026-10&windowDays=30&csId=7
```

Send the same report query as the screen, not a JSON body containing the filters. Successful response:

```http
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=cs-overview-2026-10.xlsx
```

Use the existing blob download mechanism. The workbook currently contains six sheets:

1. Summary
2. By employee
3. First sales
4. Repeat orders
5. Contacts
6. Definitions

It uses the shared KPI computation and includes definitions/availability notes. It does not include unimplemented task tables, an eight-sheet completed-task report, a saved-segment campaign report, or a dedicated export of the currently opened metric list.

The export controller returns plain-text errors: validation uses its request-error status, and unexpected export failures currently return HTTP 400 `Error exporting data to Excel`. Authentication errors may be JSON from middleware. When requesting a blob, inspect an error blob's content type/text before showing the message; do not download an error response as an `.xlsx`.

Existing `/fileExport/existed-customer` and `/fileExport/register-customer` serve their audience-list cohorts. They can be reused by their respective customer pages, but must not silently substitute for a CS KPI metric-list export.

## 12. Portal implementation guidance and consistency

### Reuse established components and infrastructure

The current portal contains the following reusable building blocks. Extend their normal configuration/types only where needed; do not change global defaults to achieve a CS-only layout.

| Need                         | Existing portal source                                                                                                       |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| HTTP/auth/token refresh      | `app/api/api.ts`                                                                                                             |
| Endpoint registry            | `app/api/endpoint.ts`                                                                                                        |
| React Query keys             | `app/constants/query-key-enum.ts`                                                                                            |
| Overview cards               | `app/components/common/overview-card.tsx`                                                                                    |
| Table/pagination flow        | `app/hooks/use-data-table-api.ts`, common table pagination and data-table components                                         |
| Saved Customer filters       | `app/components/data-table/customer-filter-sheet.tsx`, `app/lib/customer-filters.ts`                                         |
| Saved Registered filters     | `app/components/data-table/registered-customer-filter-sheet.tsx` and its existing serializers                                |
| Audience count query pattern | `app/hooks/query/use-customer-segment-counts-query.ts`; its current Customer endpoint must not be used for Registered counts |
| Category/service choices     | `app/hooks/query/use-category-name-query.ts`, existing `category/list-names` endpoint                                        |
| Excel download pattern       | `app/hooks/mutations/use-export-excel-mutation.ts`; extend supported type with `cs-overview` if required                     |
| Shared controls              | Existing Button, Select/Combobox, Dialog, Sheet, Avatar and skeleton/error states used by adjacent pages                     |

These are reuse references, not a claim that the current portal checkout already includes the full CS page integration. Do not confuse the frontend's built-in `CustomerSegment` card definition in `app/lib/customer-segments.ts` with the server's saved CRM segment object; their shapes are different. Reuse existing saved-filter persistence rather than casting one shape to the other.

### Recommended wiring sequence

1. Register the CS reporting/target endpoint constants and response types in the existing locations. Use the existing API client; do not create another Axios instance or token store.
2. Keep one normalized report query. Include period/month/range, fixed window, employee and services in the summary cache key. Include metric, page, limit and sort in each people-list key. Reset list page to 1 when these change.
3. Load summary; render supported sections and availability states independently. Do not let a target/segment failure blank the working report.
4. Open the shared list sheet for a supported number, using the mapping above. Keep full phone and customer/order context links. Close/reset or refetch an open list when its report scope changes.
5. Fetch month-specific targets using a cache key containing month. Save all three values atomically, disable repeat submissions, then invalidate that month's target query. Do not invalidate unrelated dashboard targets.
6. Load saved segments by audience, fetch matching counts using that audience's existing list query, and open the existing matching list with saved filters applied. Bound/deduplicate count requests when many segments exist; reuse React Query keys for identical queries.
7. Wire monthly export with the exact current report query. Leave per-metric Export list disabled until its backend contract exists.
8. Keep case/assignment/deadline actions and unavailable filters disabled. Do not optimistically change ownership or contact history for 501 endpoints.

No WebSocket/SSE contract or automatic cross-session update is provided for this feature. After a successful existing contact write in the same portal session, invalidate the relevant customer/detail/count and CS summary/people queries. Another employee's browser needs refetch, window-focus refresh, manual refresh or an explicitly configured polling policy. A cached page should not be described as real-time.

### Large lists and interaction states

- Employee summary arrays arrive in full. For many employees, use the shared table with local pagination/search over the loaded array or an existing bounded-height pattern. Keep the authoritative team footer separate from the visible page, so it does not become a page subtotal. Do not claim server-side employee search/paging.
- KPI people lists use server pagination. Keep the existing page-size and Next/Previous controls; reset correctly when changing metric/filter. A 640px desktop sheet should wrap secondary details without sideways page scrolling; adapt width for smaller screens using the shared Sheet behavior.
- Links should have clear purpose and keyboard focus. Reuse the verified customer Journey route `/customer/:id/journey` and existing order navigation `/order?bulkOrderId=...` where a bulk order ID exists. Do not generate that route from a numeric order ID alone or invent employee profile links.
- Show measured zero counts/money as text rather than misleading empty links. A revenue amount alone does not establish a qualifying row count: use the corresponding count/metric and disable navigation when there are no rows.
- Use skeletons during initial loading, existing error/retry patterns on failures, and an explicit empty state for a successful empty list. Never turn loading/error states into zero KPI values.
- If previous data is retained during filter changes, visibly mark it as updating and disable actions tied to the previous scope until the new response arrives. Do not show old data under new filter labels as if current.
- Keep missing actions visible only where the design calls for a placeholder, with “Not implemented yet”. Problems found is Out of scope. No fake counts or duplicate cards for the same group.
- Customer “Never contacted” and “Overdue for rebooking” can be shown as separate live audience opportunities through existing filters. They are not “Picked up, not contacted” or an employee's overdue task deadline, and should not duplicate the daily task cards under different labels.

## 13. Error and availability checklist

| Situation                         | Expected handling                                                                        |
| --------------------------------- | ---------------------------------------------------------------------------------------- |
| 401                               | Existing portal refresh/login flow                                                       |
| 403 target access                 | Explain permission requirement; keep permitted reporting usable                          |
| 422 report filter / target value  | Read standard message and optional field; allow correction                               |
| 404 route                         | Check deployed API version/base path; do not interpret as an empty report                |
| 409 duplicate segment name        | Inline name conflict                                                                     |
| 400 segment dependency protection | Display dependency explanation; keep segment                                             |
| 501 Journey CS placeholder        | Read `error.code/message/field`; show Not implemented yet                                |
| 503 target storage                | Disable target editor; storage rollout required                                          |
| Unexpected HTTP/network failure   | Existing error/retry state; no replacement zeroes                                        |
| Partial availability              | Render supported data with its limitation; do not disable an entire report unnecessarily |

No partial-availability flag proves SQL is installed, a permission is granted, or a deployment is current. In particular, `availability.monthlyTargets.status=partial` describes branch capability; target GET still decides runtime readiness. Preserve both transport errors and capability explanations.

## 14. Rollout, acceptance and open dependencies

### Deployment order

1. Confirm intended staging API/database and deploy the matching backend branch/version.
2. Apply/review the additive target SQL and verify existing `CS-OVERVIEW` VIEW/UPDATE permission assignments. Do not grant them broadly as a frontend workaround.
3. Smoke-test summary, people, target GET/PUT and both audience count-only endpoints using authorized staging accounts.
4. Integrate the portal using existing components and the contracts above. Keep all missing workflow actions marked unavailable.
5. Reconcile representative source records against summary, paged lists and Excel before enabling management use of the report.

### Frontend and QA acceptance cases

- [ ] Month, employee and services are identical across summary, linked lists and export.
- [ ] A first purchase in the following calendar month stays in its original registration cohort when within the observation window.
- [ ] Deposits count where eligible as conversions, but not as completed paid revenue/orders.
- [ ] CS-attributed and unassisted direct sales appear in their correct groups, with direct-sale labels in details.
- [ ] Employee-row totals and deduplicated team totals are not incorrectly summed or relabeled.
- [ ] Unknown employees remain visible and linked through `csId=unknown`, with Not rated rows.
- [ ] A known zero, null money, null reached result, no denominator and missing feature all display differently.
- [ ] Contact list tiles reconcile before pagination; changing to a reached subset does not relabel it as all attempts.
- [ ] Multiple repeat orders for one customer have distinct row keys and links; unique-customer lists deduplicate them.
- [ ] Page changes and revenue sorting do not omit/duplicate unchanged records; scope changes reset page and stale state.
- [ ] Full recorded phones appear in CS lists without changing Journey's separate masked-phone API.
- [ ] Unsaved target month returns defaults without saving; a zero target stays zero; another month's targets are unaffected.
- [ ] Target VIEW-only account cannot PUT; missing target storage shows a deployment state rather than unsaved defaults.
- [ ] Both saved-segment audiences load, use the correct filters, and count with `pagination.totalItems`, not loaded row length.
- [ ] Segment count/open-list results match within unchanged data; overlapping segments are not summed as unique workload.
- [ ] Protected segment deletion is blocked; existing coupon/banner links remain intact.
- [ ] Legacy contact updates trigger relevant refetches; no claim is made that all attempts or cross-browser updates are real-time.
- [ ] Missing follow-up and per-list export actions do not submit writes, show invented progress, or report zero work.
- [ ] Changes to CS rendering do not change shared-component defaults or the existing Customer/Registered/Journey pages.

### Remaining backend/product dependencies

| Dependency                                                      | Why it blocks the full design                                                                            |
| --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Canonical CS case storage and transitions                       | Assigned work, Mine/All open/Unassigned, pickup collision handling, reassign, deadline/on-time reporting |
| Immutable attempts with purpose/channel/result/intent and audit | Reliable multi-contact history, booking-confirmation exclusion, historical employee attribution          |
| Send-time source/group and segment references                   | Correct queue grouping, saved-segment follow-up progress and duplicate-send protection                   |
| Scoped no-more-calls handling                                   | Respect contact suppression without disabling unrelated bookings/promotions                              |
| Agent/lead access policy                                        | Secure cross-employee views and reassignment; no invented permissions                                    |
| Partial refund data/rule                                        | Refund-adjusted revenue accuracy                                                                         |
| Repeat refunded/scheduled list definition                       | Consistent status tiles and corresponding list universe                                                  |
| KPI metric-list export contract                                 | Correct Export list action without substituting another audience                                         |
| Queue update notification contract                              | Reliable cross-session ownership/progress updates                                                        |

Reuse `User` identities, existing customer/booking records, `CustomerSegment`, existing role permissions and shared KPI computation. Do not add duplicate customer/segment/target fields, a second reporting engine, or overwrite-only history to imitate the missing workflow.

### Verification evidence and limits

The preceding backend implementation verification recorded **181 passing tests across six suites**, covering KPI/list/export rules, targets and registered count-only behavior. This handoff was checked against the current routes/controllers/services and portal integration patterns. Documentation checks do not replace staging reconciliation. SQL installation, live database totals, frontend integration and deployment are not certified by that unit-test result.

Primary source files for future contract changes:

- `routes/admin/csOverview.js`
- `services/csOverview/csOverviewService.js`
- `services/csOverview/csOverviewTargets.js`
- `controllers/admin/csOverview/targets.js`
- `controllers/admin/fileExport/csOverviewExcel.js`
- `routes/responseWrapper.js`
- `routes/admin/journey.js` and `services/journey/journeyHttp.js`
- `controllers/admin/customerSegment/` and `services/customer/customerFilters.js`
- `controllers/admin/customer/registerCustomer.js` and `existedCustomer.js`
- `controllers/admin/customer/customerLastContact.js` and `getLastContact.js`
- `sql/cs-overview-targets.sql`
