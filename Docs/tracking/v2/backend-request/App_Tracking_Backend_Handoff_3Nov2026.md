# App tracking: what we need from Backend

| | |
|---|---|
| **To** | Backend |
| **From** | bEasy app and portal |
| **Date** | 3 Nov 2026 |
| **Replaces** | the 2 Nov handoff (withdrawn; this file is the only current one) |
| **App code** | branch `feat/tracking-integration` (all slices), **not yet released** |
| **Tracked in** | `pending-request-backend.md` §9 to §12 |
| **Reply** | Add a dated file in `backend-response/`. We update the API spec and remove answered items from `pending-request-backend.md`. |

This is the extract Backend needs to plan, estimate and reply. Full detail and rationale are in the specs in `Docs/tracking/v2/architecture/` (named in each section).

> **The app flow does not change.** The app only sends more data. We ask Backend to store it, map it and expose it.

## Contents

0. [Ground rules](#0-ground-rules)
1. [App usage and health events](#1-app-usage-and-health-events)
2. [Registration funnel, entry and location](#2-registration-funnel-entry-and-location)
3. [Cancellation and reschedule](#3-cancellation-and-reschedule)
4. [Instant Cleaning](#4-instant-cleaning)
5. [Decisions already made, and what the app does not send](#5-decisions-already-made-and-what-the-app-does-not-send)
6. [Questions we need answered](#6-questions-we-need-answered)
7. [Appendix: ids to seed](#7-appendix-ids-to-seed)

## At a glance

| # | Area | What Backend does | App build status |
|---|---|---|---|
| 1 | App usage and health | Store new interaction, screen and error events; add 4 tables and about 10 endpoints | Built, not yet released (home and discovery, booking and checkout, orders, profile and chat) |
| 2 | Registration and location | Reorder the funnel stages; store the entry; add a location table and endpoint | Built, not yet released |
| 3 | Cancel and reschedule | Add cancellation and reschedule records; whitelist client signals; share fee rules | Events built, not yet released; request fields **not sent** until names are agreed |
| 4 | Instant Cleaning | Store `booking_mode` on journeys and orders; add 3 tables and endpoints | Built, not yet released |

---

## 0. Ground rules

These apply to everything below.

| # | Rule |
|---|---|
| 1 | **Transport stays Firebase → BigQuery → your importer.** The app does not call a new API to send events. |
| 2 | **Every event carries** `event_id` (UUIDv4, the dedupe key), `schema_version` and `seq`, plus `service_id`, `journey_id`, `reg_flow_id` or `order_id` where relevant. Quarantine only on a bad `event_id`, `schema_version` or `seq`. |
| 3 | **Unknown values are stored and flagged, never rejected.** New screen or element ids will appear with app releases. |
| 4 | **Guests** are keyed by `app_instance_id` and merged to `customer_id` at identity link. Keep guest events. |
| 5 | **Money and order facts are yours.** Client events about payment, cancel and reschedule are signals only; your records are the truth. `null` means unknown; never return 0 for unknown. |
| 6 | **No personal data** in event params: no phone, name, address text, free text or exact coordinates. Location is `area_cell` (about 1 km grid) and `zone_id` when known. |
| 7 | **Exclude** staff phones and test orders, as today. |
| 8 | **Time** is stored in UTC and returned as ISO-8601 with `+07:00`. **Money** is USD with two decimals; state whether VAT is included. |
| 9 | Interaction events (sections 1 and 4) go to **separate tables**, not `event_log`, because of volume. A listed subset also goes to `event_log` so the Customer page timeline shows it. |

---

## 1. App usage and health events

Spec: `App_Tracking_Coverage_Spec.md` §3, §6.

### 1.1 New client events

| Group | Events |
|---|---|
| Screens and overlays | `screen_viewed` (extended), `screen_exited`, `overlay_viewed` |
| Interactions | `element_clicked`, `form_submitted`, `search_performed`, `search_result_clicked`, `setting_changed`, `permission_result`, `support_contacted` |
| Errors | `app_error` |
| Cancellation and extend | `cancellation_*`, `extend_*` (see sections 3 and 4) |
| Top-up | `topup_started`, `topup_amount_selected`, `topup_payment_started` |
| Coupons | `coupon_list_viewed` |
| Address | `address_added`, `address_edited`, `address_deleted` (see section 2) |
| Chat | `chat_opened`, `chat_message_sent` |

These events from the earlier plan are covered by **`element_clicked`** with a registered `element_id`, not by a dedicated event name:

| Earlier plan | What the app sends today |
|---|---|
| `logout_clicked` | `element_clicked` `menu_logout` (Profile menu row), then `element_clicked` `logout_confirm` (confirm dialog) |
| `coupon_copied` | `element_clicked` `copy_promo_code` (See more coupon banner, `section=coupon_banner`) |
| `coupon_use_clicked` | `element_clicked` `coupon_card` (checkout coupon list, `item_id` = coupon id) |
| `content_shared` | `element_clicked` `menu_share_app` (Profile menu row; it records the tap, not whether a share completed) |

**`address_default_set` is genuinely not sent**: the set-default feature is disabled in the app, so there is nothing to track. If it is switched on later, we add the event then. Backend does not need to build for it yet.

**Stop skipping** `screen_viewed`. Keep skipping the legacy ad-hoc events until we retire them (list in the spec §3.5).

### 1.2 Key parameters

| Parameter | Values or notes |
|---|---|
| `screen_name`, `screen_group`, `previous_screen` | `snake_case` ids, 40 characters at most |
| `entry_type` | `nav`, `tab`, `push`, `deeplink`, `back` |
| `duration_seconds` | Time on a screen |
| `overlay_name`, `overlay_type` | Sheets, dialogs and popups |
| `element_id`, `element_type`, `section`, `position`, `item_id`, `target` | Taps. `item_id` is an entity id, never a label |
| `form_id`, `result`, `error_code` | Forms and results |
| `error_type` | `api`, `payment`, `webview`, `network`, `update`, `exception` |
| `endpoint_group`, `http_status`, `fatal` | Errors |

Labels are not sent. Khmer and English labels live in the registries (section 1.5).

### 1.3 Event shapes worth knowing

| Event | Shape |
|---|---|
| `setting_changed` (language) | `setting_id=language`; `setting_value` is one of `en`, `km`, `vi`, `zh-cn`, `zh-tw`, `other`; `previous_value`; `source` is one of `home_sheet`, `profile`, `default_fallback`, `other` |
| `chat_opened` | `chat_kind` is `order` or `support`; `entry` |
| `chat_message_sent` | `message_type` is `text`, `image` or `video`. Never the message text |
| `permission_result` | `context` also has the value `auto_open` |
| `form_submitted` with `form_id=review` | `result`, `tags_count`, `has_comment` (0 or 1), `error_code` (`http_<n>` or `network`). Never text |
| Order detail customer-service buttons | `support_contacted` with `channel=chat` and `order_id` |
| `coupon_list_viewed` | Comes from the checkout coupon list |

The app does **not** send `topup_completed` or `topup_failed` (see section 1.8).

### 1.4 Payment and booking notes

- The payment result overlay is named `payment_result_success` or `payment_result_failed`.
- The unavailable-schedule sheet is informational. It only sends `overlay_viewed` with `unavailable_schedule`.
- An `error_code` example: `no_slots`.

### 1.5 Registries to seed

Registry rows (`ref_app_screen`, `ref_app_element`) come from the app's id lists. The full list of ids is in the spec §4 A/C/G shipped. Labels (EN and KM) are provided by us.

| Area | App source | Slice | `endpoint_group` values |
|---|---|---|---|
| Launch, home, banners, notifications, tabs | `DiscoveryIds.all` (elements), `DiscoveryIds.allOverlays` (overlays) in `lib/analytics/discovery_tracking.dart` | Home and discovery | `home_page`, `user_info`, `no_internet`, `app_version` |
| Profile, help, support sheets, chat | `ProfileIds.all`, `ProfileIds.allOverlays` in `lib/analytics/profile_tracking.dart` | Profile and chat | none listed |
| Booking, checkout, payment, top-up | `BookingIds.all`, `BookingIds.allOverlays` in `lib/analytics/trackers/booking_tracking.dart` | Booking and checkout | `service_options`, `order_preview`, `coupon_apply`, `order_schedule`, `payment_methods`, `order_create`, `payment_status`, `payment_webview`, `topup_amounts`, `topup_create`, `topup_webview`, `topup_status` |
| Orders | `OrdersIds.all`, `OrdersIds.allOverlays` in `lib/analytics/trackers/orders_tracking.dart` | Orders | `booking_list`, `order_detail`, `order_complete`, `review_options` |
| Location and address | `AnalyticsHelper` calls in `lib/controller/location_controller.dart` | Location | `places`, `address`, `coverage` |

### 1.6 Tables

Three-normal-form additions to your model.

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

### 1.7 Endpoints

Filters: `period`, `from`, `to`, `platform`, `app_version`, `language`, `auth_state`, `source`, `campaign_id`, `service_id`, `audience`, `screen_group`. Identifiers are masked. Use the existing pagination.

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

### 1.8 Definitions we will display, and one backend-owned event

| Metric | Definition |
|---|---|
| Views | Count of `screen_viewed` |
| Unique viewers | Distinct customer (or device when guest) |
| Avg time | Mean `duration_seconds`, excluding values over 1800 |
| Exit rate | Sessions whose last screen is this one ÷ sessions that viewed it |
| CTR | Clickers ÷ unique viewers of that screen |
| Error rate | `app_error` ÷ views |

Range-level uniques must be computed from raw rows, not summed from daily rows.

**Backend-owned, please add:** `topup_completed` and `topup_failed`, written from the wallet record (as `payment_failed` is written from the payment record). The app never sends them.

---

## 2. Registration funnel, entry and location

Spec: `Registration_And_Location_Flow_Spec.md`.

### 2.1 What the app does today

> Since 27 Oct, for a new number: the user enters the phone, the OTP is sent, the user **types the code**, then fills the sign-up form, and only **then** is the code verified together with account creation. So `otp_verified` fires **after** `signup_viewed`.
>
> For an existing number the OTP is verified at once (login). **We are not changing this.**

### 2.2 Stage order (please change)

```text
registration_viewed → otp_requested → signup_started → otp_verified → registration_completed
```

- Read `otp_verified` from the event, not inferred from completion.
- Count each stage as distinct `reg_flow_id` (device while a guest).
- `otp_viewed` and `signup_viewed` can repeat inside one flow (second OTP after a rejected held code).

### 2.3 New and changed events

| Event | Params | Notes |
|---|---|---|
| `registration_started` (extended) | `entry`, `entry_detail`, `source_screen`, `campaign_code` | |
| `otp_viewed` (extended) | `flow_type` (`register` or `login`), `otp_mode` (`deferred` or `immediate`), `attempt_round` | |
| `otp_code_entered` (new, diagnostic) | `flow_type`, `otp_mode` | |
| `otp_submitted`, `otp_failed` (kept, extended) | `otp_mode` | The held code now fires both, with `otp_mode=deferred`. `error_code` stays today's values: `network`, `invalid_or_expired`, `http_<n>`, `invalid_response`, `exception`, `create_rejected`. `wrong_code`, `blocked` and `expired` are **not sent** until you confirm what you can distinguish |
| `signup_submitted` (new) | `result` (`success`, `error`, `invalid`), `error_code`, `has_email`, `has_referral` | Fires when the create call returns, before the held code is verified. `invalid` is not sent today |
| `account_created` (new) | `method`, `entry` | Firebase `sign_up` and Meta `CompleteRegistration` fire at this moment today, unchanged |
| `registration_completed` (extended) | `method`, `entry` | |
| `registration_resumed` (new) | `entry`, `result` (`back_to_checkout` or `home`) | **Not in the app yet** |

### 2.4 Entry values

`entry` is a closed set:

`home`, `tab_gate`, `checkout_address`, `checkout_schedule`, `checkout_summary`, `banner`, `push`, `deeplink`, `dialog`, `onboarding`, `other`

- The app does **not** send `checkout_summary` today. The summary prompt shares the address handler and reports `checkout_address`.
- `entry_detail` is a short code (tab name, banner id, push type).
- Old values map as: `gate` → `other`, `checkout_gate` → `checkout_address`.

### 2.5 Registration table and API

```text
registration_attempt (reg_flow_id PK, app_instance_id, customer_id NULL, entry, entry_detail NULL,
  source_screen NULL, started_at, otp_requested_at NULL, code_entered_at NULL, signup_viewed_at NULL,
  signup_submitted_at NULL, otp_verified_at NULL, completed_at NULL, otp_mode, otp_rounds int,
  last_error_code NULL, method NULL, campaign_code NULL)
```

| Endpoint | Change |
|---|---|
| `GET /funnels/registration` | Accept `entry` and `group_by=entry`; return `entryBreakdown[]` with `{entry, started, completed, conversion}` |
| Stage people rows, customer activity registration group | Include `entry` |
| `GET /attribution/registration-entries` (new) | Entry → registered → first purchase within 7 and 30 days |

### 2.6 Location events

| Event | Params | Notes |
|---|---|---|
| `service_area_checked` | `source`, `result` (`ok` or `error`), `in_range`, `area_cell`, `address_id`, `duration_ms`, `zone_id` | `in_range` only when `result=ok`; `zone_id` only if present. A failed check has no `in_range` and **must not count as out of range**. A 20 s UI timeout arrives as `app_error` with `endpoint_group=coverage` and `error_code=timeout`, not as `service_area_checked`. The check keeps running, so a late `service_area_checked` with `duration_ms` over 20000 can still arrive |
| `address_added`, `address_edited` | `source`, `result` (`success` or `error`), `method`, `has_floor`, `has_room`, `has_note`, `area_cell`, `error_code` | No `address_id` or `in_range` |
| `address_deleted` | `source`, `result` | `address_default_set` is **not sent**: the feature is disabled in the app |
| `permission_result` (location) | `result` (`granted`, `denied`, `denied_forever`, `services_disabled`), `context` (`use_current_location` or `auto_open`) | `granted` only after the OS prompt was shown; the other results on every attempt |
| `location_pin_moved` | `method` (`drag`, `tap`, `current_location`, `search`), `area_cell` | |
| `address_selected` (extended) | `source=checkout`, `method` (`default_auto`, `saved_list`, `new_address`, `existing`) | |

```text
app_service_area_check (event_id PK, occurred_at, customer_id NULL, app_instance_id, source, area_cell,
  zone_id NULL, address_id NULL, in_range bool NULL, result, duration_ms)
```

`GET /app-usage/locations` returns:

```text
{ summary{checks, outOfRange, outOfRangeRate, permissionDenied},
  outOfRangeAreas[{areaCell, lat, lng (cell centre), checks, uniqueUsers}],
  methodBreakdown[], errors[] }
```

---

## 3. Cancellation and reschedule

Spec: `Cancellation_And_Reschedule_Spec.md`.

> **Today:** `POST customer/order/v2/{id}/cancel` is sent with an empty body, so the reason never reaches you. You store no reason, fee, refund or reschedule record (`rescheduled` shows "not tracked" in the portal). The fee panels in the app are placeholders.
>
> The client events in 3.5 are built in the app (not yet released). The new request fields in 3.1 are **not sent** until you confirm the names.

### 3.1 Optional new request fields

Old apps send nothing; store `null`, shown as "Not recorded".

| Request | New field |
|---|---|
| Cancel | `{ "reasonCode": "plan_changed" or "booked_by_mistake" or "want_reschedule" or "no_longer_needed" or "other", "reasonText": "max 200, may be null", "source": "app" }` |
| `POST customer/order/bulk/edit-schedule` | `{ "source": "app", "entry": "order_detail" or "cancel_sheet" }` |

### 3.2 Cancellation record

For every cancel by anyone: customer, admin (including ABA mini-app orders), system.

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

### 3.3 Reschedule record and rule tables

For every attempt, including rejected ones.

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

`lead_minutes`, `reschedule_no` and `shift_minutes` are stored, not recomputed.

### 3.4 Event log

| Event | Extra fields |
|---|---|
| `order_cancelled` | `reason_code`, `cancelled_by`, `fee_amount`, `fee_waived`, `refund_amount`, `lead_minutes` |
| `order_rescheduled` | `old_slot`, `new_slot`, `fee_amount`, `is_free`, `reschedule_no`, `initiated_by` |
| `order_refunded` | `amount`, `destination`, `status` |

### 3.5 Client signals to whitelist

- **Cancel:** `cancellation_started`, `cancellation_reason_selected`, `cancellation_submitted`, `cancellation_result`, `cancellation_abandoned`, `cancel_to_reschedule_clicked`, `cancellation_policy_viewed`
- **Reschedule:** `reschedule_started`, `reschedule_slot_selected`, `reschedule_grace_requested`, `reschedule_submitted`, `reschedule_completed`, `reschedule_failed`, `reschedule_abandoned`

These never create or change a record.

- **Please reconcile daily:** client `cancellation_result=success` against `order_cancellation` by `order_id`, and expose the mismatch on `GET /meta/data-freshness`.
- **Please confirm** that `order_id` in app events is the `bulkOrderId`, and how it maps to your order id.

### 3.6 Fee facts

Tell us:

- where the fee rules live today
- who may waive a fee
- when you can ship `GET customer/order/v2/{id}/cancel/preview` and a reschedule fee quote

Until then the app keeps its placeholders and sends **no fee event** (`cancellation_fee_shown`, `reschedule_fee_shown`). The events also carry no `service_id`, `reschedule_count`, `is_free` or `fee_amount`. Slots are ISO-8601 with `+07:00`.

### 3.7 Endpoints

Filters: period, `service_id`, `zone_id`, `platform`, `app_version`, `source` or `campaign_id`, `customer_type`, `cancelled_by`, `reason_code`, `lead_bucket` (`gt24h`, `24h_2h`, `lt2h`, `after_start`), `partner_assigned`, `channel`, `mode`.

| Method | Path | Returns |
|---|---|---|
| GET | `/after-purchase/cancellations/summary` | Count, rate with denominator, fee kept, fee waived, refund total and pending, avg lead; breakdowns by reason, lead bucket, `cancelled_by`, channel, service, zone, partner assigned, rescheduled-before-cancel; trend |
| GET | `/after-purchase/cancellations` | Paged rows, masked customer, export |
| GET | `/after-purchase/cancel-funnel` | Signal funnel: started → reason → submitted → success; saved by reschedule; abandoned by step; failed by `error_code` |
| GET | `/after-purchase/reschedules/summary` | Count, rate, free vs paid, fee total, avg shift and lead, `reschedule_no` distribution, rejected by `failure_code`, rescheduled-then-cancelled, trend |
| GET | `/after-purchase/reschedules` | Paged rows |
| GET | `/orders/{bulkOrderId}/changes` | Every reschedule (including rejected), cancellation, fee and refund, with actor and channel |
| GET | `/meta/cancellation-reasons`, `/meta/fee-rules` | Labels (EN, KM) and active rules |
| GET | `/funnels/acquisition` | Fill `afterPurchase.rescheduled` `{orders, amount, detail{done, upcoming, paidFee}}` and `cancelled` `{orders, amount, detail{withFee, free}}`; `unavailableReason` only when truly unknown |

**Counting:** `mode=byCancelDate` (default; `cancelled_at` in range) or `mode=cohort` (orders placed in range that were later cancelled). Every response states the mode and an explicit denominator `{unit: "orders", count}`.

---

## 4. Instant Cleaning

Spec: `Instant_Cleaning_Tracking_Spec.md`.

### 4.1 Common parameters

| Param | Notes |
|---|---|
| `booking_mode` (`instant` or `scheduled`) | On events that carry `service_id`, `journey_id` or `order_id`. **Absent** (not `scheduled`) when the app does not know the mode yet, and on events with no `service_id` (order-only events) |
| `instant_hours` | On `checkout_started` and `payment_started`, for Instant services only, from the last `instant_duration_selected` |

### 4.2 Events

| Event | Params | Notes |
|---|---|---|
| `category_alert_viewed`, `category_alert_action` | | |
| `instant_availability_checked` | `available_dates`, `total_dates`, `fully_booked`, `trigger` (`load` or `refresh`) | |
| `instant_fully_booked_viewed` | | |
| `instant_duration_selected` | `hours`, `selection_method` (`user`, `auto_default`, `restored`), `was_suggested` | `auto_rooms` is reserved and never sent |
| `instant_room_toggled` | `room_key` (catalogue id), `selected`, `selected_count`, `total_minutes`, `band`, `suggested_hours` | |
| `instant_underpick_warning_viewed` | | |
| `instant_upsell_clicked`, `instant_upsell_result` | `result` carries `returned=1` only | No `converted`: read conversion from `checkout_started` of the target service |
| `instant_slot_selected` | `slot_start`, `same_day`, `lead_minutes`, `slot_source` (`selector` only), `days_ahead` | From the time selector only |
| `schedule_grace_result` | `result`, `error_code`, `lead_minutes`, `same_day`, `action` | **No `service_id` and no `booking_mode`**: join by journey or session, not service |
| `instant_progress_viewed` | `stage` | **No `order_id` yet** |
| `extend_started`, `extend_payment_selected`, `extend_closed` | `stage`, `extra_hours`, `payment_method` | No price and no `order_id` |

### 4.3 What we ask

1. Store `booking_mode` on journeys **and orders** (we need it on order records for Instant reporting).
2. Add these tables:
   ```text
   instant_availability_check (event_id, occurred_at, customer_id NULL, app_instance_id, service_id,
     available_dates, total_dates, fully_booked, trigger)
   instant_room_selection (journey_id, room_key, selected, total_minutes, band, suggested_hours, occurred_at)
   schedule_grace_error (event_id, occurred_at, service_id, booking_mode, error_code, lead_minutes,
     same_day, hour_of_day, app_version, platform)
   ```
3. **Server-side, store every rejected grace or schedule request with its code.** This is authoritative; the app event is the signal.
4. Add endpoints `GET /instant/overview`, `/instant/rooms`, `/instant/availability`, `/instant/schedule-errors`, `/instant/upsell`, `/instant/orders/progress`, and a `booking_mode` filter on `GET /funnels/booking`, orders and after-purchase.
5. Confirm:
   - the booking-mode values
   - the error-code list: `INSTANT_NOTICE_TOO_SHORT`, `BOOKING_SLOT_UNAVAILABLE`, `BOOKING_LEAD_TIME_REQUIRED`, `BOOKING_GRACE_INVALID`, `BOOKING_POLICY_INVALID`, `INSUFFICIENT_BALANCE`
   - whether an extend payment API or record is planned

---

## 5. Decisions already made, and what the app does not send

### 5.1 Decided by the owner (not open for change)

| # | Decision |
|---|---|
| 1 | **The app flow does not change.** Only data collection changes. |
| 2 | **The portal follows the app.** For example, the registration funnel order follows what the app really does (section 2.2). |
| 3 | The three `checkout_*` registration entries are shown as one "Checkout" bar with a drill-down. |
| 4 | Meta `CompleteRegistration` and Firebase `sign_up` timing stay at account creation. `af_purchase` and Meta `Purchase` timing are unchanged. |
| 5 | The new Meta and AppsFlyer events are sent from release builds only and never for staff phones. These are SDK events; Backend is not involved. |
| 6 | Cancel and reschedule **fee panels stay placeholders** in the app until Backend ships a fee preview. |
| 7 | One legacy event, `customer_source_continue`, is left unchanged. It puts typed text in a parameter name, so we do not rely on it. We flag it for the release plan. |

### 5.2 What the app does not send today

| Not sent | Why | App default |
|---|---|---|
| `wrong_code`, `blocked`, `expired` as OTP `error_code` | Backend has not said what it can distinguish | App keeps today's codes (`network`, `invalid_or_expired`, `http_<n>`, ...) |
| `registration_resumed`, `signup_submitted` with `result=invalid` | Not built yet | none |
| `checkout_summary` as a registration entry | The summary prompt shares the address handler | Sent as `checkout_address` |
| Cancel and reschedule request fields (`reasonCode`, `reasonText`, `source`, `entry`) | Names not agreed yet | Request body stays empty |
| `cancellation_fee_shown`, `reschedule_fee_shown`, `fee_amount`, `is_free` | Fees are placeholders | none |
| `service_id`, `reschedule_count` on reschedule events | No reliable value in the booking detail | none |
| `converted` on `instant_upsell_result` | Needs state shared with another screen | Read conversion from `checkout_started` of the target service |
| `order_id` on `instant_progress_viewed`, `extend_*` | The widgets do not know it | none |
| `topup_completed`, `topup_failed` | **Backend-owned** (section 1.8) | none |
| `address_default_set` | The feature is disabled in the app | none |
| Dedicated events `coupon_copied`, `coupon_use_clicked`, `logout_clicked`, `content_shared` | Covered by `element_clicked` ids (section 1.1) | Use `element_clicked` |
| Chat `app_error` (connection and upload failures), `has_ordered` | Planned for a later release | none |
| Search text for places and addresses | Privacy | `search_performed` has `result_count` only |

---

## 6. Questions we need answered

Each question has the default the app uses if no answer comes.

| # | Question | Default if no answer |
|---|---|---|
| 1 | Which of these can ship **before** the app release? We suggest: accept and store events (sections 1, 2 and 4), stage reorder (section 2), cancel and reschedule records (section 3), endpoints, fee preview last. | App ships; events are stored when Backend is ready |
| 2 | A staging date, and one sample response per endpoint. Mock-first is fine; the portal builds against the contract. | Portal builds against the contract in this file |
| 3 | Retention: raw interaction 90 days, errors 180 days, aggregates 2 years. Acceptable? Expected rows per day? | As proposed |
| 4 | Phone retention for people who requested an OTP and never registered (still open from earlier). | none |
| 5 | Is `payment_failed` writable from the payment record with the order or journey id? (still open) | none |
| 6 | Confirm the `order_id` meaning (section 3.5) and the booking-mode values (section 4). | `order_id` = `bulkOrderId` |
| 7 | Can you tell a wrong OTP code from an expired one, and how? | Keep today's `error_code` values |
| 8 | Which names will the optional cancel and reschedule request fields use (section 3.1)? | Not sent |
| 9 | Event and id whitelist: do unknown events or ids go to quarantine, or are they stored and flagged as section 0 asks? | Stored and flagged |
| 10 | Reschedule events now send slots as ISO-8601 with `+07:00`; older app versions sent display strings. Can the importer accept both? | App sends ISO |
| 11 | `form_submitted` carries `tags_count` and `has_comment`; `setting_changed` carries the language codes in section 1.3. Will the importer keep these extra params? | Stored in `metadata jsonb` |
| 12 | `schedule_grace_result` has no `service_id` or `booking_mode`. Which join do you prefer: journey, session or time window? | Join by session |
| 13 | Join keys: confirm `reg_flow_id`, `journey_id` and `app_instance_id` are available in the BigQuery export as top-level or param fields. | As in section 0 |
| 14 | When can the app expect `GET customer/order/v2/{id}/cancel/preview` and a reschedule fee quote? | Placeholders stay |

Reply by adding a dated file in `backend-response/`. We will update the API spec and remove answered items from `pending-request-backend.md`.

---

## 7. Appendix: ids to seed

Generated from the app code on `feat/tracking-integration`. Ids are `snake_case`, 40 characters at most. English and Khmer labels are provided by us. An id missing from a registry is still stored and flagged `unregistered` (section 0, rule 3). `element_id` values are unique per screen; overlay names are kept apart from element ids because an overlay can share a name with the element that opens it.

### DiscoveryIds (launch, home, banners, notifications, tabs)

**`element_id` values** (55): `splash_ad_cta`, `splash_ad_skip`, `update_now`, `update_later`, `retry_connection`, `retry_server`, `language_picker`, `promo_fab`, `promo_popup`, `promo_close`, `bcombo_card`, `sign_in_card`, `refresh_home`, `service_card`, `see_all_services`, `retry_services`, `banner_card`, `latest_promo_card`, `turn_on_notifications`, `join_telegram`, `telegram_float_button`, `close_telegram_float`, `contact_support`, `copy_promo_code`, `tab_home`, `tab_bookings`, `tab_notifications`, `tab_chat`, `tab_profile`, `tab_locked_guest`, `banner_cta`, `banner_image_zoom`, `banner_coupon_select`, `banner_terms_check`, `banner_payment_method`, `banner_sheet_close`, `receipt_upload`, `receipt_submit`, `receipt_take_photo`, `receipt_choose_album`, `receipt_cancel`, `guest_sign_in_continue`, `notification_card`, `announcement_card`, `mark_all_read`, `mark_all_read_confirm`, `mark_all_read_cancel`, `tab_notifications_list`, `tab_announcements`, `refresh_notifications`, `refresh_announcements`, `notif_banner_not_now`, `notif_banner_open_settings`, `announcement_close`, `announcement_cta`

**`overlay_name` values** (12): `language_picker`, `promo_popup`, `lucky_draw`, `coming_soon`, `guest_sign_in_prompt`, `banner_payment_methods`, `banner_aba_transfer`, `receipt_source`, `mark_all_read`, `announcement_detail`, `force_update`, `optional_update`

### ProfileIds (profile, help, support sheets, chat)

**`element_id` values** (64): `menu_profile_info`, `menu_my_location`, `menu_my_coupons`, `menu_language`, `menu_share_app`, `menu_help_center`, `menu_contact_support`, `menu_about`, `menu_logout`, `logout_confirm`, `language_option`, `help_faq`, `help_terms`, `help_privacy`, `help_email`, `about_website`, `about_facebook`, `about_instagram`, `about_tiktok`, `about_telegram`, `about_rednote`, `contact_support`, `contact_telegram`, `contact_phone`, `contact_email`, `change_avatar`, `avatar_take_photo`, `avatar_choose_album`, `avatar_save_image`, `gender_field`, `gender_option`, `dob_field`, `dob_done`, `dob_cancel`, `save_profile`, `delete_account`, `delete_account_confirm`, `refresh_coupons`, `tutorial_category`, `tutorial_card`, `tutorial_play`, `tutorial_toggle_play`, `tutorial_fullscreen`, `tutorial_related`, `tutorial_helpful_yes`, `tutorial_helpful_no`, `skip_customer_source`, `source_option`, `customer_source_continue`, `chat_tile`, `refresh_chats`, `place_order`, `chat_back`, `send_message`, `attach_camera`, `story_shutter`, `story_switch_camera`, `story_gallery`, `story_close`, `media_retake`, `media_send`, `media_back`, `media_open`, `gallery_pick`

**`overlay_name` values** (7): `contact_support`, `logout_confirm`, `avatar_source`, `gender_picker`, `dob_picker`, `delete_account_confirm`, `story_gallery`

### BookingIds (booking, checkout, payment, top-up)

**`element_id` values** (39): `back`, `contact_support`, `retry`, `schedule_appointment`, `add_to_booking`, `showcase_task_info`, `equipment_included`, `addon_done`, `sheet_close`, `sub_category_selected`, `show_all_options`, `option_info`, `start_booking`, `refresh_price`, `change_schedule`, `read_full_policy`, `apply_coupon`, `use_other_promo`, `confirm_pay`, `sign_up_gate`, `cross_sell_next`, `select_payment_method`, `pair_sheet_cancel`, `pair_sheet_confirm`, `schedule_confirm`, `schedule_close`, `coupon_card`, `search_coupon`, `back_home`, `try_again`, `result_close`, `topup_amount`, `topup_confirm`, `topup_history_item`, `topup_status_done`, `aba_transfer_submit`, `aba_transfer_close`, `topup_from_balance`, `topup_new`

**`overlay_name` values** (15): `schedule_sheet`, `payment_sheet`, `payment_result_success`, `payment_result_failed`, `cross_sell_sheet`, `pair_service_sheet`, `task_info`, `equipment_included`, `service_addons`, `unavailable_schedule`, `topup_payment_sheet`, `topup_aba_transfer`, `topup_detail`, `topup_low_balance`, `sub_category_detail`

**`endpoint_group` values** (12): `order_preview`, `coupon_apply`, `order_create`, `order_schedule`, `payment_methods`, `service_options`, `payment_status`, `payment_webview`, `topup_create`, `topup_amounts`, `topup_webview`, `topup_status`

### OrdersIds (bookings, order detail, extend, review)

**`element_id` values** (33): `tab_active`, `tab_history`, `order_card`, `explore_services`, `refresh_bookings`, `back`, `contact_support`, `contact_us`, `pay_now`, `book_again`, `leave_review`, `swipe_complete`, `cleaners_not_yet`, `cleaners_done`, `service_add_on`, `refresh_order`, `gift_apply`, `gift_cancel`, `extend_hours`, `extend_close`, `extend_confirm`, `extend_done`, `addon_service_item`, `addon_add`, `addon_increase`, `addon_decrease`, `addon_checkout`, `addon_close`, `review_star`, `review_tag`, `review_submit`, `review_done_close_icon`, `review_done_close`

**`overlay_name` values** (6): `order_payment_sheet`, `cleaners_not_finished`, `addon_services`, `order_addon`, `gift_coupon_confirm`, `extend_hours`

**`endpoint_group` values** (4): `booking_list`, `order_detail`, `order_complete`, `review_options`

### Screens (`ref_app_screen`)

55 screens. `screen_name` → `screen_group`:

| Group | Screens |
|---|---|
| `launch` | `splash`, `splash_ad` |
| `system` | `no_internet`, `server_error`, `force_update`, `webview` |
| `auth` | `welcome`, `sign_in`, `sign_up`, `otp`, `set_password`, `customer_source`, `notification_prompt` |
| `home` | `home`, `home_search`, `see_more`, `banner_detail` |
| `booking` | `service_detail`, `service_addons` |
| `checkout` | `checkout`, `coupon_list`, `schedule_sheet`, `payment_sheet`, `payment_result`, `appointment_status` |
| `orders` | `bookings`, `order_detail`, `order_cancel`, `order_reschedule`, `order_extend`, `leave_review`, `review_done` |
| `notifications` | `notifications`, `notification_detail` |
| `chat` | `chat_list`, `chat_room`, `chat_story_create` |
| `profile` | `profile`, `profile_info`, `change_password`, `change_avatar`, `locations`, `location_edit`, `location_search`, `language`, `payment_methods`, `my_coupons`, `help_center`, `tutorial_center`, `tutorial_detail`, `about` |
| `topup` | `topup`, `topup_history`, `topup_status` |
| `other` | `unnamed_page` |
