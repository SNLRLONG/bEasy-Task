# Customer Journey — Backend API spec

> **CS Overview (31 Oct 2026):** the CS Overview **report** is now being built on branch `feature/cs-kpi` (task `Tasks/2-todo/Portal-cs-overview.md`). Its live contract is the backend handoff `backend-response/CS_Overview_Frontend_Handoff_29Oct2026.md` (`/api/admin/cs-overview*`, **not** `/cs/overview`). The CS inbox, pick-up and contact-log screens stay **parked**: `/cs/*` below is still 501.

## Backend status (from backend review, re-issued 12 Oct 2026)

Backend status follows `Journey_V2_Portal_Review.md` (14 Oct 2026, includes the guest/customer event-ownership rules and mobile identity contract). Dev (`api-dev.beasy.info`) serves booking, registration, acquisition, customer page and activity. §9 lists the portal-side checks.

- **Base:** `{DEV_API}/api/admin`, Bearer token, success `{ "data": … }`, errors `{ "error": { "code", "message", "field" } }`. A missing or expired token (401) still uses the old shape `{ "success": false, "message": "…" }`.
- **Not implemented → `501 NOT_IMPLEMENTED`:** the portal shows "Not available yet", never an error page.

| Area | Endpoint | Status |
|---|---|---|
| Booking funnel + people | `GET /funnels/booking`, `…/stages/{stage}/people` | ✅ built (guest rows `kind: "device"`) |
| Registration funnel + people | `GET /funnels/registration`, `…/stages/{stage}/people` | ✅ built |
| **Acquisition funnel + people + after-purchase orders** | `GET /funnels/acquisition`, `…/stages/{stage}/people`, `…/after-purchase/{key}/orders` | ✅ built, ad data `null` until Meta (§3 of the review, §8 here) |
| Export | `POST …/people/export` | ✅ booking, registration, acquisition |
| Filter options | `GET /meta/filter-options` | ✅ built, portal wired 13 Oct (§9) |
| Abandon rules | `GET /abandon-rules` | ✅ built. `PUT` ⛔ 501 (view only) |
| Send to CS, log contact | `POST /cs/cases`, `POST /cs/contacts` | ⛔ 501 this release (portal buttons off) |
| CS inbox, pick-up, overview | `/cs/*` | ⛔ 501, **parked in portal** |
| Customer page | `/customers/{id}/summary journey-canvas sessions activity orders contacts` | ✅ built (`/contacts` returns `[]`; `/activity` replaces `/event-log`) |
| Integrations | `GET /integrations` | ✅ built (Meta and TikTok both `connected: false`) |
| Audiences | `/audiences…` | ⛔ 501, UI hidden |

## Shared filters (query of funnel and list endpoints)

| Param | Values | Default |
|---|---|---|
| `period` | `today` `7d` `30d` `60d` `90d` `custom` | `30d` |
| `from`, `to` | `YYYY-MM-DD`, only with `custom` | |
| `customerType` | `all` `registered_no_order` `existing` `guest` `saved_segment` | `all` |
| `segmentId` | int, with `saved_segment` | |
| `serviceIds` | comma list of service ids | all |
| `sources` | comma list of campaign keys, e.g. `meta:Instant_15,other:Organic` | all |
| `device` | `all` `android` `ios` `web` | `all` |
| `countBy` | `customer` `journey` (booking only) | `customer` |
| `unit` | `people` `devices` (registration only) | `people` |
| `channel`, `campaignId`, `adsetId` | acquisition only | |
| `customerKind` | `all` `new` `existing` (acquisition only) | `new` (backend, per the 19 Oct handoff; unverified) |
| `sourceType` | `all` `attributed` `organic` `unknown` (acquisition only). `unknown` is not organic | `all` |

**Period:** includes today. `from` is inclusive, `to` is **exclusive**: `30d` on 9 Oct 2026 = `from 2026-09-10`, `to 2026-10-10`. The responses echo `period.from` / `period.to` in this form. **To confirm with backend:** for `custom`, whether `to` is also exclusive (portal would then send the day after the last picked day).

Source keys: `meta:<campaign>`, `tiktok:<campaign>`, `other:Organic|Push campaign|Banner|Direct`.

---

## 1. Funnels

### `GET /funnels/{funnel}` — funnel numbers
`funnel` = `booking` | `registration` | `acquisition`. Query: shared filters. Perm: VIEW.
**Status:** `booking`, `registration` and `acquisition` live (acquisition detail: §8).

```json
{
  "funnel": "booking",
  "unit": "customers",
  "period": { "from": "2026-09-09", "to": "2026-10-09" },
  "dataAsOf": "2026-10-08T09:20:00+07:00",
  "kpis": [
    { "key": "viewed", "value": 1000, "format": "count", "previous": 920 },
    { "key": "recoverable", "value": 100, "format": "count", "previous": 90, "openValue": 1450 },
    { "key": "biggest_drop", "value": 0.58, "format": "ratio", "previous": 0.55, "stage": "service_viewed" }
  ],
  "stages": [
    { "code": "service_viewed", "reached": 1000, "dropped": 580, "continued": 0.42,
      "pctOfFirst": 1, "listable": true, "abandoned": 348, "notAbandonedYet": 232,
      "previous": { "reached": 926, "dropped": 530 } },
    { "code": "purchased", "reached": 110, "dropped": null, "continued": null,
      "pctOfFirst": 0.11, "listable": true, "previous": { "reached": 98, "dropped": null } }
  ],
  "insight": { "stage": "service_viewed", "dropped": 580, "total": 1000, "pct": 0.58 },
  "customerTypeCounts": { "all": 1000, "registered_no_order": 570, "existing": 310, "guest": 120, "saved_segment": 0 },
  "filterCounts": {
    "services": { "1": 420, "2": 260 },
    "sources": { "meta:Instant_15": 380, "other:Organic": 260 },
    "devices": { "all": 1000, "android": 560, "ios": 380, "web": 60 }
  }
}
```

Rules:
- `dropRate` = dropped / reached at the stage (stage-relative) and `dropPctOfFirst` = dropped / first measurable stage (whole funnel), both 0 to 1, per the 19 Oct handoff. The portal shows `dropRate` on the Dropped tile and in the table, `dropPctOfFirst` in the table and the tile tooltip, and computes them the same way if a field is missing.
- `reached` is cumulative (furthest step ≥ stage). `dropped = reached − next.reached`. `continued = next.reached / reached`. `pctOfFirst = reached / first.reached`.
- `previous` = the same period length immediately before `period`. Always sent.
- `abandoned` / `notAbandonedYet` (booking only) split `dropped` using the abandon rules (§2). They add up to `dropped`.
- `filterCounts`: each menu's counts follow all **other** filters, not its own.
- `listable: false` only for acquisition `ad_clicked` (Meta totals, no people).
- `kpis[].key` per funnel: booking `viewed, purchased, recoverable, biggest_drop` · registration `page_viewed, otp_requested, registered, otp_not_verified` (was `otp_failed`; the portal accepts both until the rename is on dev) · acquisition `ad_clicks, app_opens, first_purchases, click_to_buy`. `format`: `count` | `ratio` | `usd`. `recoverable` = abandoned customers at details/slot/checkout who are reachable; `openValue` = their booking value (USD).
- Stage codes — booking: `service_viewed booking_started details_completed slot_selected checkout_started purchased` · registration: `registration_viewed otp_requested otp_verified signup_started registration_completed` · acquisition: `ad_clicked app_opened registered service_viewed booking_started details_completed slot_selected checkout_started purchased`.
- Unit: `customers` / `journeys` (booking), `people` / `devices` (registration), `devices` (acquisition; before sign-up a person is a device). Guest events must be kept per device (see backend request 3.1).

Extra fields:

| Funnel | Field | Shape |
|---|---|---|
| registration | `otpReasons` | `[{ "reason": "sms_not_delivered", "count": 180, "pct": 0.36 }]`. Reasons: `sms_not_delivered gateway_error wrong_code blocked expired_or_left`. Counts add up to `otp_requested.dropped` |
| acquisition | `afterPurchase` | `[{ "key": "rescheduled", "orders": 31, "ofPurchases": 0.17, "amount": 0, "detail": { "done": 26, "upcoming": 5, "paidFee": 4 } }]`. Keys: `payment_failed_then_paid auto_promo rescheduled cancelled refunded service_completed`. From order records |
| acquisition | `campaigns` | `[{ "campaignId": "meta:Instant_15", "name": "Instant_15", "adClicks": 10000, "appOpens": 5200, "registered": 2100, "purchased": 180, "clickToBuy": 0.018, "active": true }]` (`active` = the one in the filter) |

---

### `GET /funnels/{funnel}/stages/{stage}/people` — people behind a number
Perm: VIEW. **Status:** live for all three funnels. Query: shared filters +

| Param | Notes |
|---|---|
| `state` | `reached` \| `dropped` (required) |
| `reason` | registration `otp_requested` dropped only: one OTP reason |
| `abandoned` | `all` \| `yes` \| `no`. Works for `reached` and `dropped` |
| `converted` | booking only: `true` = purchased in this journey, `false` = not; omitted = all. Works for `reached` and `dropped` |
| `purchased` | registration only: `yes` \| `no`, whether the user has any paid order. Response adds `purchaseBreakdown { purchased, noOrderYet }` (before this filter) and per row `everPurchased`, `firstPurchasedAt`. Portal sends it; not on dev yet |
| `q` | search name or phone, inside this list |
| `cursor`, `limit` | paging |

Errors: `422 NOT_LISTABLE` for acquisition `ad_clicked`.

```json
{
  "count": 50,
  "unit": "customers",
  "abandonStrip": { "abandoned": 46, "notYet": 4, "ruleMinutes": 30 },
  "reach": { "push": 31, "cs": 12, "adsOnly": 7, "suppressed": 3 },
  "breakdown": [ { "key": "moved_on", "count": 110, "pct": 0.69 } ],
  "convertedBreakdown": { "converted": 28, "notConverted": 134, "everPurchased": 9 },
  "items": [
    {
      "id": "4821", "kind": "customer", "name": "Sokha Chan",
      "phoneMasked": "(+855) 12 ••• 501", "deviceLabel": "Android · app 3.4.1",
      "registered": true, "furthest": "checkout_started", "service": "Instant Cleaning",
      "source": "Meta · Instant_15", "interest": "very_high",
      "reach": "push", "suppressed": false, "value": 15,
      "lastActivityAt": "2026-10-08T09:42:00+07:00", "idleMinutes": 35,
      "abandonState": "abandoned", "converted": false, "everPurchased": true
    },
    {
      "id": "dev_a91f3c2", "kind": "device", "phoneMasked": null,
      "deviceLabel": "Android · app 3.4.1", "deviceId": "a91f…3c2", "registered": false,
      "furthest": "app_opened", "reach": "ads_only", "suppressed": false,
      "lastActivityAt": "2026-10-08T09:40:00+07:00", "idleMinutes": 20,
      "abandonState": "none", "sessions": 3,
      "otp": { "result": "locked", "gatewayCode": "503", "lockedMinutes": 30, "resends": 1 }
    }
  ],
  "nextCursor": null
}
```

Rules:
- `count`, `reach`, `abandonStrip` describe the **whole** filtered list, not the page. `reach.push + cs + adsOnly = count`. `suppressed` is a tag over the same people (they are in a tile and also counted here).
- `reach`: `push` = registered, valid token, marketing allowed, not suppressed · `cs` = has phone, not push-reachable, not suppressed · `ads_only` = guests/devices.
- `suppressed` = asked for no marketing / "no more calls". Kept in the list, excluded from push, CS and audiences.
- `abandonState`: `abandoned` (idle ≥ the rule time of its furthest step) | `not_yet` | `none`. `idleMinutes` = now − last activity.
- `breakdown`: reached list → `moved_on`, `stopped_here`; registration OTP dropped list → one row per reason.
- `converted` = purchased in this journey (paid order); `everPurchased` = any earlier paid order and never hides a new abandoned journey. `convertedBreakdown` is booking only.
- `interest`: `low medium high very_high converted`. `otp.result`: `not_delivered gateway_error wrong_code locked resent expired`.
- Sort: last activity, newest first.
- **Row key:** with `countBy=journey` the same customer can appear more than once, so rows also carry `journeyId`; use it as the row key (`id` repeats).
- **Registration list:** rows can be `kind: "device"` (`id: "dev_…"`, `deviceId`, `otp`, `phoneMasked` from the OTP request, `name: null`). `abandonState` is always `none`.

### `POST /funnels/{funnel}/stages/{stage}/people/export` — export a list
Perm: EXPORT. Log in the activity log. **Status:** live for all three funnels.

```json
{
  "query": { "...same filters + state/reason/abandoned/q, no cursor/limit": "" },
  "selection": { "mode": "all", "ids": [], "excludeIds": ["4822"] },
  "columns": ["name_phone", "customer_id", "service", "furthest_step", "open_value", "source", "interest", "reachability", "session_ids"]
}
```
`selection.mode`: `all` (the whole list minus `excludeIds`) or `ids` (only `ids`). Same object is used by every list action below.

Response: `{ "rows": 50, "guests": 7, "suppressed": 3, "url": "https://…/export.csv" }`. Guests export a device id instead of a phone; suppressed rows are included and marked "do not contact". The `url` is a **private link valid for 1 hour**: open it immediately, never store it. The CSV has **real phone numbers** (for calling).

---

## 2. Abandon rules

| Rule | Meaning | Default |
|---|---|---|
| `service_viewed` | viewed, no booking started | 1440 min |
| `booking_started` | no details completed | 60 |
| `details_completed` | no slot picked | 60 |
| `slot_selected` | no checkout | 60 |
| `checkout_started` | no payment | 30 |
| `payment_failed` | no successful retry | 30 |

A journey is **Abandoned** when its furthest step has had no next step for ≥ that many minutes. A purchase clears it immediately. Changing a rule applies from then on; funnel counts do not change.

### `GET /abandon-rules` — Perm: VIEW
```json
{
  "rules": [ { "key": "checkout_started", "minutes": 30 } ],
  "updatedAt": "2026-10-07T16:00:00+07:00",
  "updatedBy": "Admin Dara",
  "history": [ { "changedAt": "2026-10-07T16:00:00+07:00", "changedBy": "Admin Dara", "key": "checkout_started", "from": 60, "to": 30 } ]
}
```

### `PUT /abandon-rules` — Perm: UPDATE (admin). Audit-log it. ⛔ 501 (view only until backend ships it)
`payment_failed` is listed but unused (no payment-failed events stored yet).
Body `{ "rules": [ { "key": "checkout_started", "minutes": 45 } ] }` (all six). `minutes` 5–10080 else `422 RULE_OUT_OF_RANGE` (with `field`). Response = same as GET.

---

## 3. Send to CS

Takes `ListAction` = `{ "query": {…list filters incl. funnel, stage, state…}, "selection": { "mode", "ids", "excludeIds" } }`.

### `POST /cs/cases` — send people to the CS inbox. Perm: ADD (CS-OVERVIEW). ⛔ 501 this release
```json
{ "query": {}, "selection": { "mode": "ids", "ids": ["4821"] },
  "reason": "sales_follow_up", "dueAt": "2026-10-08T18:00:00+07:00", "priority": "normal", "note": "" }
```
`reason`: `sales_follow_up` | `booking_help` (only sales follow-ups count in CS results). `priority`: `low normal high`. **No assignee**: the first agent to pick a case up owns it. Suppressed people are left out. `query` **must include `funnel`, `stage` and `state`** together with the list filters. Response: `{ "created": 9, "leftOutSuppressed": 3, "alreadyOpen": 2 }` (`alreadyOpen` = skipped because they already have an open case; show all three numbers). OTP-failed devices (phone saved at OTP request) are allowed.

---

## 4. Log a CS contact

### `POST /cs/contacts` — log a CS contact. Perm: UPDATE (CS-OVERVIEW). ⛔ 501 this release
```json
{ "customerId": 4821, "channel": "phone", "result": "spoke", "said": "will_book_later",
  "problem": "none", "purpose": "sales_follow_up",
  "nextFollowUp": "2026-10-09T10:00:00+07:00", "note": "" }
```

| Field | Values |
|---|---|
| `channel` | `phone telegram in_app_chat` |
| `result` | `no_answer` (attempt) · `busy` (attempt) · `spoke` (reached) · `wrong_number` (flags the phone) |
| `said` (optional) | `will_book_later booked_now needs_price_info not_interested no_more_calls` |
| `problem` | `payment app_bug price_or_slot partner none` (counts in "Problems found") |
| `purpose` | `sales_follow_up booking_confirmation help` (confirmation calls are not sales contact) |

`said = no_more_calls` suppresses the customer: lists then show `suppressed: true` and new CS cases leave them out. Response: the saved contact `{ …input, "id": 12, "agent": "Sreymom Chea", "contactedAt": "…" }`. Revenue/orders never come from this form.

---

## 5. CS

> **Superseded for the report (31 Oct 2026).** The working CS Overview report is `GET /cs-overview`, `GET /cs-overview/people`, `POST /fileExport/cs-overview` and `GET/PUT /cs-overview/targets/:month`; see `backend-response/CS_Overview_Frontend_Handoff_29Oct2026.md`. Backend says the `/cs/*` routes below, including `GET /cs/overview`, are **501 placeholders**. The inbox, pick-up and contact-log screens remain parked in the portal.

### `GET /cs/cases` — inbox. Perm: VIEW (CS-OVERVIEW)
`customerId` can be `null` (registration drop-offs with only a phone). `[ { "id": 1, "customerName": "Sokha Chan", "phoneMasked": "(+855) 12 ••• 501", "reason": "sales_follow_up", "sourceList": "booking:checkout_started:dropped", "dueAt": "…", "priority": "normal", "owner": null, "createdAt": "…" } ]`

### `POST /cs/cases/{id}/pick-up` — take a case. Perm: UPDATE (CS-OVERVIEW)
No body. The first caller becomes `owner`; later callers get `409 CASE_ALREADY_PICKED`. Response: the updated case (shape of `GET /cs/cases` item, with `owner` set). Not used by the UI yet. A close endpoint is not specified yet.

### `GET /cs/overview` — Perm: VIEW (CS-OVERVIEW)
Backend: `other` first sales = walk-in / staff-entered (direct sale) orders; `firstSales.target` = `0.30`. Whether that matches "Telegram, walk-in, partner referral" and who may change `target` are open (review Part B 6 and 10).
Month = current month; first/repeat sales use a **30-day window** per customer.
```json
{
  "month": "2026-09",
  "provisional": { "customers": 214, "since": "2026-09-07" },
  "waitingInInbox": 9,
  "today": {
    "agents": [ { "agentId": 1, "name": "Sreymom Chea",
      "pickedUp": { "total": 52, "registered": 30, "existing": 22 },
      "notContacted": 4, "dueToday": 5, "overdue": 1, "onTime": 0.88, "attempts": 88, "reached": 61 } ],
    "total": { "agentId": 0, "name": "Team", "pickedUp": { "total": 184, "registered": 112, "existing": 72 },
      "notContacted": 23, "dueToday": 17, "overdue": 6, "onTime": 0.89, "attempts": 223, "reached": 150 }
  },
  "problems": { "total": 18, "payment": 7, "appBug": 5, "priceOrSlot": 4, "partner": 2 },
  "firstSales": {
    "contacted":  { "customers": 420, "withFirstSale": 126, "conversion": 0.30, "revenue": 3780 },
    "noContact":  { "customers": 1100, "withFirstSale": 187, "conversion": 0.17, "revenue": 5610 },
    "byAgent": [ { "agentId": 1, "name": "Sreymom Chea", "contacted": 140, "firstSale": 46, "conversion": 0.33, "revenue": 1380 } ],
    "total": { "agentId": 0, "name": "Team", "contacted": 420, "firstSale": 126, "conversion": 0.30, "revenue": 3780 },
    "doubleContacted": 10,
    "target": 0.30,
    "other": { "sales": 24, "revenue": 610 }
  },
  "repeat": {
    "byAgent": [ { "agentId": 1, "name": "Sreymom Chea", "followedUp": 60, "orderedAgain": 21, "conversion": 0.35, "repeatBookings": 27, "repeatRevenue": 810 } ],
    "total": { "agentId": 0, "name": "Team", "followedUp": 148, "orderedAgain": 46, "conversion": 0.31, "repeatBookings": 58, "repeatRevenue": 1740 }
  }
}
```
Rules: `onTime`/`conversion` are `null` when there is nothing to divide by or outcomes were not recorded (UI shows N/A / Unavailable, never 0). Team totals count each customer once (rows can add up to more). Revenue = completed service value after discounts and refunds, excl. tax, credited to the **last sales follow-up before the order**. "Contacted" = a sales-follow-up contact before the first purchase, same registration group and window as "no contact". `other` = first sales from Telegram, walk-in, partner referral. `target` is configurable.

---

## 6. Customer page

All: Perm VIEW (MARKETING-CUSTOMER or MARKETING-JOURNEY). `{id}` = customer id.

### `GET /customers/{id}/summary`
```json
{
  "id": 4821, "name": "Sokha Chan", "phoneMasked": "(+855) 12 ••• 501",
  "customerSince": "2026-05-04", "cameFrom": "Meta · Instant_15", "device": "Android 3.4.1",
  "district": "Toul Kork, Phnom Penh", "registered": true,
  "status": "abandoned", "interest": "very_high", "interestChange": "up",
  "furthest": "checkout_started", "openIntentValue": 15, "lifetimeSpend": 102,
  "pastOrders": 2, "pushOn": true,
  "atAGlance": [ { "service": "Instant Cleaning", "state": "abandoned", "stepIndex": 5, "at": "2026-10-08T09:42:00+07:00" } ],
  "numbers": { "sessions30d": 7, "lastActiveAt": "…", "pushThisWeek": { "used": 1, "limit": 3 }, "bestChannel": "push" },
  "recent": [ { "title": "Started checkout", "subtitle": "Instant Cleaning · $15", "at": "…" } ]
}
```
`status`: `active abandoned not_yet converted cancelled completed`. `stepIndex` 1–6 (booking funnel position). `bestChannel`: `push cs ads_only`. `interestChange`: `up` `down` `same`.
`interest` (backend rule): `converted` paid · `very_high` checkout in the last 3 days · `high` checkout older, or details/slot in the last 3 days · `medium` details/slot older, or booking started in the last 3 days · `low` the rest.

### `GET /customers/{id}/journey-canvas`
```json
{ "source": { "label": "Meta · Instant_15", "campaign": "Instant_15", "firstSeenAt": "2026-05-04T10:02:00+07:00" },
  "services": [ { "serviceId": 1, "name": "Instant Cleaning", "furthest": "checkout_started", "status": "abandoned",
    "idleMinutes": 35, "value": 15, "sessions": 3,
    "afterPurchase": { "kind": "completed", "at": "…", "feeKept": 0 } } ],
  "totals": { "services": 4, "sessions": 7, "days": 30 } }
```
`afterPurchase` (optional): `kind` `completed` | `cancelled`.

### `GET /customers/{id}/sessions?serviceId=1`
`[ { "id": "ses_31c07e", "serviceId": 1, "startedAt": "…", "endedAt": "…", "entry": "Meta · Instant_15", "device": "Android app 3.4.1", "events": 5, "furthest": "checkout_started", "status": "abandoned", "recordingUrl": "https://clarity.microsoft.com/…" } ]` — newest first. `recordingUrl` `null` when none (Clarity, by session id).

### `GET /customers/{id}/activity` (was `/event-log`, which the backend removes once the portal confirms)
Query (all optional): `serviceId`, `event`, `status`, `source`, `from`, `to`.
```json
{
  "groups": [
    { "kind": "session", "id": "ses_31c07e", "day": "2026-10-08", "title": "Instant Cleaning",
      "startedAt": "…", "endedAt": "…", "entry": "Meta · Instant_15", "furthest": "checkout_started",
      "status": "abandoned", "recordingUrl": null,
      "events": [
        { "id": "evt_8f2a41c9", "at": "…", "event": "checkout_started", "details": "Booking value $15",
          "serviceName": "Instant Cleaning", "by": "app", "source": "Meta · Instant_15",
          "ids": { "eventId": "evt_8f2a41c9", "sessionId": "ses_31c07e", "device": "Android app 3.4.1", "campaignId": "meta_238841", "deepLink": "beasy://service/instant" } }
      ] }
  ],
  "summary": { "events": 18, "sessions": 5, "services": 4, "orders": 2 }
}
```
- Group `kind`: `session` (a service visit) · `order` (an order's lifecycle; add `orderId`) · `guest` (activity before sign-up, merged at registration).
- `by`: `app backend system meta team`. Order/payment events (`payment_failed`, `purchase_completed`, `order_rescheduled`, `order_cancelled`, `order_refunded`, `service_completed`, `promo_applied` with auto/manual in `details`) come from **backend order records**, never from app events.
- `event` codes used by the UI: `ad_click first_open guest_first_open registration_merged app_opened banner_clicked search_performed search_result_clicked service_viewed booking_started booking_details_completed slot_selected checkout_started promo_applied payment_failed purchase_completed reschedule_started order_rescheduled cancellation_started cancellation_fee_shown order_cancelled order_refunded service_completed`.
- Within a group: newest first. Groups: newest first.

### `GET /customers/{id}/orders`
`[ { "id": 5193, "service": "Upholstery", "status": "completed", "orderStatus": "COMPLETED", "paymentStatus": "PAID", "total": 51, "placedAt": "…", "slotAt": "…", "paymentMethod": "ABA KHQR" } ]` — `status`: `upcoming completed cancelled refunded` (journey summary, kept for compatibility). `orderStatus` is the raw Order management status (`PENDING ACCEPTED IN_PROGRESS COMPLETED CANCELLED`) and `paymentStatus` the raw payment status (backend response 20 Oct). The portal shows `orderStatus` and `paymentStatus`, and falls back to `status`. `total` **includes VAT** (CS revenue excludes VAT).

### `GET /customers/{id}/contacts`
`[ { "id": 12, "customerId": 4821, "channel": "phone", "result": "spoke", "said": "will_book_later", "problem": "none", "purpose": "sales_follow_up", "nextFollowUp": "…", "note": "", "agent": "Sreymom Chea", "contactedAt": "…" } ]` — newest first.

---

## 7. Audiences (UI hidden for now behind a flag; all endpoints in this section ⛔ 501)

### `GET /audiences` ⛔ 501
`[ { "id": 1, "name": "Checkout dropped · Sep 8 – Oct 7", "builtFrom": "booking:checkout_started:dropped", "people": 50, "push": 31, "ads": 47, "updatedAt": "…", "status": "synced", "destinations": ["push", "meta"] } ]`. `status`: `synced syncing failed`. `push`/`ads` `null` when not applicable. `builtFrom` = `<funnel>:<stage>:<state>` or `saved_segment`.

### `POST /audiences/estimate` ⛔ 501
Body `{ "rules": [ { "id": "r1", "field": "journey_step", "operator": "is", "value": "checkout_started", "group": "and", "locked": true } ] }` →
`{ "estimated": 47, "push": 31, "ads": 47, "csOnly": 9, "leftOutSuppressed": 3, "computedAt": "…" }`

Rule fields: `journey_step behaviour service source customer_type saved_segment order_count last_order date frequency`. Operators: `is is_not has has_not within before after exactly at_least at_most`. Groups: `and` (all must match), `or` (one OR group), `exclude`. One person counted once.

### `POST /audiences` ⛔ 501
Body `{ "name": "…", "builtFrom": "booking:checkout_started:dropped", "rules": [ …as above… ], "destinations": ["push", "meta", "csv"] }` → the created audience (shape of `GET /audiences` item). `409 NAME_TAKEN` if the name exists. Saving never sends anything. Meta sync runs daily (matched by phone or device id, hashed).

### `POST /audiences/{id}/retry-sync` ⛔ 501 → the audience (status `syncing`).
### `GET /integrations` → `{ "meta": { "connected": true }, "tiktok": { "connected": false } }`
Live. Today both are `connected: false`. The acquisition funnel (§8) reads `meta.connected` to decide whether ad clicks exist.

---

## 8. Acquisition funnel

The full contract (query, response, rules, errors, people list, after-purchase orders, filter options) is in [Journey_Acquisition_API_Request.md](../backend-request/Journey_Acquisition_API_Request.md). Backend built it (14 Oct review). Portal-specific notes:

- Without Meta: `ad_clicked` has `reached: null` with `unavailableReason` (`meta_not_connected`, or `filter_not_applicable` when `serviceIds`, `device` or `customerKind` is set); `baseStage` is `app_opened`. The portal shows "—", never 0.
- `previous` is always sent. `afterPurchase.rescheduled` and `cancelled` fee fields are `null` (`not_tracked`) until backend stores them.
- Backend response 20 Oct 2026 (not yet on dev): `unit` is `people` and `identityUnit` is `customer_or_device`; only `unit=people` is accepted (the portal forces it for acquisition). `purchaseCounts` is `{ firstTimeBuyers, buyers, orders, purchaseEvents }` and the four need not match. Each `afterPurchase` row has `denominator: { unit: "orders", count }` and `ofPurchases` (shown as "5 of 11 orders (45%)"). Unknown source is `other:Unknown` (never Organic), listed in `filter-options` and `filterCounts.sources`; `sourceType` and `sources` combine with AND.
- People lists carry `summaryScopes`: booking `abandonStrip` is counted before the `abandoned` and `converted` filters; `convertedBreakdown` after `abandoned`, before `converted`; registration `purchaseBreakdown` before `purchased`; `count`, `reach`, `breakdown` after all list filters. Registration `q` matches name and phone (including OTP-only phones; `012 345 678`, `012345678`, `+855 12 345 678`, `85512345678` are one number); every registration row has the `name` key, `null` when unknown.
- Campaign key is `meta:<campaign>` everywhere (filters, `campaigns[].campaignId`, filter-options `key`).
- Open items for backend: `backend-request/pending-request-backend.md` §4 and `backend-request/backlog.md` (Meta import).

---


## Excluded from this spec (for now)
- Push notifications: bulk recovery push (preview and send) and single-customer push (checks and send). The portal UI exists and runs on mock data; the endpoints will be specified in a later version.
- AI summary: `GET /customers/{id}/ai-summary` and its refresh.
- Related event rows (`recovery_push_sent`, `push_opened`) are therefore not expected in the event log yet.

## Empty for now (backend, 14 Oct 2026) — not bugs; show "—" / "Not available", never 0

| Field | Value today |
|---|---|
| `summary.numbers.pushThisWeek.used`, `recordingUrl`, `afterPurchase.feeKept` | `null` |
| `rescheduled`, `cancelled` fee fields | `null` (`not_tracked`) |
| `sources`, `cameFrom`, `entry` | mostly `"Organic"` (see `backend-request/Campaign_Source_Request.md`) |
| Registration `otpReasons` | `expired` and `not_verified_unknown` always; `gateway_error`, `wrong_code`, `rate_limited`, `blocked`, `expired_or_left` when `otpReasonAvailability` marks them `true` (backend, 30 Oct: `backend-response/Journey_OTP_Reasons_Backend_Response_30Oct2026.md`). `delivery` is `false`: "Not tracked yet", never 0. Flags can be omitted when there are no registration flows; the portal treats an omitted flag with a zero count as not tracked |
| People rows `lastActivityAt`, `idleMinutes` | can be `null` (invalid time): the portal shows "Time unavailable" |
| `/customers/{id}/contacts` | `[]` |

Backend stage mapping (booking): add-ons, options, address, date, time and pair service → `booking_started`; summary and promo → `slot_selected`; payment started, order created and confirmation → `checkout_started`. Registration: `registration_viewed` = phone entered; `signup_started` = the app reported the sign-up form; logins of existing customers are excluded.

## Not wired yet
- ~~Service, source and device option lists → `GET /meta/filter-options`~~ wired 13 Oct (services, campaigns, ad sets; the device list is still a constant, see §9 item 6).
- Saved CRM segments for "Saved segment…" → existing `customer-segment` endpoint.
- Coupon list in the push dialogs → existing coupon list endpoint.
- Setup › Integrations page (connect Meta / TikTok).

## Backend work these endpoints depend on
See `backend-request/pending-request-backend.md` and `backend-request/backlog.md`.

---

## 9. Contract check, portal ↔ backend (13 Oct 2026)

Portal state: all items below marked **portal done** are built and unit/type checked, **not yet run against a live backend** (not on dev). Items marked **backend** need an answer or a change before the portal can finish. Nothing here changes an endpoint the backend already built unless stated.

### 9.1 What the portal now does

| Item | Portal behaviour |
|---|---|
| Activity | Calls `/customers/{id}/activity`. **Backend may remove `/event-log`.** |
| Period cap | Custom picker blocks > 92 days (inclusive days, `to` exclusive, so the same count as `to − from`). A `422 PERIOD_TOO_LONG` still shows "Period too long, show last 30 days". Presets stop at `90d`, so they never hit the cap |
| No Meta | When `baseStage = app_opened`, KPI cards `ad_clicks`, `click_to_buy` and the Ad clicks / Click to buy columns of the campaign table are hidden. The `ad_clicked` stage stays, greyed, tooltip "Meta not connected" |
| After purchase orders | Number opens a sheet: `GET …/after-purchase/{key}/orders` with the page filters, `limit=50`, `cursor`. `count: null` (`rescheduled`) shows "not tracked yet" |
| Filter options | `GET /meta/filter-options` feeds the Service menu (falls back to categories if it fails), the Source menu (campaigns with zero people still listed) and the Acquisition Channel / Campaign / Ad set selects. The mock campaigns were deleted |
| Channel values | Portal sends `channel=meta|tiktok|other` only (was `google`, `organic`, which the backend answers `422 INVALID_FILTER`) |
| Campaign key | `campaignId` = `campaigns[].campaignId` = filter-options `campaigns[].key`, e.g. `meta:Instant_15` |

### 9.2 Needs the backend (3 confirmations; rechecked 14 Oct, the first draft had 10)

| # | Topic | Detail | Ask |
|---|---|---|---|
| 1 | **`GET /meta/filter-options` and `channel`** | Portal types follow `backend-request/Journey_Acquisition_API_Request.md` §4: `services[{id,name}]`, `campaigns[{key,name,channel,adsets[{id,name}]}]`, `devices[]`. The review says it "works" but shows no response, and lists no valid `channel` values (only that an unknown one is `422`) | Send one real response. Confirm `channel` is `all`, `meta`, `tiktok` or `other` (default `all`), as in §8.2 |
| 2 | **Campaign and ad set id** | Filters, `campaigns[].campaignId` and filter-options `key` must be the same string (`meta:Instant_15`). §8.7 q3 is still open, and the review does not show real ids | Confirm `meta:<name>` for all four, and the ad set id format (string, e.g. `"238841"`) |
| 3 | **`unavailableReason` on `ad_clicked`** | Spec §8.2 sends `filter_not_applicable` when `serviceIds`, `device` or `customerKind` are set; the review lists only `meta_not_connected` and `not_tracked` | FYI only: portal now accepts all three, no change needed if you send it |

### 9.3 Decisions (14 Oct 2026, product owner)

| Review § | Question | Decision |
|---|---|---|
| 6.5 | `device=web` is always 0 | **Remove Web.** Portal Device menu is now All, Android, iOS. Backend: drop `web` from `filter-options.devices` and `filterCounts.devices`, and treat `device=web` as `422 INVALID_FILTER` |
| 6.6 | Role permissions | **Out of scope for this branch** (`feat/journey-cms`). Backend keeps admin-login-only for now; revisit when roles are planned. Portal still hides screens and export behind `MARKETING-JOURNEY` |
| 6.9 | `refunded.amount` | **Backend's choice.** Send `null` when the refund amount is not stored, otherwise the value as a number or a numeric string; the portal shows whatever is there and "—" for `null`. No change needed on the portal when refund amounts are stored later |
| 6.7 | 92 days | OK. Presets stop at 90 days |

The other review questions (6.1 to 6.4) are confirmed as in 9.4.

### 9.4 Portal replies to the review's questions (proposed, to confirm)

| Review §6 | Proposed reply |
|---|---|
| 1 Cohort = first install, 7-day window | ✅ matches §8.2 |
| 2 Two phones count once | ✅ |
| 3 `first_purchases` = first-ever paid order inside 7 days | ✅ |
| 4 `campaigns[]` follows service / device / customerKind | ✅ (only `campaignId`, `adsetId` ignored) |
| 5 `device=web` | Remove web (9.3) |
| 6 Role permissions | Out of scope for this branch (9.3) |
| 7 92 days | ✅ |
| 9 `refunded.amount` | `null` or number/numeric string, portal displays it (9.3) |
| 8 `/meta/` and ad blockers | Cannot test until dev is live. Portal will report after the first dev test |

### 9.5 Portal-side, not for the backend

- Order `status` in the after-purchase sheet reuses the customer Orders tab labels (`upcoming`, `completed`, `cancelled`, `refunded`); an unknown value prints raw.
- Guest rows no longer break if `deviceLabel` is missing (the review's guest example does not list it).
- 92-day edge: the picker allows 92 inclusive days (`to − from = 92`, `to` exclusive). If the backend counts the other way, a 92-day pick returns `PERIOD_TOO_LONG` and the portal shows the "too long" card, so nothing breaks.
- Retest on dev (customers 3370, 3547, 3555, 3526) when live: checklist in review §7.
- Khmer text for the new keys needs a native review. `/setup/integrations` has no route.
