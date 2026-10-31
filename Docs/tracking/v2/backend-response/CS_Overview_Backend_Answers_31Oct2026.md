# CS Overview: backend answers

Updated: 2026-10-31
From: backend (branch `feature/cs-kpi`, not committed or deployed yet)
Replying to: `CS_Overview_Backend_Questions.md` (2026-10-31)

Short version: B1, C1 and C2 are now built on the backend. A1 needs one SQL script plus a dev deploy. B2 and B3 are confirmed as you described. C3 needs nothing. D1 and D2 are product decisions, so we've asked the lead.

## A. Needed before you can test

**A1. Target storage on dev.**
- The table script is `sql/cs-overview-targets.sql`. It is safe to run more than once. It creates `cs_overview_targets` and gives the **ADMIN** role `CS-OVERVIEW` VIEW, ADD, UPDATE and EXPORT.
- Dev runs `feature/cs-kpi` and has this table only after we deploy and apply the script. We'll tell you when that's done. Until then, `GET /targets/:month` returns **503** ("CS target storage is not installed yet"), so keeping the editor disabled on 503 is correct.
- Role access: other roles need `CS-OVERVIEW` set on the Roles & Permissions page. A user without the permission gets **403**.

## B. Decisions

**B1. Server-side protection: yes, added.**

| Endpoint | Requires |
| --- | --- |
| `GET /api/admin/cs-overview` | `CS-OVERVIEW` VIEW |
| `GET /api/admin/cs-overview/people` | `CS-OVERVIEW` VIEW |
| `POST /api/admin/fileExport/cs-overview` | `CS-OVERVIEW` **EXPORT** |
| `GET /api/admin/cs-overview/targets/:month` | `CS-OVERVIEW` VIEW |
| `PUT /api/admin/cs-overview/targets/:month` | `CS-OVERVIEW` UPDATE |

- When the permission is missing, the API answers 403 with `{ message: "CS-OVERVIEW VIEW permission is required", data: { field: "permission" } }`.
- Roles are read like login does: `user.role`, or `user.type` when that is null. There is no automatic admin access, which matches the portal.
- Please gate the Export button with `CS-OVERVIEW` EXPORT, the same as the API.

**B2. Targets outside a single month: confirmed.** Show rates without an on-target or under-target indicator, with the note "Targets are set per month". Don't average monthly targets.

**B3. Target mapping: confirmed.**
- `firstSaleConversionTarget` applies to `firstSales.contacted.rate` and each `firstSales.contacted.byEmployee[].rate`.
- `repeatConversionTarget` applies to `repeatSales.total.rate` and each `repeatSales.byEmployee[].rate`.
- `completedOnTimeTarget`: show the target only. There is no actual on-time value yet.
- Compare with `rate * 100 >= target`, so a rate equal to the target meets it.
- When `rate` is `null` (nothing to divide by, or "Employee not recorded"), show no indicator.

## C. Data

**C1. Employee list: changed.** The summary's `employees` now lists everyone with activity in the report **plus every active CS staff member**. CS staff means users with `type = CUSTOMER_SERVICE`, or with `CUSTOMER_SERVICE` in their `role`.

```json
"employees": [
  { "csId": 8, "name": "Sokha Chan", "hasActivity": false, "isCsStaff": true },
  { "csId": 7, "name": "Dara", "hasActivity": true, "isCsStaff": true }
]
```

- Use this list for the employee filter. You don't need a separate staff endpoint.
- The by-employee tables (`byEmployee`) still contain only employees with data in the period. To show everyone, add a zero row for each `employees` entry with `hasActivity: false`.
- Selecting an employee with no activity returns valid empty numbers: 0 counts, and N/A for rates.

**C2. Segment counts: new batch call.** Use `POST /api/admin/customer/counts`:

```json
{ "audience": "customer", "queries": [ { "...": "the same params you send to customer/existed" } ] }
```

- `audience` is `customer` (counts like `GET customer/existed`) or `registered` (counts like `GET customer/register-customer`). It defaults to `customer`.
- `queries` holds 1 to 50 filter sets. Paging keys (`page`, `limit`, `countOnly` and so on) are ignored.
- The response is `{ audience, counts: [12, 0, null, ...], errors: [{ index: 2, message: "Count is unavailable" }] }`. Counts come back in the same order as `queries`, and a failed count is `null` instead of failing the whole call.
- Each count uses exactly the same query as that list's `countOnly` path (a test checks this), so the numbers match `pagination.totalItems`.
- The server runs at most 4 counts at a time. You can send the baseline and all segment params in one request.

**C3. Order links.** Every order and direct sale is created with a `bulkOrderId`; the column is required in the model, and direct-sale edit/cancel routes are keyed by it. So `bulkOrderId` should not be `null` for current data. Your fallback (plain order id, no link, when it is null) is fine for any old row. Since both `ORDER` and `DIRECT_SALE` rows come back, use `order.type` if direct sales need to open the direct-sale screen instead of `/order`.

## D. Roadmap

- **D1. Case storage and `/cs/*` endpoints:** no date yet. This is a product and priority decision, so we've asked the lead and will reply. Until then, plan for "Not implemented yet" in this release.
- **D2. "Problems found by CS":** it is out of scope for this release (it's not in the 1 October brief). Whether it's removed permanently is the lead's decision, so please don't remove it from the design until they confirm.
- **D3. Per-list export:** added to the backend backlog. Keep the button disabled.

## E. Noted

Agreed on all points: `api.ts` unwrapping, the same query for summary, people and export, resetting to page 1, and "Unavailable" / "— Not implemented yet". Please send any mismatches you find when you compare against staging.
