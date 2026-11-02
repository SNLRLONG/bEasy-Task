# bEasy App — Tracking Coverage Audit and Specification

Updated: 2026-11-02 · Status: **draft for review** (no code changed) · Plan: [`Tasks/_plans/Cross-tracking-coverage.md`](../../../../Tasks/_plans/Cross-tracking-coverage.md)

Extends [bEasy-tracking-architecture-guide.md](bEasy-tracking-architecture-guide.md) (journey funnel, `event_log`, portal Customer Journey). That guide covers the **booking and registration funnel**. This spec covers what it does not: **every other screen and tap**, app health, and what Meta and AppsFlyer receive. Where the two disagree, the architecture guide wins for funnel events and this spec wins for the new interaction events.

Registration order, registration entry and the location flow are specified in [Registration_And_Location_Flow_Spec.md](Registration_And_Location_Flow_Spec.md), which takes precedence over §4 B and §4 H for those screens. Instant Cleaning is specified in [Instant_Cleaning_Tracking_Spec.md](Instant_Cleaning_Tracking_Spec.md), which takes precedence over §4 D/E for `bookingMode == 'INSTANT'`. Cancellation and reschedule are specified in [Cancellation_And_Reschedule_Spec.md](Cancellation_And_Reschedule_Spec.md), which takes precedence over §3.4 and §4 F for those flows.

Evidence base: `bEasy-app/lib` at `main` on 2 Nov 2026. Per-file counts: [App_Interaction_Inventory.md](App_Interaction_Inventory.md). The backend repo (`bEasy-api/`) is **not cloned**, so every statement about backend behaviour comes from `backend-request/pending-request-backend.md` and `backend-response/`, not from code (marked *doc-only*).

---

## 1. Audit summary

### 1.1 What is tracked today

| Platform | What it receives | Where in code |
|---|---|---|
| **Firebase Analytics** | ~55 contract/helper events (journey, booking, promo, checkout, registration/OTP, push, banner, reschedule) plus ~35 ad-hoc click/screen events; `screen_viewed` from a navigator observer; user props `auth_state`, `first_campaign`, `utm_*` | `lib/analytics/analytics_helper.dart` (single choke point `AnalyticsHelper.track`), `screen_observer.dart`, `trackers/*` |
| **Meta App Events** | 7: `ViewContent`, `SelectService` (custom), `InitiateCheckout`, `AddPaymentInfo`, `ViewBooking` (custom), `CompleteRegistration`, `Purchase`. Gated by ATT | `lib/app_tracking/facebook_helper.dart` |
| **AppsFlyer** | Install/deep-link attribution, `app_opened` link touch, `af_purchase` only | `lib/app_tracking/appflyer_helper.dart` |
| **Microsoft Clarity** | Session replay (non-staff) — no custom tags | `Helper.maybeInitClarity` |
| **FCM / In-App Messaging** | Push + inbox; `push_received`, `push_opened`, `notification_*_viewed` | `util/firebase_messaging_service.dart`, `local_notifications_service.dart` |
| **Crash / performance / error monitoring** | **None** (no Crashlytics, no Performance, no API-error event) | — |

Staff phones and non-release builds are excluded in `_sendEnabled` (`Helper.kMode && !isStaff`). Confirm Meta and AppsFlyer apply the same exclusion (G-14).

### 1.2 Coverage numbers

137 view files have interaction handlers or tracking calls; **554 handlers, 105 tracking calls**. Tracking is concentrated in Auth, Home, Orders v2 and the checkout CTA. Zero tracking calls: **Chat (50 handlers), Notifications list (12), Web views (7)**, and nearly all of Profile (99 handlers, 7 calls) and Shared widgets (95, 8). Full table: [App_Interaction_Inventory.md](App_Interaction_Inventory.md).

### 1.3 Findings

| # | Finding | Evidence | Impact |
|---|---|---|---|
| G-1 | `screen_viewed` only fires for `PageRoute`s. Bottom tabs (inside `app_screen.dart`), bottom sheets, dialogs, popups and web-view content are invisible | `screen_observer.dart:11` `if (route is! PageRoute) return` | Cancel/reschedule/extend sheets, payment sheet, coupon sheet, contact sheet never counted as views |
| G-2 | Backend **skips** `screen_viewed`, `banner_clicked`, `click_promote`, `app_backgrounded/foregrounded`, `review_submitted`, `order_rescheduled`, all splash events | `pending-request-backend.md` line 22 (doc-only) | These reach Firebase/BigQuery but never the portal |
| G-3 | Two naming systems. Contract events (`service_viewed`, params with IDs) vs ad-hoc events whose parameter equals the event name (`contact_us_telegram` = `{contact_us_telegram: 1}`, `home_screen`, `sign_up_screen`) | `analytics_helper.dart:294-310`; 8 call sites | No `screen`/`element`/`source` dimension; cannot answer "which Telegram button" |
| G-4 | `contact_us_telegram` fired from 8 places with no source; `notification_banner_not_now` / `turn_on_notifications_click` fired from 5 screens with no screen | grep of `view/`, `util/`, `helper/` | One number for different surfaces |
| G-5 | `sign_up_screen` fires from three different screens (`signup_screen`, `welcome_screen`, `new_signup_screen`) | `view/screen/account/*` | Ambiguous; mixes legacy and new auth |
| G-6 | Orders: v2 detail has 10 tracking calls but the **cancel sheet (13 handlers), reschedule sheet (13), extend sheet (4), services card, bottom actions** have none. `cancellation_started`, `cancellation_fee_shown` (matrix N5) not implemented | `order_detail/widgets/*` | Cancellation reasons and fee impact invisible on the client side |
| G-7 | Search untracked: `home_service_search_screen.dart` has input + results, no `search_performed` / `search_result_clicked` (both in architecture §11.1 as "new") | file has 3 handlers, 0 calls | No search-term demand data |
| G-8 | Profile menu: 13 rows, 3 tracking calls. Language, My location, Payment methods, Coupons, Share app, Help centre, T&C, About, Logout, Tutorials, Change password, Change avatar not tracked as clicks | `profile_screen.dart:274-410` | Settings usage unknown |
| G-9 | Chat has zero events beyond the tab tap (`chat_feature/views`: 50 handlers) | grep | Support demand and chat adoption unmeasurable |
| G-10 | Top-up: screen view and one click tracked; amount chosen, payment method, success/failure not | `top_up_screen.dart` 24 handlers, 2 calls | Wallet funnel blind (money flow) |
| G-11 | Address management: add/edit/delete/set-default, map search have no events | `new_location_screen` (16), `new_search_location` (5), `location_screen` (10) | Address friction invisible; `address_selected` only fires inside booking |
| G-12 | No error/health telemetry: API failures, payment errors, empty/error states, no-internet screen, force-update sheet, permission outcomes | `no_internet_screen`, `handle_server_error_screen`, `custom_button_sheet_update_app` untracked | Cannot monitor the app, only the funnel |
| G-13 | Meta and AppsFlyer cover ~7 and 1 events; standard events with ad-optimisation value (`Search`, `AddToCart`, `Schedule`, `Contact`) are missing; AppsFlyer lacks `af_content_view`, `af_initiated_checkout`, `af_complete_registration`, `af_login` | `facebook_helper.dart`, `appflyer_helper.dart` | Weak lookalike / retargeting audiences, no in-app funnel in AppsFlyer cohorts |
| G-14 | Verify staff exclusion also applies to Meta/AppsFlyer sends | `_sendEnabled` only guards Firebase | Staff test orders could pollute ad data |
| G-15 | Duplicate / legacy screens exist next to current ones (`order_detail_screen` vs `_v2`, `signin_screen`/`signup_screen` vs `new_auth/*`, `appointment_status_*` ×3, `banner_detail_screen2`, `lucky_draw*`). Not all are reachable | `view/screen/**` | Instrument only live screens; delivery tasks confirm reachability before adding events |
| G-16 | Event params carry no UI context: no `screen_name`, `app_language`, push/location permission state | `track()` signature | Cannot segment behaviour by language or permission state |
| G-17 | Clarity has no custom tags | `Helper.maybeInitClarity` | Cannot jump from a portal screen to its session replays |
| G-18 | Main-flow `value` in several ad-hoc events is not sent; `currency` hard-coded `USD` in `_serviceParameters` | `analytics_helper.dart:277-290` | Fine today (USD-only); note if KHR is introduced |

---

## 2. Principles for the new events

1. **Reuse before adding.** Funnel events stay as in the architecture guide. New events are only for gaps in §1.3.
2. **Few generic events, not hundreds of names.** Firebase keeps at most 500 distinct event names per app and 25 parameters per event. Interaction coverage goes through `screen_viewed`, `overlay_viewed`, `element_clicked`, `form_submitted`, `search_performed`, `app_error`, plus the few domain events in §4.
3. **Same wrapper.** Everything through `AnalyticsHelper.track`, so `event_id`, `seq`, `schema_version` are always present. No direct `logEvent`.
4. **Stable ids, never labels.** `screen_name` and `element_id` are `snake_case` ids from a registry (§3.2), never translated text. Khmer/English labels live in the portal (`ref_app_screen`, `ref_app_element`).
5. **No PII.** No phone, name, address, free-text message, coupon-search text beyond the existing `promo_code`, or card data. Search `query` is lower-cased, trimmed, max 30 chars, only for the service search box.
6. **Legacy events stay for one release** (dual-fire) so Firebase audiences and Meta/Marketing reports do not break; remove after the portal shows parity (decision O-3).
7. **Money and order facts remain backend-owned** (architecture §1, §11.1). Client events for payment/cancel/reschedule are diagnostics only.

---

## 3. Event model

### 3.1 Common parameters

Existing (`event_id`, `schema_version`, `seq`, `journey_id`, `service_id`, `reg_flow_id`) are unchanged. New, added by the wrapper from app state so call sites do not pass them:

| Param | Type | Example | Applies to |
|---|---|---|---|
| `screen_name` | string ≤40 | `checkout` | every new generic event; also added to legacy click events when fired from a screen with a registry id |
| `app_language` | string | `km` / `en` | **user property**, not per event |
| `push_enabled` | string | `granted` / `denied` / `unknown` | user property |
| `location_permission` | string | `granted` / `denied` / `unknown` | user property |
| `has_ordered` | string | `0` / `1` | user property (set after first confirmed order; backend remains source of truth) |

Constraint: Firebase allows 25 user properties; `auth_state`, `first_campaign`, `utm_campaign`, `utm_source` already use 4.

### 3.2 Registries

`lib/analytics/tracking_registry.dart` (new) holds two constant sets. The ids below are the **seed**; delivery tasks complete them for their screens and add the same rows to `ref_app_screen` / `ref_app_element` through the backend (§5.4).

**`screen_name`** — `splash`, `splash_ad`, `welcome`, `sign_in`, `sign_up`, `otp`, `set_password`, `customer_source`, `notification_prompt`, `home`, `home_search`, `see_more`, `banner_detail`, `service_detail`, `service_addons`, `checkout`, `coupon_list`, `schedule_sheet`, `payment_sheet`, `payment_result`, `appointment_status`, `bookings`, `order_detail`, `order_cancel`, `order_reschedule`, `order_extend`, `leave_review`, `review_done`, `notifications`, `notification_detail`, `chat_list`, `chat_room`, `chat_story_create`, `profile`, `profile_info`, `change_password`, `change_avatar`, `locations`, `location_edit`, `location_search`, `language`, `payment_methods`, `my_coupons`, `help_center`, `tutorial_center`, `tutorial_detail`, `about`, `topup`, `topup_history`, `topup_status`, `webview`, `no_internet`, `server_error`, `force_update`. `screen_group`: `launch | auth | home | booking | checkout | orders | notifications | chat | profile | topup | system`.

**`element_id`** — pattern `<verb_or_noun>_<target>` unique within a `screen_name`, e.g. `schedule_appointment`, `confirm_pay`, `apply_coupon`, `read_full_policy`, `menu_my_location`, `menu_language`, `menu_logout`, `tab_bookings`. `element_type`: `button | link | icon | tab | card | banner | chip | menu_row | toggle | input | sheet_action | back`. Per-screen seed lists are in §4.

### 3.3 New generic events

| Event | Trigger | Params (beyond common) | Platforms |
|---|---|---|---|
| `screen_viewed` *(exists, extend)* | A page route, a bottom tab becoming visible, or a web-view page opens. Debounce: not twice in a row for the same `screen_name` | `screen_name`, `screen_group`, `previous_screen`, `entry_type` (`nav`/`tab`/`push`/`deeplink`/`back`) | Firebase; **backend ingest (new)** |
| `screen_exited` | Leaving a screen after ≥1 s | `screen_name`, `duration_seconds` (int, capped 1800) | Firebase; backend ingest |
| `overlay_viewed` | A bottom sheet, dialog, popup, or promo overlay appears | `overlay_name`, `overlay_type` (`sheet`/`dialog`/`popup`/`toast`), `screen_name`, `trigger` (`user`/`auto`/`push`) | Firebase; backend ingest |
| `element_clicked` | A tap on a registered interactive element | `screen_name`, `element_id`, `element_type`, `section` (optional), `position` (int, optional), `item_id` (optional id of the entity: banner, service, order, notification, coupon), `target` (optional: screen id, `url`, `telegram`, `phone`) | Firebase; backend ingest |
| `form_submitted` | Submit of a form (sign in/up, profile info, change password, address, review, cancel reason, top-up amount) | `form_id`, `result` (`success`/`error`/`invalid`), `error_code` (optional), `field_count` | Firebase; backend ingest |
| `search_performed` *(architecture §11.1)* | Debounced 600 ms after typing stops, or on submit | `query` (≤30), `result_count`, `screen_name` | Firebase; Meta `Search`; AppsFlyer `af_search` |
| `search_result_clicked` *(§11.1)* | Result tapped | `query`, `rank`, `service_id` | Firebase |
| `setting_changed` | Language switched, notification toggle, etc. | `setting_id` (`language`/`notifications`/...), `setting_value` (short code; not `value`, which is numeric on money events) | Firebase; sets user property |
| `permission_result` | After an OS prompt for push / location / photos / camera (pairs with `push_permission_viewed/result` in the automation guide §11) | `permission` , `result` (`granted`/`denied`/`limited`), `context` (`pre_prompt`/`system`) | Firebase; backend ingest |
| `support_contacted` | Telegram, phone, in-app chat or email support entry used | `channel` (`telegram`/`phone`/`chat`/`email`), `source_screen`, `element_id`, `order_id` (optional) | Firebase; Meta `Contact`; AppsFlyer `af_contact` |
| `app_error` | API failure shown to the user, payment error, web-view load error, no-internet screen, force-update sheet, unhandled exception caught by the app | `error_type` (`api`/`payment`/`webview`/`network`/`update`/`exception`), `screen_name`, `endpoint_group` (e.g. `order_preview`, never the full URL), `http_status`, `error_code`, `fatal` (0/1) | Firebase; Crashlytics (fatal/non-fatal); backend ingest |

### 3.4 Domain events to add (client)

| Event | Trigger | Params | Why |
|---|---|---|---|
| `cancellation_started` / `cancellation_fee_shown` | Cancel sheet opened / fee line shown | `order_id`, `fee_amount` | Matrix N5; pending request 3.4 |
| `cancellation_reason_selected` | Reason chosen in cancel sheet | `order_id`, `reason_code` | Why customers cancel |
| `extend_started` / `extend_payment_selected` | Extend-hour sheet open / pay method chosen | `order_id`, `payment_method` | Upsell funnel (the `extend_hour_pay` legacy click exists) |
| `topup_started`, `topup_amount_selected`, `topup_payment_started` | Wallet top-up funnel | `amount`, `payment_method`; `value`, `currency` as doubles | G-10. Completion/failure is **backend-owned** (`topup_completed`, `topup_failed`) |
| `coupon_list_viewed`, `coupon_copied`, `coupon_use_clicked` | Profile → My coupons | `coupon_id` (not code text) | Offer engagement beyond `promo_applied` |
| `address_added`, `address_edited`, `address_deleted`, `address_default_set` | Address management | `address_id`, `source` (`profile`/`checkout`), `has_zone` (0/1) — **never** street text | G-11 |
| `chat_opened`, `chat_message_sent` | Chat room opened / message sent | `message_type` (`text`/`image`/`video`/`story`) — **never** content | G-9 |
| `content_shared` | Share app / share order | `content_type`, `method` | Referral |
| `logout_clicked`, `account_delete_requested` | Profile | — | Retention signals |

### 3.5 Legacy events: keep, map, retire

| Legacy event (count of call sites) | Replacement | Action |
|---|---|---|
| `contact_us_telegram` (8) | `support_contacted` + `element_clicked` | Dual-fire one release, add `source_screen` |
| `home_screen`, `booking_screen`, `profile_screen`, `location_screen`, `profile_info_screen`, `see_more_screen`, `topup_screen`, `top_up_history`, `top_up_status`, `banner_detail_screen`, `homepage_notification_screen_view`, `sign_in_screen`, `sign_up_screen`, `splash_*` | `screen_viewed` | Dual-fire, then retire |
| `tracking_pages` | `screen_viewed` (`entry_type=tab`) + `element_clicked` (`type=tab`) | Dual-fire |
| `service_click`, `bcombos_click`, `click_promote`, `language_selection`, `sign_in_sign_up`, `guest_mode_button`, `sign_in_continue`, `create_account_button`, `customer_source_*`, `notification_prompt_*`, `notification_banner_not_now`, `turn_on_notifications_click`, `join_telegram_channel`, `increase/decrease_quantity`, `one_item_at_a_time`, `instant_cleaning_explore_other_options`, `reschedule_button_*`, `cancel_booking_confirm`, `extend_hour_pay`, `pay_now_order_details`, `leave_review*`, `rating_selected`, `fully_book_button`, `cancel_button`, `register_button_on_home` | `element_clicked` (with `screen_name`) | Dual-fire; `register_button_on_home` and others used in Meta/GA audiences must be checked by marketing first (O-3) |

---

## 4. Screen-by-screen coverage map

Legend — **Have**: tracked today. **Add**: new in this spec. Platform: **FA** Firebase, **M** Meta, **AF** AppsFlyer, **BE** backend-owned record, **CR** Crashlytics. Every screen below also gets `screen_viewed`/`screen_exited` (not repeated). Element ids are seeds; delivery tasks finish them from a fresh scan of the file.

### A. Launch (task App-tracking-home-discovery)
| Screen (file) | Have | Add |
|---|---|---|
| Splash, splash ad (`splash/*`) | `splash_ad_screen`, `splash_cta_button`, `splash_skip_button`, app_opened, install | `element_clicked`: `splash_ad_cta`, `splash_ad_skip`; `app_error` for `no_internet`, `server_error`, `force_update` screens (they have 0 calls) |
| Force-update sheet (`custom_button_sheet_update_app`) | – | `overlay_viewed`, `element_clicked` `update_now`, `update_later` |

### B. Auth and onboarding (task App-tracking-account-profile-chat)
| Screen | Have | Add |
|---|---|---|
| Welcome, sign in, sign up, OTP (`account/*`, `new_auth/*`) | Registration funnel events, OTP events, `guest_mode_button`, `sign_in_continue`, `create_account_button` | `form_submitted` (`sign_in`, `sign_up`, `otp`), `element_clicked` for country-code picker (`country_code_picker`), Telegram login (`login_telegram`), terms link (`terms_link`), `app_error` on OTP send/verify API failure (reason already in `otp_failed`) |
| Set password, Change password | – | `form_submitted` (`set_password`, `change_password`) |
| Customer source, notification prompt | `customer_source_*`, `notification_prompt_*` | `permission_result` (`push`), unify under `element_clicked` |
| Registration entry from checkout gate (`confirm_and_pay_checkout_screen:1596` `sign_up`) | – | `registration_started` with `entry=checkout_gate` — verify it fires |

### C. Home and discovery (task App-tracking-home-discovery)
| Screen | Have | Add |
|---|---|---|
| Home (`home_screen`, 17 handlers) | `homepage_viewed`, `service_list_viewed`, `service_click`, `bcombos_click`, `click_promote`, `language_selection`, banners viewed/clicked | `element_clicked`: `search_bar`, `notification_bell`, `top_up_card`, `history_card`, `service_card` (+`item_id`, `position`), `see_all_services`, `greeting_avatar`, pull-to-refresh (`refresh_home`), `app_error` on home API failure |
| Home search (`home_service_search_screen`) | – | `search_performed` (also when `result_count=0`), `search_result_clicked` |
| See more (`see_more_screen`) | `see_more_screen`, `contact_us_telegram` | `element_clicked` per category card |
| Banner detail (`banner_detail_screen`, 11 handlers; PDF/image viewers) | `banner_detail_screen`, `register_button_on_home` | `element_clicked`: `banner_cta`, `banner_share`, `banner_pdf_open`, `banner_image_zoom` |
| Home sections (`home_latest_from_beasy_section`, `telegram_widget`) | `bcombos_click`, `join_telegram_channel`, `turn_on_notifications_click` | `element_clicked` with `section`, `position` |
| Bottom tabs (`app_screen`) | `tracking_pages` | `screen_viewed entry_type=tab`; guest tap on a locked tab → `element_clicked` `tab_locked_guest` |

### D. Service detail and booking (task App-tracking-booking-checkout)
| Screen | Have | Add |
|---|---|---|
| Service detail (`category_building_screen`, 23 handlers) | `service_viewed`, `booking_started`, `service_option_selected`, `duration_selected`, add-on and pair events, `task_info_viewed`, `equipment_viewed`, `increase/decrease_quantity`, `one_item_at_a_time`, `contact_us_telegram` | `element_clicked`: `schedule_appointment`, `need_more_than_2_hours`, `showcase_task_info`, `retry` (+`app_error` on service API failure), `add_to_booking`; `overlay_viewed` for task-info and equipment sheets |
| Sub category, product option selector, instant rooms, time selector, lucky draw (only if reachable) | – | `element_clicked` for `room_selected`, `instant_time_selected`, `sub_category_selected` (`item_id`) |
| Schedule sheet / calendar / slot chips | `date_selected`, `time_selected`, `slot_selected` | `overlay_viewed` `schedule_sheet`; `app_error` for "no slots"/"unavailable schedule" sheet (`custom_unavailable_schedule_sheet`) |
| Address pick inside booking | `address_selected` | `address_*` events (§3.4) when added from checkout |

### E. Checkout and payment (task App-tracking-booking-checkout; Meta/AppsFlyer: App-tracking-meta-appsflyer)
| Screen | Have | Add |
|---|---|---|
| Checkout (`confirm_and_pay_checkout_screen`, 22 handlers, 3600 lines) | `booking_summary_viewed`, `checkout_started`, `begin_checkout`, `add_payment_info`, `payment_started`, `promo_*`, `contact_us_telegram`; Meta `InitiateCheckout`, `AddPaymentInfo` | `element_clicked`: `refresh_price`, `change_schedule`, `read_full_policy` (cancellation policy sheet), `add_note`, `apply_coupon`, `remove_coupon`, `select_payment_method`, `confirm_pay` (exists as click), `sign_up_gate`; `overlay_viewed` for policy and payment sheets; `app_error` for price-preview, apply-coupon, payment-create failures; Meta `Schedule` on `slot_selected` |
| Coupon list (`apply_coupon_list_screen`) | `promo_*` via tracker, `notification_banner_not_now` | `coupon_list_viewed`, `element_clicked` `coupon_card`, `search_coupon` |
| Payment (ABA webview, payment sheet, result sheet, appointment status ×3) | `payment_failed`, `payment_returned_success`, `order_confirmation_viewed`, `purchase` | `overlay_viewed` `payment_result`; `app_error error_type=payment` on webview load/return failure; `element_clicked` on result CTAs (`view_order`, `back_home`) |

### F. Orders (task App-tracking-orders-after-purchase)
| Screen | Have | Add |
|---|---|---|
| Bookings list (`booking_screen`) | `booking_screen`, Meta `ViewBooking` | `element_clicked` order card (`item_id`=order id), tabs (upcoming/past), `app_error` list failure |
| Order detail v2 + widgets | `leave_review`, `pay_now_order_details`, `reschedule_*`, `cancel_booking_confirm`, `extend_hour_pay`, `turn_on_notifications_click` | `element_clicked`: `contact_us`, `call_cleaner`, `chat_cleaner`, `view_cleaner`, `services_card_expand`, `where_when_edit`; cancel sheet: `cancellation_started`, `cancellation_fee_shown`, `cancellation_reason_selected`, `form_submitted(cancel_reason)`; extend sheet: `extend_started`, `extend_payment_selected`; reschedule sheet: tracker events exist, add `overlay_viewed` and `element_clicked` `confirm_new_time`, `try_again` |
| Review (`leave_review`, `review_done`) | `rating_selected`, `leave_review_submitted`, `review_submitted` | `form_submitted(review)` with `tags_count`, `has_comment` (0/1, never text) |
| Swipe-to-complete, cleaners-not-finished dialog, service add-on screens | – | `element_clicked` and `overlay_viewed` |
| Order detail v1 (`order_detail_screen`, 2037 lines) | duplicates of v2 | Confirm reachability; if dead, remove the dual events instead of extending |

### G. Notifications (task App-tracking-home-discovery)
| Screen | Have | Add |
|---|---|---|
| Inbox (`announcement_list_screen`, 12 handlers) | `notification_inbox_viewed` (helper exists) | `element_clicked`: `mark_all_read`, `mark_as_read`, `notification_card` (`item_id`), `tab_*`; `notification_deleted`; `notification_cta_clicked` (automation guide §11) |
| Detail (`announcement_not_details`, detail sheet) | `notification_detail_viewed`, `contact_us_telegram` | CTA click with `notification_id`, `cta` |

### H. Profile and settings (task App-tracking-account-profile-chat)
| Screen | Have | Add |
|---|---|---|
| Profile (`profile_screen`, 13 handlers) | `profile_screen`, notification banner events | `element_clicked`: `menu_my_location`, `menu_payment_methods`, `menu_language`, `menu_notifications`, `menu_my_coupons`, `menu_share_app`, `menu_help_center`, `menu_terms`, `menu_chat`, `menu_contact_support`, `menu_about`, `menu_logout`; `logout_clicked` |
| Profile info, avatar, change password | `profile_info_screen` | `form_submitted(profile_info)`, `element_clicked` `change_avatar`; `permission_result` photos/camera |
| Location list/edit/search (`location_screen`, `new_location_screen`, `new_search_location`) | `location_screen` | `address_*`, `permission_result(location)`, `search_performed` for place search (no text beyond 30 chars; place query is not a customer address), `element_clicked` `use_current_location`, `set_default` |
| Language | `language_selection` on Home | `setting_changed(language)` + user property `app_language` |
| Payment methods, My coupons | `notification_banner_not_now`, `turn_on_notifications_click` | `coupon_*` events, `element_clicked` `add_payment_method` if reachable |
| Help centre, tutorials, about | – | `element_clicked` per row, `tutorial_viewed` (`tutorial_id`) via `screen_viewed` + `item_id`, video play (`element_clicked` `tutorial_play`) |

### I. Chat (task App-tracking-account-profile-chat)
`message_screen`, `chat_feature/views/*` (50 handlers): `screen_viewed` for list/room/story-create/media-preview; `chat_opened`, `chat_message_sent`, `element_clicked` `attach_media`, `camera`, `gallery`, `story_create`, `story_post`; `app_error` on socket/MQTT connect failure (`error_type=network`). No message content or file names.

### J. Top-up (task App-tracking-booking-checkout)
`top_up_screen` (24 handlers), history, status, ABA payment screen: `topup_started`, `topup_amount_selected`, `topup_payment_started`, `element_clicked` `topup_history`, `app_error error_type=payment`; completion/failure from backend; Meta/AppsFlyer: none (not a service purchase) unless marketing asks (O-5).

### J2. Cross-screen controls: language switch and the top-right customer-service (CS) button
Audited 2 Nov. **Language:** one controller method, `LocalizationController.selectLanguage` (`localization_controller.dart:75`), serves both pickers (Home top-right sheet `home_screen.dart:1216`, Profile → Language `language_screen.dart:44`); `syncServerLanguage` changes it silently after sign-in. Today only the Home icon tap fires the legacy `language_selection` (opening, not the choice). Add in `selectLanguage`: `setting_changed` (`setting_id=language`, `value=km|en`, `source=home_sheet|profile|server_sync`, `previous_value`) plus user property `app_language` (also set at app start), and `element_clicked` `language_picker` on the Home icon. **CS button:** top-right icon on Sign in, OTP, Sign up, Customer source (opens `CustomContactSupportSheet`) and Order detail (opens an order chat via `MessageController.postConversation`). Add `element_clicked` `contact_support` on each (`target=sheet|order_chat`, `item_id`=order id on order detail), `overlay_viewed` `contact_support` for the sheet, and `support_contacted` for each real channel: Telegram (replaces anonymous `contact_us_telegram`, kept one release), hotline call, email, order chat; every one with `source_screen`. Owners: T-0067 (language, sheet, customer source), T-0074 (sign in/OTP/sign up buttons), T-0066 (order detail button).

### K. Shared widgets and system (assigned to the slice that owns the screen using them)
`custom_show_dialog` (21 handlers, 3 calls), `custom_banner_list`, `custom_contact_support_sheet`, `custom_telegram_banner`, `custom_announcement_detail_sheet`, `custom_promo_overlay`, `custom_notification_offer_banner`, `custom_category_selection_alert_sheet` (see `docs/category-selection-alert-sheet-trigger.md`): each fires `overlay_viewed` once from the widget with a `trigger`, and `element_clicked` with the caller's `screen_name`. Shared widgets receive `screen_name` as an optional constructor argument or read it from `ScreenTrackingObserver.current`; the foundation task decides which (default: observer).

---

## 5. Platform assignment

### 5.1 Meta (App Events)

Existing 7 stay as is (verification in T-0031 / [Meta_App_Events_Setup.md](Meta_App_Events_Setup.md); no change to them here). Proposed additions, each behind the existing ATT gate and staff exclusion:

| Meta event | Fires from | Params |
|---|---|---|
| `Search` | `search_performed` | `search_string` (≤30), `content_type=service` |
| `Schedule` | `slot_selected` | `content_id` (service id), `value`, `currency` |
| `Contact` | `support_contacted` | `content_type=channel` |
| `AddToCart` | first `booking_details_completed` per journey | `content_id`, `value`, `currency` |

No Meta event for screens, errors or settings (no ad-optimisation value; avoids sending behavioural data to a third party). `Lead` and `Subscribe` are not proposed (no matching bEasy action).

### 5.2 AppsFlyer

Add (wrapper `AppFlyerHelper.logEvent`): `af_content_view`, `af_search`, `af_add_to_cart`, `af_initiated_checkout`, `af_add_payment_info`, `af_complete_registration`, `af_login`, `af_schedule`, `af_contact` (custom), `af_purchase` (exists). Purpose: in-app funnel per campaign cohort and retention by media source. Revenue stays on `af_purchase` only.

### 5.3 Firebase vs Crashlytics vs Clarity

| Need | Tool | Detail |
|---|---|---|
| Behaviour and funnel | Firebase → BigQuery → backend | all events in §3 |
| Crashes, ANRs, non-fatal exceptions | **Firebase Crashlytics (new)** | `FlutterError.onError`, `PlatformDispatcher.onError`; log `app_error` with `fatal` flag for correlation; set user id = `customer_id` (same as `setUserId`), no PII |
| Startup, API latency, screen render | **Firebase Performance (new, optional)** | HTTP metrics from `DioClient` interceptor, custom trace `app_start`, `checkout_preview` (O-4) |
| Visual reproduction | Clarity | `Clarity.setCustomTag('screen_name', …)` on `screen_viewed`; `Clarity.sendCustomEvent` for `app_error` |

### 5.4 Backend-owned (unchanged, listed for completeness)
`purchase_completed`, `order_cancelled`, `order_rescheduled`, `order_refunded`, `payment_failed`, `service_completed`, `registration_completed`, `OTP_SENT`, `notification_sent`, `cs_contact_logged`, and new `topup_completed`, `topup_failed`.

---

## 6. Backend requirements

*(All doc-only; the API repo is not cloned. Open asks are copied into `backend-request/pending-request-backend.md` §8.)*

### 6.1 Ingest rules
- Accept and store the §3.3/§3.4 client events. Stop skipping `screen_viewed`; keep skipping the legacy list until retired.
- Dedupe by `event_id`; order by `(event_timestamp, seq)`; quarantine only when `event_id`, `schema_version` or `seq` is invalid (same rule as today). Unknown `screen_name`/`element_id` are stored and flagged `unregistered`, never rejected.
- Guests keyed by `app_instance_id`, merged to `customer_id` at `identity_link` (architecture §5.5); keep guest interaction events.
- **Do not put interaction events in `event_log`/journey tables.** That table drives funnel counts and the Customer page timeline; ~10–30× more rows would slow it. Use a separate table (6.2). A configurable subset (`screen_viewed`, `support_contacted`, `app_error`, `search_performed`, `cancellation_*`) is also written to `event_log` so the Customer page timeline shows them (portal toggle "Show taps").

### 6.2 Data structures (3NF, additions to architecture §6/§11.2)

```text
ref_app_screen   (screen_name PK, screen_group, label_en, label_km, is_active)
ref_app_element  (screen_name FK, element_id, element_type, label_en, label_km, PK(screen_name, element_id))
app_interaction_event (
  event_id PK uuid, event_name, occurred_at, session_id FK -> app_session,
  customer_id NULL, app_instance_id, screen_name, element_id NULL, element_type NULL,
  section NULL, position NULL, item_id NULL, target NULL, metadata jsonb,
  app_version, platform, app_language, auth_state)           -- partition by day; raw retention 90 days
app_screen_view (
  event_id PK, session_id FK, customer_id NULL, app_instance_id, screen_name,
  previous_screen NULL, entry_type, duration_seconds NULL, occurred_at)   -- retention 90 days
app_error_event (
  event_id PK, occurred_at, session_id FK, customer_id NULL, app_instance_id,
  error_type, screen_name NULL, endpoint_group NULL, http_status NULL, error_code NULL,
  fatal bool, app_version, platform)                                       -- retention 180 days
app_usage_daily (
  day, screen_name, element_id NULL, platform, app_version, app_language, auth_state,
  campaign_code NULL, views, unique_users, unique_guests, total_duration_seconds,
  clicks, clickers, exits, errors,
  PK(day, screen_name, COALESCE(element_id,''), platform, app_version, app_language, auth_state, COALESCE(campaign_code,'')))
```

`app_usage_daily` is a derived, rebuildable read model (nightly job after the existing `pipeline.run`, plus hourly for today). `unique_*` are per-day counts; range uniques are computed from `app_screen_view`/`app_interaction_event` (not summed from daily rows).

### 6.3 Metric definitions

| Metric | Definition |
|---|---|
| Views | count of `screen_viewed` for the screen |
| Unique viewers | distinct `customer_id` (or `app_instance_id` when guest) with ≥1 view in range |
| Avg time on screen | mean `duration_seconds` of `screen_exited`, excluding >1800 s |
| Exit rate | sessions whose **last** `screen_viewed` is this screen ÷ sessions that viewed it |
| Element clicks / clickers | count / distinct users of `element_clicked` |
| Element CTR | clickers ÷ unique viewers of the same `screen_name` in range |
| Next-screen share | among views of screen A, share whose next `screen_viewed` in the session is B |
| Error rate | `app_error` count ÷ screen views (per `screen_name`, `endpoint_group`) |
| Crash-free users | from Crashlytics (BigQuery export), shown as a single tile |

### 6.4 APIs for the portal

All under the existing portal conventions (`period`, `from`/`to`, filters from `GET /meta/filter-options`, masked identifiers, `x-total-count` pagination). Common filters: `platform`, `app_version`, `language`, `auth_state` (`guest`/`registered`), `source`, `campaign_id`, `service_id`, `audience` (saved CRM segment), `screen_group`.

| Method | Path | Returns |
|---|---|---|
| GET | `/meta/app-screens` | Registry: screens and elements with labels (EN/KM), group, type, `unregistered` count |
| GET | `/app-usage/overview` | KPI tiles + daily trend: sessions, active users, screens per session, avg session time, top screens, top elements, errors, crash-free users |
| GET | `/app-usage/screens` | Table: screen, views, unique viewers, avg time, exit rate, error rate, trend (sparkline), paged |
| GET | `/app-usage/screens/{screen_name}` | One screen: KPIs, trend, elements list with clicks/CTR, previous/next-screen shares |
| GET | `/app-usage/elements` | Flat element table: screen, element, type, clicks, clickers, CTR, trend |
| GET | `/app-usage/flows` | Transition counts `from_screen → to_screen` (top N, min count), for a flow diagram |
| GET | `/app-usage/errors` | Grouped errors: `error_type`, `endpoint_group`, `http_status`, `error_code`, count, affected users, first/last seen, app versions |
| GET | `/app-usage/search-terms` | Top search queries, count, zero-result share, result-click rank |
| GET | `/customers/{id}/activity` | *(exists)* add `include=interaction` to return the interaction rows interleaved |
| POST | `/app-usage/export` | Excel/CSV of any table above (same filters) |

Response sketch (screens):
```json
{ "data": [ { "screenName": "checkout", "screenGroup": "checkout", "label": {"en":"Checkout","km":"…"},
  "views": 12840, "uniqueViewers": 7410, "avgSeconds": 74, "exitRate": 0.31, "errorRate": 0.012,
  "trend": [ {"day":"2026-11-01","views":410} ] } ],
  "meta": { "from": "2026-10-03", "to": "2026-11-02", "freshAt": "2026-11-02T03:00:00+07:00" } }
```

### 6.5 Privacy and volume
- No PII in params; `query` and `item_id` only as defined. `metadata` is whitelisted per event.
- Expected volume: 25–60 interaction events per session; with ~N sessions/day backend sizes the partitioned table (ask backend to estimate). Client mitigations: no scroll events, no per-keystroke events, `screen_exited` only ≥1 s, debounced search.
- Consent: today Firebase collects without an in-app consent screen; the architecture §11.5 rules apply unchanged. Meta/AppsFlyer sends remain behind ATT.

---

## 7. Portal visualization

New group **Marketing → App Usage** (route `app-usage`, tabs below), built with the responsive design system v2 components (`Portal-rds-*`, see the plan for the sequencing with Portal-rds-marketing which owns the current marketing routes). Reads only the APIs in 6.4; same period/filter bar as Customer Journey.

| Tab | Content | Charts / components |
|---|---|---|
| **Overview** | KPI tiles (active users, sessions, screens/session, avg session time, error rate, crash-free users); trend of active users; top 10 screens; top 10 elements; top errors | line (users/sessions), horizontal bars (top screens/elements), stat tiles with delta vs previous period |
| **Screens** | Sortable table (views, viewers, avg time, exit rate, error rate, sparkline); click row → side panel with elements, previous/next screens | table (RDS data-table), sparkline, bar list |
| **Elements** | All clickable elements: clicks, clickers, CTR, trend; filter by screen/type; `unregistered` badge for new ids | table, small multiples trend on row expand |
| **Flows** | Where users go next: `from → to` transition counts, min-count slider | Sankey/flow (top 20 transitions) with table fallback on mobile |
| **Search** | Top queries, zero-result queries, click rank | table, bar |
| **Health** | Errors grouped by type/endpoint/version, trend, affected users; crash-free users and top crashes (from Crashlytics export) | stacked bars by `error_type`, table with drill to affected users list |

Filters (all tabs): period (presets + custom), platform, app version, language, auth state, acquisition source/campaign, service, audience/segment, screen group. Segmentation compare: "Guest vs Registered", "Android vs iOS", "Khmer vs English", "Campaign A vs B" via a compare toggle (two series).

Customer page: existing Activity tab gets a **Show taps** toggle (default off) rendering `screen_viewed`/`element_clicked` rows in the session timeline; Customer Journey step cards get a link "Screens in this session".

Registry editing: Settings → *Tracking registry* (read-only list from `/meta/app-screens` with `unregistered` ids; label editing is a later request).

Charts follow the dataviz skill palette and the portal's existing journey colours. Khmer and English labels for every string (`app/locales/marketing/{en,km}/`).

---

## 8. Quality and verification

- **Contract:** extend `EventContract.required` with the new events (`screen_name` required on `element_clicked`, `overlay_viewed`, `form_submitted`, `app_error`; `element_id`/`element_type` on `element_clicked`). Unit test per event builder (pattern in `test/`), plus a **registry test**: every `screen_name`/`element_id` used in code exists in the registry and matches `^[a-z][a-z0-9_]{0,39}$`.
- **Lint-style check:** a test greps `lib/view` and fails if a file with `onTap`/`onPressed` on a registered widget has no registry mapping (opt-out list allowed with a reason).
- **Device QA:** Firebase DebugView matrix per slice (screen, action, expected event, expected params), recorded in the task file. Backend sign-off with one real batch per event before release.
- **Volume budget:** event names stay below 150 (limit 500); ≤12 params/event (limit 25).
- **Regression:** compare daily counts of legacy vs new during the dual-fire release; difference >5% is investigated.

## 9. Rollout

1. Foundation + registry (app) → backend accepts events in staging (can run in parallel).
2. Slices (home, booking/checkout, orders, account/chat) ship progressively, each with dual-fire legacy events.
3. Monitoring (Crashlytics, `app_error`) ships first or with foundation because it is the safest, highest-value data.
4. Portal App Usage builds against the contract with mock data, goes live when backend data is.
5. Retire legacy events one release after portal parity.

## 10. Decisions and open questions

Decisions made from docs and project rules (no answer needed):
- D-1 Transport stays **Firebase → BigQuery → backend** (architecture §1); no direct client-to-API event posting.
- D-2 Generic events plus registries, not per-element event names (Firebase name cap, mapping simplicity).
- D-3 Interaction events live in their own tables; only a subset also in `event_log`.
- D-4 Backend remains source of truth for money and order state.
- D-5 Dead/duplicate screens are not instrumented until confirmed reachable.

Open questions (recommendation first; planning does not wait):
- **O-1** Adopt **Crashlytics** now? *Recommend yes* — no crash visibility today; small native change.
- **O-2** Adopt **Firebase Performance**? *Recommend later (backlog)*; `app_error` + Crashlytics cover the need now.
- **O-3** Retire legacy ad-hoc events? *Recommend dual-fire one release, then retire* after marketing confirms no GA/Meta audience depends on them (esp. `register_button_on_home`, `service_click`, `bcombos_click`).
- **O-4** Raw retention 90 days / errors 180 days acceptable? *Recommend yes*; aggregates kept 2 years.
- **O-5** Top-up to Meta/AppsFlyer? *Recommend no* until marketing defines a value for it.
- **O-6** Register labels: who maintains Khmer labels for `ref_app_*`? *Recommend marketing via portal later; seed from this spec.*
