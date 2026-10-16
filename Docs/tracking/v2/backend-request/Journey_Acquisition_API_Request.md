# Acquisition funnel — API request

Base `/api/admin`. Auth, envelope and errors as in the other funnel endpoints. Perm: VIEW.

## 1. `GET /funnels/acquisition`

| Param | Values | Default |
|---|---|---|
| `period` | `today` `7d` `30d` `60d` `90d` `custom` | `30d` |
| `from`, `to` | `YYYY-MM-DD`, only with `custom` | |
| `channel` | `all` `meta` `tiktok` `other` | `all` |
| `campaignId` | campaign key, e.g. `meta:Instant_15` | all |
| `adsetId` | ad set id (needs `campaignId`) | all |
| `customerKind` | `new` `existing` | both |
| `serviceIds` | comma list of service ids | all |
| `device` | `all` `android` `ios` `web` | `all` |

`new` = no succeeded order before the first app open. `existing` = had one.

### Response

```json
{
  "funnel": "acquisition",
  "unit": "devices",
  "period": { "from": "2026-09-09", "to": "2026-10-09" },
  "dataAsOf": "2026-10-08T09:20:00+07:00",
  "baseStage": "ad_clicked",
  "kpis": [
    { "key": "ad_clicks", "value": 10000, "format": "count", "previous": 9100 },
    { "key": "app_opens", "value": 5200, "format": "count", "previous": 4800 },
    { "key": "first_purchases", "value": 180, "format": "count", "previous": 150 },
    { "key": "click_to_buy", "value": 0.018, "format": "ratio", "previous": 0.0165 }
  ],
  "stages": [
    { "code": "ad_clicked", "reached": 10000, "dropped": null, "continued": null,
      "pctOfFirst": 1, "listable": false, "previous": { "reached": 9100, "dropped": null } },
    { "code": "app_opened", "reached": 5200, "dropped": 3100, "continued": 0.40,
      "pctOfFirst": 0.52, "listable": true, "previous": { "reached": 4800, "dropped": 2900 } }
  ],
  "insight": { "stage": "app_opened", "dropped": 3100, "total": 5200, "pct": 0.60 },
  "customerKindCounts": { "all": 5200, "new": 4700, "existing": 500 },
  "filterCounts": {
    "channels": { "all": 5200, "meta": 3900, "tiktok": 0, "other": 1300 },
    "campaigns": { "meta:Instant_15": 2100, "other:Organic": 1100 },
    "adsets": { "238841": 340 },
    "services": { "1": 420, "2": 260 },
    "devices": { "all": 5200, "android": 3100, "ios": 2000, "web": 100 }
  },
  "afterPurchase": [
    { "key": "rescheduled", "orders": 31, "ofPurchases": 0.17, "amount": 20,
      "detail": { "done": 26, "upcoming": 5, "paidFee": 4 } }
  ],
  "campaigns": [
    { "campaignId": "meta:Instant_15", "name": "Instant_15", "adClicks": 10000, "appOpens": 5200,
      "registered": 2100, "purchased": 180, "clickToBuy": 0.018, "active": true }
  ]
}
```

(Example shows two stages and one after-purchase row; the real response has all nine and all six.)

### Rules

- **Stages, in order:** `ad_clicked app_opened registered service_viewed booking_started details_completed slot_selected checkout_started purchased`.
- **Who is counted:** once per person. A person is the customer if the device is linked to one, else the device. Before sign-up the unit is a device.
- **Cohort:** people whose first app open is in `period`. Each later stage counts them if reached within 7 days of that first open.
- **Cumulative:** `reached` = furthest step ≥ stage, so it never rises. `dropped = reached − next.reached`. `continued = next.reached / reached`. `pctOfFirst = reached / baseStage.reached`. The last stage has `dropped` and `continued` `null`.
- Stages after `registered` count registered people only.
- `previous` = the same period length right before `period`, same filters. Always sent.
- **`ad_clicked`:** Meta/TikTok total. `listable: false`, `dropped` and `continued` always `null`. If a number is not available, `reached` is `null` with `unavailableReason`: `meta_not_connected` or `filter_not_applicable` (`serviceIds`, `device` or `customerKind` is set). `click_to_buy` is then `null`.
- `baseStage` = `ad_clicked` when it has a number, else `app_opened`.
- `insight` = biggest `dropped` among stages that have one (not `ad_clicked`).
- **KPIs:** `first_purchases` = people whose first-ever succeeded order is in the window. `click_to_buy = purchased / ad_clicked`, `null` if clicks are `null` or 0. `format`: `count` or `ratio`.
- **`filterCounts`:** each menu follows all the other filters, not its own. `adsets` only when `campaignId` is set.
- **`afterPurchase`** comes from order records of the cohort's purchases. `ofPurchases = orders / purchase orders`. `amount` in USD.

| Key | Counts | `amount` | `detail` |
|---|---|---|---|
| `payment_failed_then_paid` | paid after a failed payment, retry within 30 min | order value | |
| `auto_promo` | auto-applied promo | discount given | `auto`, `manual` |
| `rescheduled` | rescheduled at least once | reschedule fees paid | `done`, `upcoming`, `paidFee` |
| `cancelled` | cancelled | fees kept | `withFee`, `free` |
| `refunded` | refunded | amount refunded | |
| `service_completed` | completed | order value | `upcoming` |

- **`campaigns[]`:** follows `period` and `channel`, ignores `campaignId` and `adsetId`. Sorted by `purchased` descending, max 50. `active` = the row matching `campaignId`. `clickToBuy` `null` if `adClicks` is `null` or 0.
- **Errors:** `422 INVALID_FILTER` for an unknown `campaignId`, or `adsetId` without `campaignId`.

---

## 2. `GET /funnels/acquisition/stages/{stage}/people`

Query: all filters above, plus:

| Param | Notes |
|---|---|
| `state` | `reached` \| `dropped` (required) |
| `q` | search name or phone, inside this list |
| `cursor`, `limit` | paging |

```json
{
  "count": 3100,
  "unit": "devices",
  "reach": { "push": 0, "cs": 0, "adsOnly": 3100, "suppressed": 0 },
  "breakdown": [ { "key": "moved_on", "count": 2100, "pct": 0.40 } ],
  "items": [
    { "id": "dev_a91f3c2", "kind": "device", "deviceId": "a91f…3c2", "deviceLabel": "Android · app 3.4.1",
      "name": null, "phoneMasked": null, "registered": false, "furthest": "app_opened",
      "source": "Meta · Instant_15", "adset": "Phnom Penh 25-44",
      "sessions": 3, "reach": "ads_only", "suppressed": false,
      "lastActivityAt": "2026-10-08T09:40:00+07:00", "abandonState": "none" },
    { "id": "4821", "kind": "customer", "name": "Sokha Chan", "phoneMasked": "(+855) 12 ••• 501",
      "deviceLabel": "Android · app 3.4.1", "registered": true, "furthest": "service_viewed",
      "source": "Meta · Instant_15", "adset": "Phnom Penh 25-44", "reach": "push", "suppressed": false,
      "lastActivityAt": "2026-10-08T09:42:00+07:00", "abandonState": "none" }
  ],
  "nextCursor": null
}
```

- `count`, `reach`, `breakdown` describe the whole filtered list, not the page. `reach.push + cs + adsOnly = count`.
- `breakdown` keys: `moved_on`, `stopped_here`.
- `abandonState` is always `none`.
- Sort: last activity, newest first.
- `422 NOT_LISTABLE` for `ad_clicked`.

### Export — `POST /funnels/acquisition/stages/{stage}/people/export`

Perm: EXPORT. Same body and response as the booking export. Devices export the device id instead of a phone.

---

## 3. `GET /funnels/acquisition/after-purchase/{key}/orders`

The orders behind an `afterPurchase` number. `key` as in the table above. Query: the filters in §1 + `cursor`, `limit`.

```json
{
  "count": 31,
  "items": [
    { "orderId": 5193, "customerId": 4821, "customerName": "Sokha Chan", "service": "Upholstery",
      "status": "completed", "total": 51, "amount": 5, "placedAt": "2026-10-02T10:00:00+07:00",
      "slotAt": "2026-10-05T14:00:00+07:00", "source": "Meta · Instant_15" }
  ],
  "nextCursor": null
}
```

`amount` = this order's part of the key's amount (fee, discount or refund). `total` includes VAT. Newest first.

---

## 4. `GET /meta/filter-options`

```json
{
  "services": [ { "id": 1, "name": "Instant Cleaning" } ],
  "campaigns": [
    { "key": "meta:Instant_15", "name": "Instant_15", "channel": "meta",
      "adsets": [ { "id": "238841", "name": "Phnom Penh 25-44" } ] }
  ],
  "devices": ["android", "ios", "web"]
}
```
