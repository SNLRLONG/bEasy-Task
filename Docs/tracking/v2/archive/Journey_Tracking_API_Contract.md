> **Archived:** early mock-based contract. The current contract is [../api/Journey_Backend_API_Spec.md](../api/Journey_Backend_API_Spec.md).

# Journey analytics API contract

Proposed HTTP contract for the portal. The admin app does not call these routes. It calls the same shapes from `bEasy-portal/app/lib/journey-analytics/service.ts`, which reads a deterministic in-memory fixture.

Every payload includes:

```json
{
  "dataSource": "mock",
  "connectedToBigQuery": false,
  "reportingTimeZone": "Asia/Phnom_Penh",
  "dataAsOf": "2026-10-02T09:00:00+07:00",
  "period": { "preset": "30d", "from": "2026-09-03", "to": "2026-10-03" },
  "filters": {
    "period": "30d",
    "serviceId": null,
    "platform": null,
    "appVersion": null,
    "audience": "all",
    "viewBy": "journey",
    "campaignId": null
  },
  "filtersIgnored": [],
  "result": {}
}
```

`to` is exclusive. `filtersIgnored` lists filters that did not apply, so the UI can say so.

Authorisation for a future live API: the caller must be a signed-in portal user. These test screens do not add a permission module. In production builds the screens render a disabled notice unless `VITE_JOURNEY_ANALYTICS_TEST=true`.

Cache: responses are pure functions of the fixture and the query. A live API should cache the daily read models, not the raw export, and vary the cache on the filter set. Do not send raw event collections to the dashboard.

Errors use `AnalyticsQueryError`:

| Code | When | Body |
| --- | --- | --- |
| `INVALID_FILTER` | Custom dates missing, not `YYYY-MM-DD`, `from >= to`, or a bad cursor or stage | `{ "code", "message", "field" }` |
| `NOT_FOUND` | Unknown journey id | Same shape |

Empty funnels return stages with count 0, not an error. The UI then shows an empty state.

---

## Shared query

`period` is `today`, `7d`, `30d`, `90d`, or `custom`. Custom requires `from` and `to`.

`serviceId` integer or null. `platform` is `ios` or `android` or null. `appVersion` string or null. `audience` is `all`, `registered`, `guest`, or `existing_customer`. `viewBy` is `journey` or `customer`. `campaignId` string or null.

Pagination: `limit` default 20 or 25, maximum 100. `cursor` is an offset encoded as a string. Response:

```json
{ "items": [], "total": 0, "nextCursor": null }
```

---

## `GET /analytics/journey/overview`

KPIs for the period and the previous period of equal length.

`result.activeUsers`, `sessions`, `bookings`, `revenueUsd`, and `bookingConversion` are `{ value, previous, delta, unit }`. Units are `people`, `sessions`, `orders`, `usd`, and `ratio`.

Also: `serviceViews`, `checkoutWithoutPurchase`, `paymentFailed`, `cancelledOrders`, `completedServices`, a `definitions` map, and the booking `funnel`.

```json
{
  "result": {
    "activeUsers": { "value": 120, "previous": 100, "delta": 0.2, "unit": "people" },
    "bookings": { "value": 40, "previous": 0, "delta": null, "unit": "orders" },
    "bookingConversion": { "value": 0.11, "previous": 0.09, "delta": 0.2222, "unit": "ratio" },
    "definitions": {
      "bookings": "Succeeded orders whose payment time falls in the period. This can differ from the funnel purchase count, which counts journeys that started in the period."
    }
  }
}
```

Definitions are the ones in [Journey_Tracking_Analytical_Model.md](../architecture/Journey_Tracking_Analytical_Model.md). `delta` null means the previous period was zero and this period was not.

## `GET /analytics/journey/trend`

`result.points[]`: `{ date, activeUsers, sessions, bookings, revenueUsd }`. One point per calendar date in the period. Bookings on a point use order time for that date.

## `GET /analytics/journey/funnel/booking`

`result` is a funnel:

```json
{
  "funnel": "booking",
  "viewBy": "journey",
  "countingUnit": "Unique journeys started in the period",
  "definition": "A journey counts at every stage up to its furthest stage. Missing intermediate events are counted and flagged as imputed. Purchase is the backend order event.",
  "filtersIgnored": [],
  "overallConversion": 0.11,
  "monotonic": true,
  "stages": [
    {
      "stage": "service_viewed",
      "eventName": "service_viewed",
      "label": "Service viewed",
      "count": 1000,
      "fromPrevious": 1,
      "dropped": 580,
      "droppedPct": 0.58,
      "increase": null,
      "medianMinutesFromPrevious": null,
      "timingSample": 0,
      "imputed": 0
    }
  ]
}
```

Stage order: `service_viewed`, `booking_started`, `details_completed` (event `booking_details_completed`), `slot_selected`, `checkout_started`, `purchase_completed`.

## `GET /analytics/journey/funnel/booking/stages/{stage}`

Journeys that reached that stage, paginated. Unknown stage → `INVALID_FILTER`.

## `GET /analytics/journey/funnel/registration`

Same stage shape. `viewBy` is `reg_flow` or `identity`. `filtersIgnored` contains `serviceId` when a service filter was sent. Stages: `registration_started`, `otp_viewed`, `otp_verified`, `signup_viewed`, `registration_completed`.

## `GET /analytics/journey/funnel/acquisition?mode=all_opens|first_opens`

People, not journeys. Registration and booking steps can rise relative to the previous step. `filtersIgnored` may contain `serviceId`. Ad click is not a stage.

## `GET /analytics/journey/services`

`result.services[]`: `serviceId`, `serviceName`, `journeys`, `bookingStarted`, `checkoutStarted`, `purchases`, `conversion`, `checkoutWithoutPurchase`, `paymentFailed`, `cancelled`, `medianMinutesToPurchase`.

Sorted by journeys descending. The service filter limits which services appear; rates inside a row are still journey-based.

## `GET /analytics/journey/retention`

```json
{
  "result": {
    "split": {
      "customersWithPurchase": 30,
      "firstTime": 22,
      "returning": 8,
      "repeatRate": 0.2667,
      "definition": "Among customers with a succeeded order in the period, returning means their first succeeded order was before the period."
    },
    "lifecycle": { "visitor": 4, "first_order_done": 10, "lapsed": 1 },
    "cohorts": [
      {
        "weekStart": "2026-09-01",
        "size": 12,
        "weeks": [{ "weekIndex": 1, "customers": 2, "rate": 0.1667 }]
      }
    ],
    "cohortDefinition": "Weekly demo grain. Production retention in the automation guide is monthly."
  }
}
```

Unfinished cohort weeks use `customers: null` and `rate: null`.

## `GET /analytics/journey/campaigns`

`result.attribution` states the 7-day last-touch rule and that it is not holdout lift. `result.campaigns[]`: `campaignId`, `name`, `source`, `appOpens`, `registrations`, `journeys`, `purchases`, `revenueUsd`, `journeyConversion`. Null campaign id is organic.

## `GET /analytics/journey/follow-up`

Interest rows at `very_high` or `high` whose last activity is in the period. `result.definition` restates the bands. Rows include identity, service, interest, status, payment outcome, and last activity.

## `POST /analytics/journey/segments/preview`

Body:

```json
{
  "logic": "AND",
  "rules": [
    { "id": "1", "field": "behaviour", "op": "HAS", "value": "service_viewed", "windowHours": 168 },
    { "id": "2", "field": "behaviour", "op": "HAS_NOT", "value": "booking_started" }
  ],
  "exclude": [{ "id": "3", "field": "crm_segment", "op": "IS", "value": "lost_90d" }]
}
```

`result`:

```json
{
  "estimatedCount": 14,
  "reachableByPush": 11,
  "identityKeys": ["c:1111"],
  "sample": [{ "identityKey": "c:1111", "label": "Customer 1111" }],
  "previewOnly": true,
  "saved": false,
  "note": "Preview only. This does not save a segment, export identifiers, or activate a campaign."
}
```

`sample` is at most eight identities. Query filters still apply, so the estimate is for the reporting period.

## `GET /analytics/journey/events`

Query: `layer=raw|clean`, `eventName`, `sessionId`, `identity`, `quality=all|issues`, `direction=asc|desc`, plus pagination.

Raw audience uses the customer id on the event as exported, before linking. Clean audience uses the resolved id. Raw does not include backend-derived purchase rows. Clean can include them with `derived: true`.

## `GET /analytics/journey/journeys` and `GET /analytics/journey/journeys/{id}`

List uses the journey filter (started in the period). Detail returns the journey, its order if any, the interest row, and its clean events. Unknown id → `NOT_FOUND`.

## `GET /analytics/journey/orders`

Orders whose `created_at` is in the period, after the same dimension filters. Audience uses the order’s customer id.

## `GET /analytics/journey/quality`

Counts for the period: raw rows, clean rows, meaningful rows, quarantined, duplicate event ids, technical duplicates, proposed events, journeys with skipped steps, and people with analytics consent off. `issues` is a sample of at most 40 flagged clean events.

## `GET /analytics/journey/schema` and `GET /analytics/journey/filters`

Schema returns the event catalog (`proposed`, `inDemo`), services, campaigns, and CRM segment ids. Filters returns the option lists for the controls. Neither is period-scoped. Both still set `dataSource` and `connectedToBigQuery`.

---

## v2.1 additions (proposed, 8 Oct 2026)

> **Superseded for the backend team by [Journey_Backend_API_Spec.md](../api/Journey_Backend_API_Spec.md)** (exact paths, request and response shapes as the portal calls them). Keep this section as background only.

Screens: [Journey_Tracking_UI_Flow_v2.1.md](../requirements/Journey_Tracking_UI_Flow_v2.1.md). Definitions: [Journey_Tracking_Analytical_Model.md](../architecture/Journey_Tracking_Analytical_Model.md) section 4a. The portal will call the live backend (`/api/admin/...` style, to be aligned with the backend team). The mock-only envelope above (`dataSource`, `connectedToBigQuery`) is kept for the fixture phase.

Shared query additions: `customer_type` = `all | registered_no_order | existing | guest | saved_segment` (+ `segment_id`), `count_by` = `customer | journey` (default `customer` for the booking funnel), `compare` = `previous_period | none`, `period` gains `60d`, `unit` = `people | devices` (registration), `channel`, `campaign_id`, `adset_id` (acquisition).

### Funnels

| Method | Path | Notes |
| --- | --- | --- |
| GET | `/funnels/booking` | Adds per stage `reached`, `dropped`, `continued`, `abandoned`, `not_abandoned_yet`. KPIs: `viewed`, `purchased`, `recoverable_now { customers, open_value }`, `biggest_drop`. Counts also under `compare` when requested |
| GET | `/funnels/registration` | 5 stages. `reasons[]` for the OTP-requested stage: `sms_not_delivered`, `gateway_error`, `wrong_code`, `blocked`, `expired_or_left`, each with `count` and `pct` |
| GET | `/funnels/acquisition` | 9 stages. Stage 1 `source: "meta_total"`, `listable: false`. `afterPurchase { rows[] }`. `campaigns[]` for the compare table |
| GET | `/funnels/{funnel}/stages/{stage}/people` | `state=reached|dropped`, `reason` (registration), `abandoned=yes|no|all`, `q` (name or phone), `sort=last_activity`, `cursor`. Response: `header { definition, count, unit, filters }`, `strip { abandoned, not_yet }`, `reach { push, cs, ads_only, suppressed }`, `items[]`, `next`. `items[]`: `kind customer|device`, masked phone, device, app version, status tag, furthest step, service, source, interest, reach, value, last activity, OTP fields. Stage with `listable:false` returns `422 NOT_LISTABLE` |
| GET | `/funnels/{funnel}/stages/{stage}/people/export` | Same query plus `columns[]`. Guests export device id. Suppressed rows carry `do_not_contact: true` |
| GET | `/abandon-rules` · PUT `/abandon-rules` | Admin only. Six rules with `step`, `minutes`. Response notes effective-from time |

### Actions

| Method | Path | Notes |
| --- | --- | --- |
| POST | `/actions/recovery-push/preview` | Body: list query + selected ids (or "all"). Returns the eligibility ladder: `will_send`, `skipped_weekly_limit`, `removed_just_bought`, `not_abandoned_yet`, `push_off`, `guests`, `suppressed` |
| POST | `/actions/recovery-push` | Body: preview body + `message { en, km }`, `promo_id?`, `when`. Re-checks orders and Abandoned status at send. Returns `sent`, `skipped` by reason. Writes `action_send` rows |
| POST | `/cs/cases` | Body: list query + ids, `reason: sales_follow_up|booking_help`, `due_at`, `priority`, `note`. No assignee. Returns `created`, `left_out_suppressed` |
| POST | `/cs/cases/{id}/pick-up` | First caller becomes owner. `409` if taken |
| POST | `/cs/contacts` | Log contact: `customer_id`, `channel`, `result`, `said?`, `problem?`, `purpose`, `next_follow_up?`, `note?`. `said = no_more_calls` adds suppression |
| GET | `/cs/overview` | `today[]` per agent, `problems`, `first_sales { contacted, none, byAgent[], other }`, `repeat { byAgent[] }`, `provisional { customers, windowEnds }`, `targets` |

### Customer page

| Method | Path | Notes |
| --- | --- | --- |
| GET | `/customers/{id}/summary` | Header (status, device, source), KPIs, journey at a glance, numbers, recent activity |
| GET | `/customers/{id}/ai-summary` · POST `/refresh` | `what_happened`, `next_step`, `best_offer`, `written_at`, `language`. Push text is a draft only |
| GET | `/customers/{id}/journey-canvas` | Source node, service nodes (furthest step, status, abandoned_for, value), after-purchase nodes |
| GET | `/customers/{id}/sessions?service_id=` | Sessions newest first with `status`, `events`, `recording_url` |
| GET | `/customers/{id}/event-log` | As in the architecture guide, plus `by` (`app|backend|system|meta|team`), `promo_source`, order sub-events, pre-sign-up events marked `guest_device`, filters `source`, `status` |
| GET | `/customers/{id}/orders` · `/contacts` | Tabs |

### Audiences

`POST /audiences/estimate` returns `estimated`, `reach { push, ads, cs_only }`, `left_out_suppressed`. `POST /audiences` accepts `built_from { funnel, stage, state }`, `locked_rules[]`, `destinations[]` (`push`, `meta`, `tiktok`, `csv`). Saved list status: `synced | syncing | meta_sync_failed`; `POST /audiences/{id}/retry-sync`. Name uniqueness: `409 NAME_TAKEN`. `GET /integrations` reports `meta.connected`, `tiktok.connected`.

### Errors added

`NOT_LISTABLE`, `NAME_TAKEN`, `CASE_ALREADY_PICKED`, `RULE_OUT_OF_RANGE`.

---

## Endpoints specified elsewhere and not implemented

From the architecture and automation guides, left unimplemented on purpose:

- `POST /audiences` save, membership history, and `POST /audiences/{id}/export`
- Playbook CRUD, action send, holdout results
- Notification inbox and offer redemption
- Ad-spend import, CAC, ROAS
- Per-customer event log at `/customers/{id}/event-log` (the existing customer page is a separate, older view and was not changed)

A live backend should implement the read endpoints above against the Postgres read models, and should keep returning `connectedToBigQuery: false` until a job actually fills those models from BigQuery. The flag must not be set true for a hand-maintained extract.
