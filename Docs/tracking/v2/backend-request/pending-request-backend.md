# Pending Backend Requests: Customer Journey

**To:** Backend · **Updated:** 21 Oct 2026
**Done and removed:** guest events, stage/people endpoints, customer page, acquisition funnel (see [Journey_V2_Portal_Review.md](../backend-response/Journey_V2_Portal_Review.md)). Later items are in [backlog.md](backlog.md). Campaign source is in [Campaign_Source_Request.md](Campaign_Source_Request.md).

The app already ships these Firebase events, so it cannot rename them. Where our names differ from the backend contract, please map ours.

## 1. Map mobile parameter names (no app change)

| Event | Mobile parameter | Backend name | Action |
|---|---|---|---|
| `service_option_selected`, `pair_service_added`, `pair_service_removed` | `option_id` | `product_option_id` | Map it |
| `service_option_selected` | `option_group_id` | none | Keep in `metadata` |
| `pair_service_selected`, `pair_service_added` | `pair_service_id` | `product_id` (to confirm) | Map if it is a product ID, else `metadata` |
| `addon_selected`, `addon_quantity_changed`, `addon_removed` | `addon_id` | `product_option_id` (to confirm) | Map if it is an option ID, else `metadata` |
| `addon_*` | `addon_group_id`, `quantity`, `value`, `currency` | none | Keep in `metadata` (`value` is a double, `currency` is `USD`) |

**Please reply** with the final mapping for the two "to confirm" rows.

**`address_selected`:** the app sends `area_cell` (about 1 km grid, e.g. `11.56_104.92`) and `zone_id` only when the address has one. Do not quarantine when `zone_id` is missing and `area_cell` is present. Keep `area_cell` in `metadata`.

**Skip silently** (not errors): `view_content`, `select_service`, `install_amount`, `splash_ad_screen`, `splash_cta_button`, `splash_skip_button`, `app_backgrounded`, `app_foregrounded`, `order_rescheduled`, `review_submitted`, `screen_viewed`, `banner_clicked`, `click_promote`, `referrer_install`, `skan_install`, `skan_open`.

## 2. Read `X-Journey-Id` on more endpoints

The app sends it on all of these. Today backend reads it only on `POST customer/order/v6/create`.

| Endpoint |
|---|
| `customer/order/v6/preview` |
| `customer/order/v6/apply-coupon`, `customer/coupon*` |
| `customer/order/v5/preview/summary` |
| `customer/order/v6/extra-addon` |
| `customer/order/*available-schedule*` |
| `payment/*checkout*` |

**Why:** Firebase events can be lost, and the BigQuery import is daily. An API call proves the user reached the step, so abandonment and recovery pushes can use it within minutes. It also ties coupon and price choices to the final order.

**Rules:**
1. The header is optional. Missing or malformed means null, and the request must not fail.
2. Accept a `journey_id` only if it belongs to the same customer or the same `x-app-instance-id`. Otherwise ignore it.
3. Tell us how a journey is handled when one order has several services (pair or cross-sell). The app sends the journey of the service being ordered.
4. Gateway, WAF and CORS must allow `X-Journey-Id`, `X-App-Instance-Id`, `X-Reg-Flow-Id`, `X-Campaign-Code`.

## 3. Data still missing (from the 14 Oct review)

| # | Priority | Request | Why |
|---|---|---|---|
| 3.1 | **A** | Abandon rules: `PUT /abandon-rules` (now `501`), the sweep that sets Abandoned, immediate clear on purchase | Lists show Abandoned vs Not abandoned yet |
| 3.2 | **B** | OTP request record: delivery status, wrong attempts, lock, resend count, reason | OTP-failed list |
| 3.3 | **B** | Track reschedule (old/new slot, fee), cancellation fee kept, refund amount | After-purchase shows `null` for these today |
| 3.4 | **B** | Accept `banner_clicked`; decide on `search_performed`, `search_result_clicked`, `reschedule_started`, `cancellation_started`, `cancellation_fee_shown` | Event log rows |
| 3.5 | **B** | Device OS and app version per customer and guest device | Customer header, list rows |
| 3.6 | **B** | `/customers/{id}/contacts` returns `[]` today; confirm when real data comes | Customer page |

## 4. Portal ↔ backend contract follow-ups (from the 14 Oct review)

| # | Request |
|---|---|
| 4.1 | **Remove web:** drop `web` from `filter-options.devices` and `filterCounts.devices`; treat `device=web` as `422 INVALID_FILTER` (decided 14 Oct, no web app data) |
| 4.2 | **Send one real `GET /meta/filter-options` response** and confirm `channel` is `all`, `meta`, `tiktok` or `other` (default `all`) |
| 4.3 | **Confirm ids:** campaign key is `meta:<name>` in filters, `campaigns[].campaignId` and filter-options `key`; ad set id is a string such as `"238841"` |
| 4.4 | **Custom period:** is `to` exclusive (like presets)? Portal sends the day after the last picked day |
| 4.5 | **`refunded.amount`:** send `null` until refund amounts are stored, then the real number |
| 4.6 | **Remove `/customers/{id}/event-log`:** the portal now uses `/activity` |
| 4.7 | **Roles:** none needed now (admin login only). Revisit when roles are planned |

Mobile (not backend): set Firebase `user_id` to the real user id on login, clear it on logout, never use a `GUSTMODE` id, and keep `journey_id` across guest login. Needed before end-to-end sign-off.

## 5. Marketing suite

Push campaigns, promo-code ledger, package deals, content library and search: see [Marketing_Suite_Backend_Request.md](Marketing_Suite_Backend_Request.md).

## 6. Tech review handoff (19 Oct 2026)

Backend answered on 20 Oct: [Journey_Tech_Review_Backend_Response.md](../backend-response/Journey_Tech_Review_Backend_Response.md). What is still open is in [Journey_Tech_Review_Backend_Request.md](Journey_Tech_Review_Backend_Request.md): dev deploy with masked samples (including `summaryScopes` and `definitions`), the 12 to 18 Oct reconciliation on production, registration stages from backend records, and the real-data check for duplicate orders. Backend's earlier summary: `../backend-response/Journey_Tech_Review_Handoff_19Oct2026.md`.

Not requested now (backlog): assisted-conversion identifier, finance amounts, consent data, people-list filters for last activity, furthest stage, value, CS status, full phone in journey APIs (`backlog.md`).

## 7. UTM fields per placement (21 Oct 2026)

Task T-0035. Marketing will tag every social link with `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_id`, `utm_term` (template `bEasy_UTM_Tracking_Template.xlsx`). Please agree the join key with us first (T-0033; we recommend `utm_id`), then:

1. Accept and store `utm_id`, `utm_content`, `utm_term` on the customer attribution touch and on `APP_OPENED` (new fields from mobile, T-0034).
2. Return them in `GET /funnels/acquisition` (campaign rows and people lists) and `GET /meta/filter-options`, and allow filtering by them.
3. Keep using the stored touch when an open has no campaign (see [Campaign_Source_Request.md](Campaign_Source_Request.md)).

Not asked yet: exact header and field names. They are fixed in T-0033.

## Questions

1. Retention of the phone number saved at OTP request for people who never register?
2. Can `payment_failed` be written from the payment record, with the order or journey ID?
3. Which of 3.4 can ship without an app release?
