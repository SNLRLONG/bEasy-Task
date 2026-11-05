# Pending Backend Requests: Customer Journey

**To:** Backend · **Updated:** 4 Nov 2026 (CS Action and Performance asks and segment data clean-up: item 18; Direct Sale create attachment: item 17; Direct Sale address Save as name: item 16; reschedule reason, limit and count for the admin portal: item 15; single handoff for items 9 to 12: [App_Tracking_Backend_Handoff_3Nov2026.md](App_Tracking_Backend_Handoff_3Nov2026.md), the earlier 2 Nov file is withdrawn; delta file item 14; booking, checkout and top-up ids in item 9; profile, language, support and chat ids in item 9; landing page public API item 13; location item aligned with T-0077; Instant item aligned with T-0082; cancel and reschedule item 11 aligned with T-0079; orders ids and review params in item 9)
**Done and removed:** guest events, stage/people endpoints, customer page, acquisition funnel (see [Journey_V2_Portal_Review.md](../backend-response/Journey_V2_Portal_Review.md)). Later items are in [backlog.md](backlog.md). Campaign source is in [Campaign_Source_Request.md](Campaign_Source_Request.md). CS Overview replies are in [CS_Overview_Backend_Answers_31Oct2026.md](../backend-response/CS_Overview_Backend_Answers_31Oct2026.md); still open: section 8 below (what is missing for real data).

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

Backend answered on 20 Oct: [Journey_Tech_Review_Backend_Response.md](../backend-response/Journey_Tech_Review_Backend_Response.md). Nothing is open in [Journey_Tech_Review_Backend_Request.md](Journey_Tech_Review_Backend_Request.md); the OTP reasons question was answered on 30 Oct: [Journey_OTP_Reasons_Backend_Response_30Oct2026.md](../backend-response/Journey_OTP_Reasons_Backend_Response_30Oct2026.md). Backend's earlier summary: `../backend-response/Journey_Tech_Review_Handoff_19Oct2026.md`.

Not requested now (backlog): assisted-conversion identifier, finance amounts, consent data, people-list filters for last activity, furthest stage, value, CS status, full phone in journey APIs (`backlog.md`).

## 7. UTM fields per placement (21 Oct 2026)

Task T-0035. Marketing will tag every social link with `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_id`, `utm_term` (template `bEasy_UTM_Tracking_Template.xlsx`). Please agree the join key with us first (T-0033; we recommend `utm_id`), then:

1. Accept and store `utm_id`, `utm_content`, `utm_term` on the customer attribution touch and on `APP_OPENED` (new fields from mobile, T-0034).
2. Return them in `GET /funnels/acquisition` (campaign rows and people lists) and `GET /meta/filter-options`, and allow filtering by them.
3. Keep using the stored touch when an open has no campaign (see [Campaign_Source_Request.md](Campaign_Source_Request.md)).

Not asked yet: exact header and field names. They are fixed in T-0033.

## 8. CS Overview: what is still missing for real data (2 Nov 2026)

Moved to its own file so backend can read one document: [CS_Overview_Backend_Request_2Nov2026.md](CS_Overview_Backend_Request_2Nov2026.md) (8 asks: contact capture, CS cases and Today numbers, response time, Problems found decision, `doubleContacted`, one-user permission mismatch, open handoff contracts, legacy contacts). `POST /customer/counts` is live on dev, so that ask is closed.

## 9. App usage and health events (2 Nov 2026)

Task T-0070. Full contract: [App_Tracking_Coverage_Spec.md](../architecture/App_Tracking_Coverage_Spec.md) §6. The app will send new generic events (`screen_viewed` extended, `screen_exited`, `overlay_viewed`, `element_clicked`, `form_submitted`, `search_performed`, `search_result_clicked`, `setting_changed`, `permission_result`, `support_contacted`, `app_error`, plus a few domain events listed in §3.4 of the spec). Please:

1. **Stop skipping `screen_viewed`** and accept the new events. Dedupe by `event_id`; quarantine only on bad `event_id`, `schema_version` or `seq`. Store unknown `screen_name` / `element_id` flagged `unregistered`, do not reject.
2. **Store them outside `event_log`**: tables `app_screen_view`, `app_interaction_event`, `app_error_event`, registries `ref_app_screen`, `ref_app_element`, and the read model `app_usage_daily` (columns in spec §6.2). Write a subset (`screen_viewed`, `support_contacted`, `app_error`, `search_performed`, `cancellation_*`) to `event_log` too for the Customer page timeline.
3. **Add endpoints** (spec §6.4): `GET /meta/app-screens`, `/app-usage/overview`, `/app-usage/screens`, `/app-usage/screens/{screen_name}`, `/app-usage/elements`, `/app-usage/flows`, `/app-usage/errors`, `/app-usage/search-terms`, `POST /app-usage/export`, and `include=interaction` on `GET /customers/{id}/activity`. Same filters as Journey (`period`, `platform`, `app_version`, `language`, `auth_state`, `source`, `campaign_id`, `service_id`, `audience`).
4. **Write backend-owned** `topup_completed`, `topup_failed` from the wallet record (like `payment_failed`).
5. Confirm retention (raw 90 days, errors 180 days, aggregates 2 years) and an expected row-volume estimate per day.

Registry rows: `ref_app_screen` and `ref_app_element` are seeded from the ids the app actually sends (launch, home, banners, notifications and tab ids are in `DiscoveryIds`; list in spec §4 A/C/G shipped). Unknown ids are stored as `unregistered`. Profile, language, help, support-sheet and chat ids are in `ProfileIds.all` / `ProfileIds.allOverlays` (`lib/analytics/profile_tracking.dart`). Booking, checkout and top-up ids (T-0065) are in `BookingIds.all` / `BookingIds.allOverlays` (`lib/analytics/trackers/booking_tracking.dart`); its `app_error` `endpoint_group` values are `BookingIds.allGroups`, listed in spec §4 D/E/J shipped; the app sends `topup_started|amount_selected|payment_started` and `coupon_list_viewed` and never `topup_completed|topup_failed`. Orders ids (T-0066) are in `OrdersIds.all` / `OrdersIds.allOverlays` (`lib/analytics/trackers/orders_tracking.dart`); its `app_error` `endpoint_group` values are `booking_list`, `order_detail`, `order_complete`; `form_submitted` `form_id=review` carries `result`, `tags_count`, `has_comment` (0/1) and `error_code` (`http_<n>`/`network`), never text; order detail customer-service buttons send `support_contacted channel=chat` with `order_id`. Home and discovery (T-0064), account, profile and chat (T-0067) and booking, checkout and top-up (T-0065) and orders (T-0066) tracking are built in the app, not yet released. `setting_changed` language uses the app's codes `en|km|vi|zh-cn|zh-tw|other`; `chat_opened` carries `chat_kind` and `entry`, `chat_message_sent` only `message_type`.

Ask for a staging date first; the portal builds against the contract with mock data until then.

## 10. Registration funnel order, registration entry, location events (2 Nov 2026)

Tasks T-0075, T-0077. Contract: [Registration_And_Location_Flow_Spec.md](../architecture/Registration_And_Location_Flow_Spec.md) §A.4-A.6, §B, §C.3. The app changed on 27 Oct: for a new number the user types the OTP code, fills the sign-up form, and only then is the code **verified** (`otp_verified` fires after `signup_viewed`). Please:

1. **Reorder the registration stages** to `registration_viewed → otp_requested → signup_started → otp_verified → registration_completed`. Read `otp_verified` from the event, not inferred from completion.
2. **Store per attempt** (`reg_flow_id`): `entry`, `entry_detail`, `source_screen`, `flow_type`, `otp_mode`, `otp_rounds`, and timestamps for each step (table in spec A.6). Accept new events `otp_code_entered`, `signup_submitted` (fires when the create call returns, before the held code is verified), `account_created`, and later `registration_resumed` (not in the app yet). Map `entry` `gate` to `other` and `checkout_gate` to `checkout_address` (the app sends no `checkout_summary` today). `otp_failed.error_code` keeps today's app values; tell us whether you can distinguish `wrong_code`, `blocked`, `expired` so the app can send them.
3. **Funnel API:** `GET /funnels/registration` accepts `entry` and `group_by=entry`, returns `entryBreakdown[]`; stage people rows and the customer activity registration group include `entry`. Add `GET /attribution/registration-entries` (registered, first purchase within 7 / 30 days, per entry).
4. **Location:** accept `service_area_checked`, `address_added|edited|deleted`, `permission_result` (`context` `use_current_location|auto_open`), `location_pin_moved`; table `app_service_area_check` (`in_range` and `address_id` nullable) and `GET /app-usage/locations` (spec C.2, C.3). A failed coverage check has `result=error` and no `in_range`: do not count it as out of range; a UI timeout arrives as `app_error` `endpoint_group=coverage`. `address_default_set` is not sent. `address_selected` adds `source=checkout` and `method`. Never expect street text or exact coordinates; `area_cell` only.

Which of these can ship before the app release? Item 1 and 3 can, once the old event order is accepted. Owner decision (2 Nov): the app flow does not change; `sign_up` / `CompleteRegistration` keep firing at account creation, and a new `account_created` event marks it. Count "registration completed" from `registration_completed` only.

## 11. Cancellation and reschedule: records, fields, APIs (2 Nov 2026)

Task T-0080. Full contract: [Cancellation_And_Reschedule_Spec.md](../architecture/Cancellation_And_Reschedule_Spec.md) §3. This replaces items 3.3 and 3.4 above once agreed (then delete them). The app flow does not change; we only add optional fields and events. The client events are built in the app (T-0079, not yet released); the optional request fields in item 3 are **not** sent yet and wait for your agreed names. Slots in `reschedule_*` events are ISO-8601 with `+07:00`. Please:

1. **Store a cancellation record** (`order_cancellation`) for every cancel, whoever does it (customer app, admin portal incl. ABA mini-app orders, system): `cancelled_by`, `channel`, `reason_code`, `reason_text`, `lead_minutes`, `order_status_before`, `partner_assigned`, `fee_rule_code`, `fee_amount`, `fee_waived`, `refund_amount`, `refund_destination`, `refund_status`, `reschedule_count_before`. Columns in spec §3.2.
2. **Store a reschedule record** (`order_reschedule`) for every attempt, including rejected ones with `failure_code`: old/new slot, `shift_minutes`, `lead_minutes_before_old_slot`, `reschedule_no`, `is_free`, `fee_*`, `initiated_by`, `channel`, `partner_changed`.
3. **Accept optional new fields** on the existing calls, old apps still send nothing: cancel `{ reasonCode, reasonText, source }` on `POST customer/order/v2/{id}/cancel`; `{ source, entry }` on `POST customer/order/bulk/edit-schedule`. Reason enum in spec §2.1; unknown or missing reason is stored as `null` and shown as "Not recorded".
4. **Fee facts:** confirm the fee rules and windows (`cancellation_fee_rule`), where they live today, and who may waive a fee. Provide `GET customer/order/v2/{id}/cancel/preview` and a reschedule fee quote so the app can later show real fees (separate app task; until then the app shows placeholders and sends no `cancellation_fee_shown` / `reschedule_fee_shown`).
5. **Event log:** write `order_cancelled`, `order_rescheduled`, `order_refunded` with the fields in spec §3.3; whitelist the client funnel events in spec §3.4 (resolves 3.4); the app does not send `service_id`, `reschedule_count`, `is_free` or `fee_amount` on these events.
6. **Endpoints** in spec §3.6: `/after-purchase/cancellations/summary`, `/after-purchase/cancellations`, `/after-purchase/cancel-funnel`, `/after-purchase/reschedules/summary`, `/after-purchase/reschedules`, `/orders/{bulkOrderId}/changes`, `/meta/cancellation-reasons`, `/meta/fee-rules`; fill `afterPurchase.rescheduled` and `cancelled` in `/funnels/acquisition`. Counting modes `byCancelDate` (default) and `cohort`; explicit denominators; `null` never 0.
7. **Confirm** `order_id` in app events is `bulkOrderId`, and how it maps to your order id; and the daily reconciliation of client `cancellation_result=success` vs records.

Questions: can items 1, 2, 5 ship before the app release? What is the earliest date for the fee preview (item 4)?

## 12. Instant Cleaning (2 Nov 2026)

Tasks T-0083, T-0082 (app build, not merged). Contract: [Instant_Cleaning_Tracking_Spec.md](../architecture/Instant_Cleaning_Tracking_Spec.md) §4. The app flow does not change; we add `booking_mode` (`instant`/`scheduled`; absent when the app does not know it yet and on events without `service_id`) to existing events and send new Instant events; `instant_hours` (on `checkout_started` and `payment_started`, Instant only) is built in T-0065. Shipped differences: `instant_upsell_result` has `returned` only (no `converted`), `instant_progress_viewed` has no `order_id`, `auto_rooms` is never sent, and `schedule_grace_result` has no `service_id`/`booking_mode` (availability, fully-booked, rooms, duration, upsell, schedule/grace errors, progress). Please: (1) accept them and store `booking_mode` on journeys and orders so Instant is a filter in every funnel, After-purchase and order report; (2) add the three tables and store every rejected grace/schedule request with its error code server-side; (3) add `GET /instant/overview|rooms|availability|schedule-errors|upsell|orders/progress` and `booking_mode` on `GET /funnels/booking`; (4) confirm the booking-mode values, the error-code list, and whether an extend payment API/record is planned.

## 13. Landing page public API (3 Nov 2026) - A is low severity

Plan: [Landing-Page-public-api-content-sync.md](../../../../Tasks/_plans/Landing-Page-public-api-content-sync.md) · Task: T-0098.

**A. Anonymous requests map to one shared guest user (low severity).** With **no token**, `GET /api/customer/user/info` returns 200 with the shared `GUSTMODE` guest row (id 1508 on prod), including its `deviceId`, `count` and `updatedAt`. `GET /api/customer/order/v5/list` also returns 200 with empty `IN-PROGRESS`/`COMPLETED` lists (guest has no orders; no issue). Please confirm this is intended, and drop `deviceId`/`fcmToken`/`referralCode` from guest responses (or return 401 to calls with no guest id).

**B. Confirm these are intentionally public and stable:** `GET customer/category/list`, `GET customer/category/{id}/equipments`, `customer/banner/list`, `customer/homepage` (banners/announcements only).

**C. Missing for the site:** (1) a public `startingPrice` + currency per category in `category/list` (`product.amount` is 0 today); (2) which category/product is Instant Cleaning and General Cleaning (prod list has neither); (3) a slim public app-version shape without `saleTarget`/force-update flags; (4) `Cache-Control` that allows caching on these public routes (now `private, no-store`, `Vary: Authorization`); (5) `customer/category/popular` returns 500.

## 14. App tracking: changes since the 2 Nov handoff (3 Nov 2026)

[App_Tracking_Backend_Delta_3Nov2026.md](App_Tracking_Backend_Delta_3Nov2026.md). No new asks: two `endpoint_group` values added to the registry (`review_options`, `places`), chat error events announced for a later release (T-0100), and confirmation that the build adds no API calls. The open items stay in §9 to §12.

## 15. Admin portal order reschedule: reason, limit, count (4 Nov 2026)

Task T-0095 (portal order details, Figma "Reschedule rules" V3.4 B2 / V4). The portal reschedule dialog already calls `PUT order/reschedule/{id}` with `scheduleStartDate` (+ `expectedRevision` for Direct Sale) and `GET order/reschedule/{id}/options`. We show Figma's Current / New dialog now. These are missing, so the dialog skips them until you reply. Related to item 11 (same records), not a duplicate:

1. **Reason:** accept `reason` (required, text) on `PUT order/reschedule/{id}` and store it on the reschedule record and the order history.
2. **Max 2 reschedules:** enforce it server side and return a clear error ("Changed twice already"). Add `rescheduleCount` (completed changes, 0 to 2) to the order detail so the portal can show "Changes used: X of 2" and disable the button.
3. **30-day rule:** new date must be within 30 days of the FIRST booked date and inside the 30-day booking window. Enforce it and drop later days from `.../options` so they show greyed.
4. **Answer:** does the current endpoint already enforce 2 or 3? The portal has no check today.

## 16. Direct Sale add address: Save as name (4 Nov 2026)

Task T-0096 (Figma create | add address). Figma has an optional Save as field (Home, Condo, Office). Not built in the portal until you confirm: does `POST direct-sale/add-users-address` accept a name for the address (field name), and is it returned on `direct-sale/users-address`?

## 17. Direct Sale create: attachment on managed (payment link) orders (4 Nov 2026)

Task T-0096 (Figma create | Notes & attachments). Tested on dev: adding a file field `attachment` to the managed create call (`paymentMode=PAYMENT_LINK`) fails with `{"code":"ERROR","message":"Unexpected field"}`, so the upload box is not shown for managed orders. Please accept one optional file (JPG, PNG or PDF, up to 10 MB) on the managed create, tell us the field name, and store it as a note attachment on the order, not as a payment receipt. Legacy orders still send the receipt as `imgUrl`.

## 18. CS Action and Performance (5 Nov 2026)

Moved to its own file so backend can read one document: [CS_Action_Performance_Backend_Request.md](CS_Action_Performance_Backend_Request.md). It covers the saved-segment clean-up and seed from production (A), switching the CS API on and the scheduler (B1, B2), endpoint gaps (C1 to C5), new endpoints, the Performance response shape and monthly targets per CS (D1 to D5) and the policy decisions (E1).

## Questions

1. Retention of the phone number saved at OTP request for people who never register?
2. Can `payment_failed` be written from the payment record, with the order or journey ID?
3. Which of 3.4 can ship without an app release?
