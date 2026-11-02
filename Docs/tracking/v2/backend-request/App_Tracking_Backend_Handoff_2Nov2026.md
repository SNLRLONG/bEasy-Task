# App tracking: what we need from Backend

**To:** Backend · **From:** bEasy app and portal · **Date:** 2 Nov 2026 · **Updated:** 2 Nov 2026

One document for four areas. Full detail and rationale live in the specs in `Docs/tracking/v2/architecture/` (names in each section); this file is the extract Backend needs to plan, estimate and reply. **The app flow does not change.** We only send more data and ask Backend to store, map and expose it. Tracked in `pending-request-backend.md` §9 to §12; when you reply, we move the answer to `backend-response/`.

## 0. Rules that apply to everything

1. **Transport stays Firebase → BigQuery → your importer.** The app does not call a new API to send events.
2. **Every event carries** `event_id` (UUIDv4, dedupe key), `schema_version`, `seq`, plus `service_id` / `journey_id` / `reg_flow_id` / `order_id` where relevant. Quarantine only on a bad `event_id`, `schema_version` or `seq`. **Unknown values are stored and flagged, never rejected** (new screen or element ids will appear with app releases).
3. **Guests** are keyed by `app_instance_id` and merged to `customer_id` at identity link. Keep guest events.
4. **Money and order facts are yours.** Client events about payment, cancel and reschedule are signals only; your records are the truth. `null` means unknown; never return 0 for unknown.
5. **No personal data** in event params (no phone, name, address text, free text, exact coordinates). Location is `area_cell` (about 1 km grid) and `zone_id` when known.
6. Exclude staff phones and test orders, as today. Time is stored UTC and returned ISO-8601 with `+07:00`. Money is USD, two decimals, state whether VAT is included.
7. Interaction events (sections 1 and 4) go to **separate tables**, not `event_log`, because of volume. A listed subset also goes to `event_log` so the Customer page timeline shows it.

## 1. App usage and health events (spec: `App_Tracking_Coverage_Spec.md` §3, §6)

**New client events.** `screen_viewed` (extended), `screen_exited`, `overlay_viewed`, `element_clicked`, `form_submitted`, `search_performed`, `search_result_clicked`, `setting_changed`, `permission_result`, `support_contacted`, `app_error`, and domain events `cancellation_*`, `extend_*`, `topup_started|amount_selected|payment_started`, `coupon_list_viewed|coupon_copied|coupon_use_clicked`, `address_added|edited|deleted|default_set`, `chat_opened`, `chat_message_sent`, `content_shared`, `logout_clicked`.

Key parameters: `screen_name`, `screen_group`, `previous_screen`, `entry_type` (`nav|tab|push|deeplink|back`), `duration_seconds`, `overlay_name`, `overlay_type`, `element_id`, `element_type`, `section`, `position`, `item_id`, `target`, `form_id`, `result`, `error_code`, `error_type` (`api|payment|webview|network|update|exception`), `endpoint_group`, `http_status`, `fatal`. Ids are `snake_case`, 40 characters at most. Labels are not sent; Khmer and English labels live in the registries below.

**Stop skipping** `screen_viewed`. Keep skipping the legacy ad-hoc events until we retire them (list in the spec §3.5).

**Tables** (3NF, additions to your model):

```text
ref_app_screen   (screen_name PK, screen_group, label_en, label_km, is_active)
ref_app_element  (screen_name FK, element_id, element_type, label_en, label_km, PK(screen_name, element_id))
app_screen_view  (event_id PK, session_id FK, customer_id NULL, app_instance_id, screen_name,
                  previous_screen NULL, entry_type, duration_seconds NULL, occurred_at)       -- keep 90 days
app_interaction_event (event_id PK, event_name, occurred_at, session_id FK, customer_id NULL,
                  app_instance_id, screen_name, element_id NULL, element_type NULL, section NULL,
                  position NULL, item_id NULL, target NULL, order_id NULL, metadata jsonb,
                  app_version, platform, app_language, auth_state, booking_mode NULL)         -- partition by day, keep 90 days
app_error_event  (event_id PK, occurred_at, session_id FK, customer_id NULL, app_instance_id,
                  error_type, screen_name NULL, endpoint_group NULL, http_status NULL,
                  error_code NULL, fatal bool, app_version, platform)                         -- keep 180 days
app_usage_daily  (day, screen_name, element_id NULL, platform, app_version, app_language, auth_state,
                  campaign_code NULL, views, unique_users, unique_guests, total_duration_seconds,
                  clicks, clickers, exits, errors)   -- rebuildable read model, nightly + hourly for today
```
Subset also written to `event_log`: `screen_viewed`, `support_contacted`, `app_error`, `search_performed`, `cancellation_*`.

**Endpoints** (filters: `period|from|to`, `platform`, `app_version`, `language`, `auth_state`, `source`, `campaign_id`, `service_id`, `audience`, `screen_group`; masked identifiers; existing pagination):

| Method | Path | Returns |
|---|---|---|
| GET | `/meta/app-screens` | Registry with labels, group, type, count of `unregistered` ids |
| GET | `/app-usage/overview` | Active users, sessions, screens per session, avg session time, top screens and elements, errors, crash-free users, daily trend |
| GET | `/app-usage/screens` | Per screen: views, unique viewers, avg time, exit rate, error rate, sparkline |
| GET | `/app-usage/screens/{screen_name}` | One screen: KPIs, trend, elements with clicks and CTR, previous and next screen shares |
| GET | `/app-usage/elements` | Per element: clicks, clickers, CTR, trend |
| GET | `/app-usage/flows` | Transition counts `from_screen → to_screen`, top N, minimum count |
| GET | `/app-usage/errors` | Grouped by `error_type`, `endpoint_group`, `http_status`, `error_code`: count, affected users, first and last seen, app versions |
| GET | `/app-usage/search-terms` | Top queries, zero-result share, click rank |
| POST | `/app-usage/export` | Excel or CSV of any table above |
| GET | `/customers/{id}/activity` | Existing; add `include=interaction` |

**Definitions we will display:** views = count of `screen_viewed`; unique viewers = distinct customer (or device when guest); avg time = mean `duration_seconds` excluding values over 1800; exit rate = sessions whose last screen is this one ÷ sessions that viewed it; CTR = clickers ÷ unique viewers of that screen; error rate = `app_error` ÷ views. Range-level uniques must be computed from raw rows, not summed from daily rows.

**Backend-owned, please add:** `topup_completed`, `topup_failed` from the wallet record (as `payment_failed` is written from the payment record).

## 2. Registration funnel, registration entry, location (spec: `Registration_And_Location_Flow_Spec.md`)

**What the app does today (since 27 Oct):** for a new number the user enters the phone, the OTP is sent, the user **types the code**, then fills the sign-up form, and only **then** is the code verified together with account creation. So `otp_verified` fires **after** `signup_viewed`. For an existing number the OTP is verified at once (login). We are not changing this.

**1. Stage order.** Please change the Registration funnel to: `registration_viewed → otp_requested → signup_started → otp_verified → registration_completed`. Read `otp_verified` from the event, not inferred from completion. Count each stage as distinct `reg_flow_id` (device while a guest); `otp_viewed` and `signup_viewed` can repeat inside one flow (second OTP after a rejected held code).

**2. New and changed events.**

| Event | Params |
|---|---|
| `registration_started` (extended) | `entry`, `entry_detail`, `source_screen` |
| `otp_viewed` (extended) | `flow_type` (`register|login`), `otp_mode` (`deferred|immediate`), `attempt_round` |
| `otp_code_entered` (new, diagnostic) | `flow_type`, `otp_mode` |
| `otp_submitted`, `otp_failed` (kept, extended) | `otp_mode`; `error_code` also `expired` |
| `signup_submitted` (new) | `result` (`success|error|invalid`), `error_code`, `has_email`, `has_referral` |
| `account_created` (new) | `method`, `entry` (Firebase `sign_up` and Meta `CompleteRegistration` fire at this moment today; unchanged) |
| `registration_completed` (extended) | `method`, `entry` |
| `registration_resumed` (new) | `entry`, `result` (`back_to_checkout|home`) |

`entry` is a closed set: `home | tab_gate | checkout_address | checkout_schedule | checkout_summary | banner | push | deeplink | dialog | onboarding | other`. `entry_detail` is a short code (tab name, banner id, push type). Old values `gate` → `other`, `checkout_gate` → `checkout_summary`.

**3. Table**
```text
registration_attempt (reg_flow_id PK, app_instance_id, customer_id NULL, entry, entry_detail NULL,
  source_screen NULL, started_at, otp_requested_at NULL, code_entered_at NULL, signup_viewed_at NULL,
  signup_submitted_at NULL, otp_verified_at NULL, completed_at NULL, otp_mode, otp_rounds int,
  last_error_code NULL, method NULL, campaign_code NULL)
```
**4. API.** `GET /funnels/registration` accepts `entry` and `group_by=entry` and returns `entryBreakdown[]` `{entry, started, completed, conversion}`. Stage people rows and the customer activity registration group include `entry`. New `GET /attribution/registration-entries`: entry → registered → first purchase within 7 and 30 days.

**5. Location events.** `service_area_checked` (`in_range`, `area_cell`, `address_id`, `result`, `duration_ms`, `source`), `address_added|edited|deleted|default_set` (`method`, `source`, `in_range`, `result`, `has_floor|room|note`), `permission_result` (location: `granted|denied|denied_forever|services_disabled`), `location_pin_moved` (`method`, `area_cell`). Table `app_service_area_check (event_id PK, occurred_at, customer_id NULL, app_instance_id, source, area_cell, zone_id NULL, in_range bool, result, duration_ms)`. Endpoint `GET /app-usage/locations`: `{summary{checks,outOfRange,outOfRangeRate,permissionDenied}, outOfRangeAreas[{areaCell,lat,lng (cell centre),checks,uniqueUsers}], methodBreakdown[], errors[]}`.

## 3. Cancellation and reschedule (spec: `Cancellation_And_Reschedule_Spec.md`)

**Today:** `POST customer/order/v2/{id}/cancel` is sent with an empty body, so the reason never reaches you. You store no reason, fee, refund or reschedule record (`rescheduled` shows "not tracked" in the portal). The fee panels in the app are placeholders.

1. **Optional new request fields** (old apps send nothing; store `null`, shown as "Not recorded"): cancel `{ "reasonCode": "plan_changed|booked_by_mistake|want_reschedule|no_longer_needed|other", "reasonText": "max 200, may be null", "source": "app" }`; `POST customer/order/bulk/edit-schedule` `{ "source": "app", "entry": "order_detail|cancel_sheet" }`.
2. **Cancellation record, for every cancel by anyone** (customer, admin incl. ABA mini-app orders, system):
```text
order_cancellation (cancellation_id PK, order_id FK, bulk_order_id, customer_id, cancelled_at,
  cancelled_by enum(customer|admin|system|partner), channel enum(app|portal|mini_app|api), admin_user_id NULL,
  reason_code NULL (customer values above + admin_customer_request|admin_no_partner|admin_duplicate|admin_other
                    |system_payment_timeout|system_unfulfilled), reason_text NULL,
  order_status_before, payment_status_before, partner_assigned bool, slot_start, lead_minutes int,
  order_total, currency, fee_rule_code NULL, fee_amount NULL, fee_waived bool NULL, fee_waived_by NULL,
  fee_waived_reason NULL, refund_amount NULL, refund_destination enum(original_payment|wallet|none) NULL,
  refund_status enum(none|pending|done|failed) NULL, refund_at NULL, reschedule_count_before int,
  app_instance_id NULL, app_version NULL, platform NULL, service_id, zone_id NULL, request_id UNIQUE)
```
3. **Reschedule record, for every attempt including rejected ones:**
```text
order_reschedule (reschedule_id PK, order_id FK, bulk_order_id, customer_id, requested_at,
  status enum(completed|rejected), failure_code NULL, initiated_by enum(customer|admin|system|partner),
  channel enum(app|portal|mini_app|api), old_slot_start, new_slot_start, shift_minutes int,
  lead_minutes_before_old_slot int, reschedule_no int, is_free bool, fee_rule_code NULL, fee_amount NULL,
  fee_payment_status enum(none|pending|paid|failed) NULL, fee_payment_method NULL, partner_changed bool NULL,
  grace_token_id NULL, app_instance_id NULL, app_version NULL, platform NULL, service_id, zone_id NULL,
  request_id UNIQUE)
cancellation_fee_rule (rule_code PK, applies_to enum(cancel|reschedule), service_id NULL, min_hours_before,
  max_hours_before NULL, fee_type enum(fixed|percent|none), fee_value, vat_included bool, valid_from, valid_to NULL)
ref_cancellation_reason (reason_code PK, label_en, label_km, group enum(customer|admin|system), is_active)
```
`lead_minutes`, `reschedule_no`, `shift_minutes` are stored, not recomputed.
4. **Event log:** write `order_cancelled` (+ `reason_code`, `cancelled_by`, `fee_amount`, `fee_waived`, `refund_amount`, `lead_minutes`), `order_rescheduled` (+ `old_slot`, `new_slot`, `fee_amount`, `is_free`, `reschedule_no`, `initiated_by`), `order_refunded` (`amount`, `destination`, `status`).
5. **Whitelist the client signals:** `cancellation_started|reason_selected|submitted|result|abandoned`, `cancel_to_reschedule_clicked`, `cancellation_policy_viewed`, `reschedule_started|slot_selected|grace_requested|submitted|completed|failed|abandoned`. They never create or change a record. Please reconcile daily: client `cancellation_result=success` vs `order_cancellation` by `order_id`, and expose the mismatch on `GET /meta/data-freshness`. **Confirm** that `order_id` in app events is the `bulkOrderId` and how it maps to your order id.
6. **Fee facts:** tell us where the fee rules live today, who may waive a fee, and when you can ship `GET customer/order/v2/{id}/cancel/preview` and a reschedule fee quote. The app keeps its placeholders and sends no fee event until then.
7. **Endpoints** (filters: period, `service_id`, `zone_id`, `platform`, `app_version`, `source|campaign_id`, `customer_type`, `cancelled_by`, `reason_code`, `lead_bucket` (`gt24h|24h_2h|lt2h|after_start`), `partner_assigned`, `channel`, `mode`):

| Method | Path | Returns |
|---|---|---|
| GET | `/after-purchase/cancellations/summary` | count, rate with denominator, fee kept, fee waived, refund total and pending, avg lead, breakdowns by reason, lead bucket, `cancelled_by`, channel, service, zone, partner assigned, rescheduled-before-cancel, trend |
| GET | `/after-purchase/cancellations` | paged rows, masked customer, export |
| GET | `/after-purchase/cancel-funnel` | signal funnel: started → reason → submitted → success, saved by reschedule, abandoned by step, failed by `error_code` |
| GET | `/after-purchase/reschedules/summary` | count, rate, free vs paid, fee total, avg shift and lead, `reschedule_no` distribution, rejected by `failure_code`, rescheduled-then-cancelled, trend |
| GET | `/after-purchase/reschedules` | paged rows |
| GET | `/orders/{bulkOrderId}/changes` | every reschedule (incl. rejected), cancellation, fee, refund, with actor and channel |
| GET | `/meta/cancellation-reasons`, `/meta/fee-rules` | labels (EN, KM) and active rules |
| GET | `/funnels/acquisition` | fill `afterPurchase.rescheduled` `{orders, amount, detail{done,upcoming,paidFee}}` and `cancelled` `{orders, amount, detail{withFee,free}}`; `unavailableReason` only when truly unknown |

Counting: `mode=byCancelDate` (default: `cancelled_at` in range) or `mode=cohort` (orders placed in range later cancelled); every response states the mode and an explicit denominator `{unit:"orders",count}`.

## 4. Instant Cleaning (spec: `Instant_Cleaning_Tracking_Spec.md`)

New common param `booking_mode` (`instant|scheduled`) on events that carry `service_id`, `journey_id` or `order_id`; `instant_hours` on `checkout_started` and `payment_started` for Instant.

Events: `category_alert_viewed|action`, `instant_availability_checked` (`available_dates`, `total_dates`, `fully_booked`, `trigger`), `instant_fully_booked_viewed`, `instant_duration_selected` (`hours`, `selection_method` = `user|auto_default|auto_rooms|restored`, `was_suggested`), `instant_room_toggled` (`room_key` catalogue id, `selected`, `selected_count`, `total_minutes`, `band`, `suggested_hours`), `instant_underpick_warning_viewed`, `instant_upsell_clicked|result`, `instant_slot_selected` (`slot_start`, `same_day`, `lead_minutes`, `slot_source`, `days_ahead`), `schedule_grace_result` (`result`, `error_code`, `lead_minutes`, `same_day`, `action`), `instant_progress_viewed` (`order_id`, `stage`), `extend_started|payment_selected|closed`.

**Please:** (1) store `booking_mode` on journeys **and orders** (we need it on order records for Instant reporting); (2) tables `instant_availability_check (event_id, occurred_at, customer_id NULL, app_instance_id, service_id, available_dates, total_dates, fully_booked, trigger)`, `instant_room_selection (journey_id, room_key, selected, total_minutes, band, suggested_hours, occurred_at)`, `schedule_grace_error (event_id, occurred_at, service_id, booking_mode, error_code, lead_minutes, same_day, hour_of_day, app_version, platform)`; (3) **server-side, store every rejected grace or schedule request with its code** (authoritative; the app event is the signal); (4) endpoints `GET /instant/overview`, `/instant/rooms`, `/instant/availability`, `/instant/schedule-errors`, `/instant/upsell`, `/instant/orders/progress`, and `booking_mode` filter on `GET /funnels/booking`, orders and after-purchase; (5) confirm booking-mode values, the error-code list (`INSTANT_NOTICE_TOO_SHORT`, `BOOKING_SLOT_UNAVAILABLE`, `BOOKING_LEAD_TIME_REQUIRED`, `BOOKING_GRACE_INVALID`, `BOOKING_POLICY_INVALID`, `INSUFFICIENT_BALANCE`), and whether an extend payment API or record is planned.

## 5. Questions we need answered

1. Which of these can ship **before** the app release? We suggest in this order: accept and store events (1, 2, 4), stage reorder (2), cancel and reschedule records (3), endpoints, fee preview last.
2. A staging date, and one sample response per endpoint (mock-first is fine; the portal builds against the contract).
3. Retention: raw interaction 90 days, errors 180 days, aggregates 2 years. Acceptable? Expected rows per day?
4. Phone retention for people who requested an OTP and never registered (still open from earlier).
5. Is `payment_failed` writable from the payment record with the order or journey id (still open)?
6. Confirm `order_id` meaning (section 3, item 5) and the booking-mode values (section 4).

Reply by adding a dated file in `backend-response/`. We will update the API spec and remove answered items from `pending-request-backend.md`.
