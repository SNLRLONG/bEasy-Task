# CS Action and Performance: backend reply to the Performance review asks (6 Nov, afternoon)

**To:** Portal · **From:** Backend · **Date:** 6 Nov 2026 · **Branch:** `codex/cs-action-performance-v1`
Replies to "New asks from the Performance review (6 Nov)" (P1–P5). Thanks for verifying B1 on api-dev.

## Summary

| # | Answer | Portal work |
|---|---|---|
| P1 | Filters: not yet. Names and values agreed below; business filters come with sections A–C (after E1). | Do not send them yet (API returns 400). |
| P2 | **Built:** Direct Sale order counts and creator split, no money, in `sections.directSales.measured`. Everything else in A–C stays blocked (table below). | Show the measured counts. |
| P3 | `valueStates` = the meaning of 0, `null` and pending. Use it as a legend. | Optional legend/tooltip. |
| P4 | **Built:** `GET /api/admin/cs/scheduler/status`. | Manager status card. |
| P5 | Settings API will be built once management approves the first values; shape below. Until then managers ask backend. | Nothing yet (keep in backlog). |

Backend deploy needs: this branch on api-dev, and run `sql/cs-action-scheduler-status.sql` (additive, safe twice). We will tell you when both are done.

---

## P1 Filters for `GET /performance`

Today: `from`, `to`, `employeeId` only. Any other key returns **400** "Filter X is not supported yet". Please don't send new keys until we confirm.

The brief's filters (source, service, first/repeat, App/Direct Sale) split **sales and revenue**, which live in sections A–C. Those sections wait for E1, so the filters ship together with them. These are the names and values we will use, so you can prepare the UI:

| Filter | Query key | Values | Applies to |
|---|---|---|---|
| App or Direct Sale | `orderType` | `ORDER`, `DIRECT_SALE` | A–C (and Direct Sale counts) |
| First or repeat customer | `customerType` | `first` (customer's first verified sale), `repeat` | A–C |
| Service | `serviceCategoryId` | `categories.id` (use the existing category list) | A–C, follow-up tasks with an order |
| Source | `source` | Unified source keys from the Telegram/app referral source list (`facebook`, `googleads`, `tiktok`, …). The list is final after `codex/telegram-customer-intake` merges, which renames the old keys. | A–C, Chat |
| Contact channel | `channel` | `phone`, `telegram`, `in_app_chat` | Follow-up (contacts) |
| Task type | `category` | `new_orders`, `registered`, `existing`, `after_service` | Follow-up (tasks) |

**When:**
- The follow-up filters (`channel`, `category`) don't depend on E1. Tell us if you want them before A–C and we'll add them to `/performance` **and** `/performance/records`, so every number still opens the same list.
- The others come with A–C after E1 is approved.

## P2 What can be shown now

| Item | Status | What blocks the rest |
|---|---|---|
| Direct Sales: order count, unique customers, completed, split by creator | **Available now** (`measured`, below) | Revenue, net and "accountable owner" need E1 (revenue/VAT/refund policy, owner rule). |
| App non-assisted / assisted (A, B) | Blocked | Both need the approved **sales-assistance window** (E1). Without it we cannot say which app order was "assisted", so even counts would be guesses. |
| Chat closing rate | Blocked | E1 (chat opportunity cohort/window) **and** the Chat branch merge (Chat sales tasks). |
| Review coverage | Blocked | E1 review eligibility (which completed orders should get a request). |
| Scope coverage | Blocked for now, **no E1 needed** | Needs the scheduler on (B2). Until then tasks come only from manual Sync, so coverage would look falsely low. We'll enable it after the scheduler has run. The "on time" part also needs the SLA (E1). |
| Outreach without sale | Blocked | E1 observation window. |

New response part. The rest of `directSales` is unchanged (`available:false`, `value:null`, `reason`):

```json
"directSales": {
  "available": false, "value": null, "reason": "Primary accountable-owner governance, revenue recognition, refund allocation and VAT policy await approval",
  "measured": {
    "orders": 12, "uniqueCustomers": 10, "completedOrders": 9,
    "scope": "team",
    "byCreator": [
      { "employeeId": 7, "employeeName": "Example Sales", "orders": 11, "uniqueCustomers": 9, "completedOrders": 8 },
      { "employeeId": null, "employeeName": "Creator not recorded", "orders": 1, "uniqueCustomers": 1, "completedOrders": 1 }
    ],
    "definition": "Direct Sale bookings created in the period (primary line), paid, deposit or completed; …"
  }
}
```

- Count = Direct Sale **bookings created in the period** that are paid, deposit or completed. Cancelled, rejected, refunded, unpaid and test/blacklisted customers are excluded. This is the same rule as the CS KPI report.
- `byCreator` = the staff who **created** the booking (`orders.sale_id`). Label it **"Created by"**, not "Owner"; the accountable-owner rule is still E1. Show `employeeId:null` rows as "Creator not recorded"; don't drop them.
- `orders` and `completedOrders` add up across rows. `uniqueCustomers` in the total counts each customer once, so it can be lower than the sum of rows.
- Not filtered by `employeeId` (same as A–C). A CS user without `CS_PERFORMANCE / UPDATE` gets `scope:"own"`, with only their own row and their own totals.
- **No money fields.** Do not compute revenue from this.

## P3 `valueStates`

`GET /performance/definitions` → `data.valueStates` explains the three ways a number can look:

| Key | Meaning | How to show |
|---|---|---|
| `zero` | Measured with the available evidence and filters; nothing matched. | Show `0`. |
| `null` | Unavailable, not configured or not applicable. The item's `reason`/`status` says why. | Show "Not available yet" (never 0). |
| `pending` | Window or work still open; not a final loss. | Show as provisional ("pending"). |

It's text for a legend or tooltip. There's nothing to calculate.

## P4 Scheduler status

`GET /api/admin/cs/scheduler/status` needs `CS_PERFORMANCE / VIEW` (ADMIN always). Example `data`:

```json
{
  "health": "ok",
  "timeZone": "Asia/Phnom_Penh",
  "config": { "enabled": true, "operatorId": 1234, "operatorName": "CS Scheduler", "lookbackMinutes": 1440, "updatedBy": 3, "updatedAt": "2026-11-07T02:00:00.000Z" },
  "checkpoint": { "windowFrom": "2026-11-06T02:00:00.000Z", "windowTo": "2026-11-07T02:00:00.000Z", "lastOrderId": 51234, "windowFinished": true },
  "lastRunAt": "2026-11-07T03:05:00.000Z",
  "lastSuccessAt": "2026-11-07T03:05:00.000Z",
  "minutesSinceLastSuccess": 3,
  "lastErrorAt": null,
  "lastError": null,
  "lastResult": { "bookingsChecked": 10, "tasksCreated": 4, "alreadyHadTask": 6, "notEligible": 0, "nextAfterId": 51234, "windowFinished": true },
  "scopeTasksCreatedLast24h": 37,
  "notes": "…"
}
```

| `health` | Meaning | Show |
|---|---|---|
| `disabled` | Scheduler is off (today). | "Automatic creation is off. Use Sync new orders." |
| `never_run` | Enabled but no run recorded yet. | Neutral "Waiting for first run". |
| `ok` | Latest run succeeded. | Green, plus "last run X min ago". If `minutesSinceLastSuccess` is much higher than the cadence (we'll tell you the approved cadence, for example more than 30 minutes for a 5-minute job), show a "may be stuck" warning. |
| `error` | Latest run failed after the latest success. | Red, plus `lastError` text. Managers can still use Sync new orders. |

- `config` / `checkpoint` can be `null` if setup was never installed. `lastRunAt`, `lastResult` and the error fields are `null` until the first run after `sql/cs-action-scheduler-status.sql` is applied.
- `lastOrderId` is the booking id the current window has reached. It's for support/debugging, so show it small.
- Suggestion: a small card next to "Sync new orders" (manager only), with refresh on open and a manual refresh button. No need to poll fast.

## P5 Where policy values (E1) will live

Today: nothing is configured, and values would be set by backend in the database. We will **not** build an empty settings screen before management approves values, because a value the system doesn't apply yet would mislead managers.

Plan once management approves the first values (we build the API and wire each value into the logic at the same time):

- `GET /api/admin/cs/policies` (CS_PERFORMANCE VIEW) and `PUT /api/admin/cs/policies/:key` (CS_PERFORMANCE UPDATE), audited with who, when, old → new and an effective date. Past values are kept, so reports use the value that applied at the time.
- One item per policy:

```json
{ "key": "scopeCallSlaMinutes", "label": "Scope call deadline", "value": 120, "unit": "minutes",
  "status": "approved", "effectiveFrom": "2026-11-10T00:00:00+07:00", "updatedBy": 3, "updatedAt": "…",
  "explanation": "Time after a confirmed booking by which the scope call is due." }
```

- Planned keys: `scopeCallSlaMinutes`, `retryIntervalMinutes`, `maxAttempts`, `escalationAfterMinutes`, `salesAssistanceWindowDays`, `reviewDelayDays`, `reviewMaxAttempts`, `reviewEligibility`, `revenueVatTreatment`, `refundAllocation`.
- **Until then:** a manager sends the approved value to backend (message or ticket), and backend applies it with an audited SQL change and confirms. Keep the settings screen in the backlog. We'll send the final field list together with the first approved values.

## Verification

43 CS tests pass, including PostgreSQL temporary fixtures run on a local PostgreSQL:
- **Scheduler status:** works before and after the status SQL; a successful run sets `ok`; a failing run (backlog over 31 days) sets `error` with the message and keeps the last success time; inactive staff are refused.
- **Direct Sale counts:** count only primary, paid/deposit/completed, non-test bookings; group "Creator not recorded"; give CS viewers only their own row.

Nothing was run on beasy-dev or production.
