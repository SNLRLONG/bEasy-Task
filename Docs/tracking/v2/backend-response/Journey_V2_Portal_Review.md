# Customer Journey v2 — admin portal review & confirm

**From:** backend (bEasy-api) · **To:** admin portal team · **Date:** 14 Oct 2026
**Base:** `{API}/api/admin` · existing admin Bearer token · success `{ data }` · errors `{ error: { code, message, field } }`
**Status:** implemented and unit-tested locally on `feature/bunlong-tracking`. **Not committed, merged, or deployed yet** — keep mocks until backend confirms it is live on dev.

---

## 1. What changed for you

| # | Change | Your action |
|---|---|---|
| 1 | **`/customers/{id}/activity`** added (your ad-blocker request) | Switch from `/event-log`. Same params, response, errors. Tell us when done → we remove `/event-log` |
| 2 | **Acquisition funnel** implemented (your `Journey_Acquisition_API_Request.md`) | Switch from mock once dev is live |
| 3 | **Guests in the booking funnel** — `customerTypeCounts.guest` and `customerType=guest` now return real data | Handle `kind: "device"` rows in the booking people list |
| 4 | Custom period capped at **92 days** (all funnels) | Handle `422 PERIOD_TOO_LONG` (`field: "to"`) |

---

## 2. Endpoint status

| Screen / feature | Endpoint | Status |
|---|---|---|
| Booking funnel + people + export | `GET /funnels/booking`, `…/stages/{stage}/people`, `POST …/export` | ✅ (now includes guests) |
| Registration funnel + people + export | `GET /funnels/registration`, … | ✅ |
| **Acquisition funnel** | `GET /funnels/acquisition` | ✅ new (no ad data yet, see §3) |
| **Acquisition people + export** | `GET /funnels/acquisition/stages/{stage}/people`, `POST …/export` | ✅ new |
| **After-purchase orders** | `GET /funnels/acquisition/after-purchase/{key}/orders` | ✅ new |
| **Filter options** | `GET /meta/filter-options` | ✅ new |
| Customer page | `/customers/{id}/summary`, `/journey-canvas`, `/sessions`, **`/activity`**, `/orders`, `/contacts` | ✅ (`/contacts` returns `[]`) |
| Abandon rules | `GET /abandon-rules` ✅ · `PUT /abandon-rules` | **View only** — `PUT` returns `501` (keep edit disabled) |
| Integrations | `GET /integrations` | ✅ `meta` / `tiktok` `connected: false` |
| CS (cases, contacts, overview) | `/cs/*` | ⛔ `501` — next phase |
| Audiences | `/audiences*` | ⛔ `501` — keep hidden |

> Correction to our earlier checklist: CS and abandon-rule **saving** are **not** available in this release.

---

## 3. Acquisition — how backend fills your spec

### Without Meta (now)
| Field | Value |
|---|---|
| `stages[ad_clicked]` | `reached: null`, `dropped: null`, `continued: null`, `pctOfFirst: null`, `listable: false`, **`unavailableReason: "meta_not_connected"`** |
| `baseStage` | `"app_opened"` (so `pctOfFirst` is relative to app opens) |
| KPIs `ad_clicks`, `click_to_buy` | `value: null`, `previous: null` |
| `filterCounts.adsets` | `{}` |
| `campaigns[].adClicks`, `clickToBuy` | `null` |
| People `adset` | `null` |
| `adsetId` filter | any value → `422 INVALID_FILTER` (no ad sets without Meta) |

### Counting rules (please confirm they match your intent)
- **Cohort:** people whose **first app open** (app install) is in `period`. Install data starts from the day guest mode goes live — earlier periods will show **0**.
- **Person:** the customer when the phone is linked to exactly one customer; otherwise the device (`unit: "devices"`). A customer with several phones counts once, from the earliest.
- **7-day window:** later stages count only what happens within 7 days of the first open.
- **`registered`:** account exists by first open + 7 days. Stages after `registered` count registered people only.
- **`customerKind=existing`:** had a paid order before the first open.
- **`first_purchases`:** cohort people whose **first-ever** paid order is inside their 7 days.
- **Source / campaign:** from app campaign info, else Firebase first-install source. **Mostly `"Organic"`** until the mobile app sends campaign info.
- `campaigns[]` follows all filters except `campaignId` / `adsetId`.

### After-purchase (`afterPurchase` + `/after-purchase/{key}/orders`)
| Key | What you get |
|---|---|
| `payment_failed_then_paid` | ✅ (app reported a failed payment within 30 min before the confirmed purchase) |
| `auto_promo` | ✅ `detail.auto` / `manual` from the coupon's auto-apply setting |
| `rescheduled` | ⚠️ **`orders/ofPurchases/amount: null`, `detail` values `null`, `unavailableReason: "not_tracked"`**. Its orders endpoint returns `count: null`, `items: []` |
| `cancelled` | ✅ `orders` · ⚠️ `amount: null`, `detail.withFee/free: null` (fee kept not stored) |
| `refunded` | ✅ `amount` = full order total (refund amount not stored separately) |
| `service_completed` | ✅ incl. `detail.upcoming` |

### Errors
| Case | Response |
|---|---|
| `ad_clicked` people list | `422 NOT_LISTABLE` |
| Unknown stage / after-purchase key | `404 UNKNOWN_STAGE` / `404 UNKNOWN_KEY` |
| `state` missing / `dropped` on `purchased` | `422 INVALID_STATE` |
| Unknown `channel`, `customerKind`, `device`, `campaignId`; `adsetId` | `422 INVALID_FILTER` |
| Custom range > 92 days | `422 PERIOD_TOO_LONG` |

---

## 4. Guest rows (booking + acquisition people lists, export)

```json
{ "id": "dev_a91f8b7c6d5e43c2", "kind": "device", "deviceId": "a91f…3c2",
  "name": null, "phoneMasked": null, "registered": false, "reach": "ads_only", ... }
```
- Use `id` as the row key (prefix `dev_`). `deviceId` is shortened for display.
- Search `q` matches name/phone only, so guests drop out of search results.
- Export: guests show `Device <id>` instead of name/phone; the response `guests` counts them.
- Guest = **device**, not a person: a reinstall or a new phone is a new device.

### Event ownership rules

- An event is stored as a **customer event** only when its reported user ID belongs to a verified `users.type = "USER"` account.
- An event with a missing, unknown, or `GUSTMODE` user ID is stored as a **device event** when `app_instance_id` is present.
- `reported_user_id` on a device event is audit information only. It does not make the event customer-owned.
- An event with neither a valid customer ID nor an `app_instance_id` is skipped because it cannot be assigned safely.
- The same `event_id` is not counted twice across customer and device activity. For historical duplicates, reporting prefers the customer event.

### Login, logout and reinstall behavior

- **Guest → login:** earlier anonymous events remain device events; events after successful login become customer events.
- If both parts use the same `journey_id`, reporting can present the guest and customer activity as one continuous journey without rewriting the raw guest rows.
- **Customer → logout:** events after logout return to device ownership, even if Firebase continues the same analytics session.
- Returning later from the same app installation keeps the same device identity.
- Uninstalling and reinstalling normally creates a new Firebase app instance and therefore a new guest device.

### Mobile identity contract

- After successful login, set Firebase Analytics `user_id` to the real backend `users.id`.
- On logout, clear Firebase Analytics `user_id` by setting it to `null` before tracking more guest activity.
- Never set Firebase `user_id` to a `GUSTMODE` database ID.
- Send `x-app-instance-id` using the same Firebase installation identity exported to BigQuery as `user_pseudo_id`.
- Keep the same `journey_id` when a guest logs in and continues the same booking journey.

### Database status

The existing `tracking_device_events` table and indexes are sufficient for these rules. No new column, table, migration, `ALTER TABLE`, reset, or backfill is required for this update.

---

## 5. UI checklist

- [ ] Use `/customers/{id}/activity`; tell us when `/event-log` can be removed
- [ ] Show `null` as "—" / "Not available" — never 0 (ad fields, `rescheduled`, cancellation fee)
- [ ] Use `unavailableReason` to explain an empty value (`meta_not_connected`, `not_tracked`)
- [ ] Acquisition: when `baseStage` is `app_opened`, hide or grey out the ad-click column/KPIs
- [ ] Handle `kind: "device"` rows in booking and acquisition lists
- [ ] Do not open a customer profile for a `kind: "device"` row
- [ ] Handle `422 PERIOD_TOO_LONG` on custom date pickers
- [ ] Keep abandon-rule edit, CS and audiences disabled / hidden (`501`)

---

## 6. Please confirm (reply ✅ or what to change)

| # | Question | Backend today |
|---|---|---|
| 1 | Acquisition cohort = first **install** in the period, 7-day window | As described in §3 |
| 2 | A customer with 2 phones counts once (earliest install) | Yes |
| 3 | `first_purchases` = first-ever paid order within the 7 days | Yes |
| 4 | `campaigns[]` should also follow service / device / customerKind filters? | Yes (only `campaignId` / `adsetId` ignored) |
| 5 | `device=web` | Always 0 — there is no web app data |
| 6 | Spec mentions **Perm VIEW / EXPORT** | Routes check admin login only. Do you need role permissions enforced? Which roles? |
| 7 | 92-day custom period limit | OK for your `90d` preset? Any longer range in the design? |
| 8 | `/meta/filter-options` path | Works; please check it is not blocked by ad blockers (contains `meta`) |
| 9 | `refunded.amount` = full order total | OK, or show `null` until we store refund amounts? |

---

## 7. Test on dev (after we confirm it is live)

Test customers: **3370, 3547, 3555, 3526**.
- [ ] `GET /customers/3370/activity` returns the same as `/event-log`
- [ ] `GET /funnels/booking?period=7d` → `customerTypeCounts.guest` > 0; `customerType=guest` list shows device rows
- [ ] `GET /funnels/acquisition?period=7d` → 9 stages, `ad_clicked` null with `meta_not_connected`
- [ ] `…/acquisition/stages/app_opened/people?period=7d&state=dropped` → device rows, paging works
- [ ] `…/acquisition/after-purchase/service_completed/orders?period=7d`
- [ ] `GET /meta/filter-options`
- [ ] Export one acquisition list → CSV opens, guests show `Device …`
- [ ] Anonymous or `GUSTMODE` browsing appears as `kind: "device"`, not as a customer
- [ ] Guest starts a journey, logs in, and completes it with the same `journey_id`
- [ ] Logged-in customer logs out; later browsing remains guest/device activity
- [ ] Returning on the same installation keeps the device identity
- [ ] Uninstall/reinstall creates a new device identity
- [ ] One `event_id` is not counted in both customer and device results
- [ ] Custom period of 120 days → `422 PERIOD_TOO_LONG`

Please send **endpoint + request + response** for anything that does not match your UI.

---

## 8. Still open on our side

| Item | Blocked by | Portal impact |
|---|---|---|
| Ad clicks, ad sets, campaign clicks | Meta / TikTok access | `null` values above |
| Real campaign names | Mobile app campaign info | Mostly "Organic" |
| Guest/customer identity contract | Mobile must set the real user ID on login, clear it on logout, and preserve the booking `journey_id` | Required before final end-to-end sign-off |
| Reschedule history + fees, cancellation fee, refund amount | Not stored by backend yet | `null` values above |
| CS, audiences | Next phase | `501` |
