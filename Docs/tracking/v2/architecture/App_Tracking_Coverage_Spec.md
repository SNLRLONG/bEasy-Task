# bEasy App — Tracking Coverage Audit and Specification

Updated: 2026-11-03 · Status: **draft for review** (T-0064 home and discovery, T-0067 account, profile and chat, T-0065 booking, checkout and top-up tracking, and T-0069 Meta and AppsFlyer events built, not merged) · Plan: [`Tasks/_plans/Cross-tracking-coverage.md`](../../../../Tasks/_plans/Cross-tracking-coverage.md)

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
| G-6 | Orders: v2 detail has 10 tracking calls but the **cancel sheet (13 handlers), reschedule sheet (13), extend sheet (4), services card, bottom actions** have none. Cancel and reschedule events are built in T-0079 (not merged); `cancellation_fee_shown` is not built (placeholder fees), see Cancellation_And_Reschedule_Spec §2 | `order_detail/widgets/*` | Cancellation reasons and fee impact invisible on the client side |
| G-7 | Search untracked: `home_service_search_screen.dart` has input + results, no `search_performed` / `search_result_clicked` (both in architecture §11.1 as "new") | file has 3 handlers, 0 calls | No search-term demand data |
| G-8 | Profile menu: 13 rows, 3 tracking calls. Language, My location, Payment methods, Coupons, Share app, Help centre, T&C, About, Logout, Tutorials, Change password, Change avatar not tracked as clicks | `profile_screen.dart:274-410` | Settings usage unknown |
| G-9 | Chat has zero events beyond the tab tap (`chat_feature/views`: 50 handlers) | grep | Support demand and chat adoption unmeasurable |
| G-10 | Top-up: screen view and one click tracked; amount chosen, payment method, success/failure not | `top_up_screen.dart` 24 handlers, 2 calls | Wallet funnel blind (money flow) |
| G-11 | Address management: add/edit/delete/set-default, map search have no events | `new_location_screen` (16), `new_search_location` (5), `location_screen` (10) | Address friction invisible; `address_selected` only fires inside booking |
| G-12 | No error/health telemetry: API failures, payment errors, empty/error states, no-internet screen, force-update sheet, permission outcomes | `no_internet_screen`, `handle_server_error_screen`, `custom_button_sheet_update_app` untracked | Cannot monitor the app, only the funnel |
| G-13 | Meta and AppsFlyer cover ~7 and 1 events; standard events with ad-optimisation value (`Search`, `AddToCart`, `Schedule`, `Contact`) are missing; AppsFlyer lacks `af_content_view`, `af_initiated_checkout`, `af_complete_registration`, `af_login` | `facebook_helper.dart`, `appflyer_helper.dart`. Built in T-0069 (§5.1, §5.2), not merged | Weak lookalike / retargeting audiences, no in-app funnel in AppsFlyer cohorts |
| G-14 | Verify staff exclusion also applies to Meta/AppsFlyer sends | `_sendEnabled` only guards Firebase. **Audit result (T-0069): open.** The new T-0069 events skip staff phones; the existing 7 Meta events and `af_purchase` have no staff exclusion and no release-only gate. Not changed; owner decision pending (§10) | Staff test orders could pollute ad data |
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
| `app_language` | string | `en` / `km` / `vi` / `zh-cn` / `zh-tw` / `other` (the app's real codes) | **user property**, not per event; set in `LocalizationController._setLocale` (built in T-0067) |
| `push_enabled` | string | `granted` / `denied` / `unknown` | user property |
| `location_permission` | string | `granted` / `denied` / `unknown` | user property |
| `has_ordered` | string | `0` / `1` | user property (set after first confirmed order; backend remains source of truth) |

Constraint: Firebase allows 25 user properties; `auth_state`, `first_campaign`, `utm_campaign`, `utm_source` already use 4.

### 3.2 Registries

`lib/analytics/tracking_registry.dart` (new) holds two constant sets. The ids below are the **seed**; delivery tasks complete them for their screens and add the same rows to `ref_app_screen` / `ref_app_element` through the backend (§5.4). Home, discovery, launch, banner and notification element ids are in `DiscoveryIds`, not here: see [§4 A/C/G shipped](#4-acg-shipped-t-0064).

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
| `setting_changed` | Language switched, notification toggle, etc. | `setting_id` (`language`/`notifications`/...), `setting_value` (short code; not `value`, which is numeric on money events). Language (built in T-0067): `setting_value` = `en|km|vi|zh-cn|zh-tw|other`, `previous_value`, `source` = `home_sheet|profile|default_fallback|other` | Firebase; sets user property |
| `permission_result` | After an OS prompt for push / location / photos / camera (pairs with `push_permission_viewed/result` in the automation guide §11) | `permission` , `result` (`granted`/`denied`/`limited`), `context` (`pre_prompt`/`system`/`auto_open`). Rules as built (T-0067): push fires on a new grant only (`pre_prompt`); photos and camera on Profile info open (`auto_open`); chat camera icon only when refused; story-screen photos only for `denied`/`limited` | Firebase; backend ingest |
| `support_contacted` | Telegram, phone, in-app chat or email support entry used | `channel` (`telegram`/`phone`/`chat`/`email`), `source_screen`, `element_id`, `order_id` (optional) | Firebase; Meta `Contact`; AppsFlyer `af_contact` |
| `app_error` | API failure shown to the user, payment error, web-view load error, no-internet screen, force-update sheet, unhandled exception caught by the app | `error_type` (`api`/`payment`/`webview`/`network`/`update`/`exception`), `screen_name`, `endpoint_group` (e.g. `order_preview`, never the full URL), `http_status`, `error_code`, `fatal` (0/1) | Firebase; Crashlytics (fatal/non-fatal); backend ingest |

### 3.4 Domain events to add (client)

| Event | Trigger | Params | Why |
|---|---|---|---|
| `cancellation_started` / `cancellation_fee_shown` | Cancel sheet opened / fee line shown. `cancellation_fee_shown` is not built (placeholder fees) | `order_id`, `entry` and order facts (exact params: Cancellation_And_Reschedule_Spec §2.1) | Matrix N5; pending request 11 |
| `cancellation_reason_selected` | Reason chosen in cancel sheet | `order_id`, `reason_code` | Why customers cancel |
| `extend_started` / `extend_payment_selected` | Extend-hour sheet open / pay method chosen | `order_id`, `payment_method` | Upsell funnel (the `extend_hour_pay` legacy click exists) |
| `topup_started`, `topup_amount_selected`, `topup_payment_started` | Wallet top-up funnel | `amount`, `payment_method`; `value`, `currency` as doubles | G-10. Built in T-0065, see [§4 D/E/J shipped](#4-dej-shipped-t-0065). Completion/failure is **backend-owned** (`topup_completed`, `topup_failed`) |
| `coupon_list_viewed` | Checkout coupon list (`apply_coupon_list_screen`) | `service_id`, count | Built in T-0065. Offer engagement beyond `promo_applied` |
| `coupon_copied`, `coupon_use_clicked` | Profile → My coupons | `coupon_id` (not code text) | Not built (T-0067 did not add them) |
| `address_added`, `address_edited`, `address_deleted` | Address management | Params as built in [Registration_And_Location_Flow_Spec.md](Registration_And_Location_Flow_Spec.md) §C.2 (supersedes `address_id`/`has_zone`; `address_default_set` not built) — **never** street text | G-11 |
| `chat_opened`, `chat_message_sent` | Chat room opened / message sent `chat_opened`: `chat_kind` (`order`/`support`), `entry` (`chat_list`/`order_detail`/`other`), `order_id` (only when a plain id). `chat_message_sent`: `message_type` (`text`/`image`/`video`) only — **never** content. Built in T-0067; `story` not built | G-9 |
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
Built in T-0064, see [§4 A/C/G shipped](#4-acg-shipped-t-0064).
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
| Registration entry from checkout gate (`confirm_and_pay_checkout_screen:1596` `sign_up`) | – | `registration_started` with `entry=checkout_address` (shared handler with the address row; see Registration spec B.1) — verified in T-0074 |

### C. Home and discovery (task App-tracking-home-discovery)
Built in T-0064, see [§4 A/C/G shipped](#4-acg-shipped-t-0064); the `Add` column below is the plan, the shipped section wins. Not built: `search_bar`, `notification_bell`, `top_up_card`, `history_card`, `greeting_avatar`, `banner_share`, `banner_pdf_open`.
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
| Service detail (`category_building_screen`, 23 handlers) | `service_viewed`, `booking_started`, `service_option_selected`, `duration_selected`, add-on and pair events, `task_info_viewed`, `equipment_viewed`, `increase/decrease_quantity`, `one_item_at_a_time`, `contact_us_telegram` | **Built (T-0065):** `element_clicked`: `schedule_appointment`, `showcase_task_info`, `equipment_included`, `retry` (+`app_error` `service_options`), `add_to_booking`, `contact_support`, `back`; `overlay_viewed` for task-info, equipment and add-on (`service_addons`) sheets. `need_more_than_2_hours` is covered by T-0082 `instant_upsell_clicked` |
| Sub category, product option selector, instant rooms, time selector, lucky draw (only if reachable) | – | **Built (T-0065):** `sub_category_selected` (`item_id`, `position`), `show_all_options`, `option_info` + `overlay_viewed` `sub_category_detail`. Rooms and Instant time are T-0082 events. **Not built:** `LuckyDrawScreen` (seasonal; its entry overlay `lucky_draw` is T-0064) |
| Schedule sheet / calendar / slot chips | `date_selected`, `time_selected`, `slot_selected` | **Built (T-0065):** `overlay_viewed` `schedule_sheet`, clicks `schedule_confirm`, `schedule_close`; `app_error` `order_schedule` with `error_code` `no_slots` (empty calendar); the unavailable-schedule sheet is informational, reported only as `overlay_viewed` `unavailable_schedule` (Instant fully booked: `instant_fully_booked_viewed`) |
| Address pick inside booking | `address_selected` | `address_*` events (§3.4) when added from checkout |

### E. Checkout and payment (task App-tracking-booking-checkout; Meta/AppsFlyer: App-tracking-meta-appsflyer)
| Screen | Have | Add |
|---|---|---|
| Checkout (`confirm_and_pay_checkout_screen`, 22 handlers, 3600 lines) | `booking_summary_viewed`, `checkout_started`, `begin_checkout`, `add_payment_info`, `payment_started`, `promo_*`, `contact_us_telegram`; Meta `InitiateCheckout`, `AddPaymentInfo` | **Built (T-0065):** `element_clicked`: `refresh_price`, `change_schedule`, `read_full_policy`, `apply_coupon`, `use_other_promo`, `select_payment_method`, `confirm_pay`, `sign_up_gate`, `cross_sell_next`, `pair_sheet_cancel`, `pair_sheet_confirm`, `back`, `contact_support`; `overlay_viewed` `payment_sheet`, `cross_sell_sheet`, `pair_service_sheet`; `app_error` for price-preview, apply-coupon, schedule, payment-methods and order-create failures; `instant_hours` on `checkout_started` and `payment_started`. **Not built:** `add_note` (text field, noise), `remove_coupon`; Meta `Schedule` built in T-0069 (§5.1) |
| Coupon list (`apply_coupon_list_screen`) | `promo_*` via tracker, `notification_banner_not_now` | **Built (T-0065):** `coupon_list_viewed`, `element_clicked` `coupon_card` (`item_id` = coupon id), `search_coupon` (once, first keystroke, no text), `apply_coupon`, `back` |
| Payment (ABA webview, payment sheet, result sheet, appointment status ×3) | `payment_failed`, `payment_returned_success`, `order_confirmation_viewed`, `purchase` | **Built (T-0065):** `overlay_viewed` `payment_result_success` / `payment_result_failed` (instead of one `payment_result`); clicks `back_home`, `try_again`, `result_close`; `app_error` `payment_webview` / `payment_status` on webview load, invalid URL, final non-paid status and 3-minute timeout. `view_order` is **not sent** (no such button; Done is `back_home`) |

### F. Orders (task App-tracking-orders, T-0066)
Built in T-0066 on branch `feat/tracking-orders`, **not yet merged**. Collection only: no screen, navigation, request, amount or return value changed; existing events keep name and params (additions only). Handler-by-handler map: [Tasks/3-in-progress/App-tracking-orders.md](../../../../Tasks/3-in-progress/App-tracking-orders.md) "Handler map". Code: `lib/analytics/trackers/orders_tracking.dart`.

**Where the ids live.** Element and overlay ids are constants in `OrdersIds`, not in `TrackingRegistry`. Backend `ref_app_element` rows must come from `OrdersIds.all` (elements) and `OrdersIds.allOverlays` (overlays), like `BookingIds`; the `app_error` groups are `OrdersIds.allGroups`. This spec does not repeat the list (one fact, one home).

| Screen | Built (T-0066) | Not built or different |
|---|---|---|
| Bookings list (`booking_screen`) | `element_clicked`: tabs `tab_active` / `tab_history` (type `tab`, only when the tab changes), `order_card` (type `card`, `item_id` = bulk order id), `refresh_bookings`, `explore_services` (empty state); `app_error` group `booking_list` | Swipe between tabs is not reported (the page change also fires after a tab tap, so it would double-report) |
| Order detail v2 (`order_detail_screen_v2`) | `element_clicked`: `back`, `contact_support` (app bar) and `contact_us` (bottom), both also send `support_contacted` (`channel=chat`, `order_id`); `pay_now`, `book_again`, `leave_review`, `swipe_complete`, `service_add_on`, `refresh_order` (the notification banner's `notif_banner_open_settings` / `notif_banner_not_now` come from the shared banner widget, not added here); overlays `cleaners_not_finished` (+ `cleaners_not_yet`, `cleaners_done`), `addon_services`, `order_addon`, `order_payment_sheet` (`select_payment_method`, `item_id` = method code), `gift_coupon_confirm` (`gift_apply`, `gift_cancel`); `app_error` groups `order_detail`, `order_complete` | `call_cleaner`, `chat_cleaner`, `view_cleaner`, `services_card_expand`, `where_when_edit`: not in the v2 UI, nothing to track. Legacy `pay_now_order_details`, `leave_review`, `reschedule_button_*` kept (dual-fire) |
| Extend sheet (hours) | `element_clicked` `extend_hours` (card button), `overlay_viewed` `extend_hours`, `extend_close`, `extend_confirm`, `extend_done` (`screen_name=extend_hours`, `section` = step) | The sheet name and `extend_started` / `extend_payment_selected` / `extend_closed` / `instant_progress_viewed` belong to T-0082 (the payment row stays `extend_payment_selected`); T-0066 only adds close, confirm and done |
| Review (`leave_review`, `review_done`) | `element_clicked`: `back`, `review_star` (position 1-5), `review_tag` (type `chip`, position only, no tag name), `review_submit`, `review_done_close_icon`, `review_done_close`; `form_submitted(review)` with `result` (`success`/`error`), `tags_count`, `has_comment` (0/1), `error_code` (`http_<n>` / `network`); never text. Legacy `rating_selected`, `leave_review_submitted` and `review_submitted` (stars) kept | Comment field and rating-options load failure not tracked (no `app_error`) |
| Cancel and reschedule sheets | Owned by T-0079, see [Cancellation_And_Reschedule_Spec.md](Cancellation_And_Reschedule_Spec.md) | `confirm_new_time` / `try_again` left to T-0079 (its sheet file); `cancellation_fee_shown` not built (placeholder fees) |
| Order detail v1 (`order_detail_screen`, `cleaner_widget`) | – | Unreachable (`OrderDetailScreen(` is constructed nowhere); not instrumented, not changed. Deleting it is an owner decision |

**`app_error` groups added:** `booking_list`, `order_detail`, `order_complete` (`review_options` is sent when the rating options fail to load). **Overlay names:** see `OrdersIds.allOverlays`. Shipped differences: `trackFormSubmitted` got typed optional `tagsCount` / `hasComment` params (`analytics_helper.dart`), and `showCustomizeSheet` / `CustomDialog.showInfoDialog2` got an optional `routeSettings` so the order sheets are named overlays.

### G. Notifications (task App-tracking-home-discovery)
Built in T-0064, see [§4 A/C/G shipped](#4-acg-shipped-t-0064). Not built: `notification_deleted`, `notification_cta_clicked`.
| Screen | Have | Add |
|---|---|---|
| Inbox (`announcement_list_screen`, 12 handlers) | `notification_inbox_viewed` (helper exists) | `element_clicked`: `mark_all_read`, `mark_as_read`, `notification_card` (`item_id`), `tab_*`; `notification_deleted`; `notification_cta_clicked` (automation guide §11) |
| Detail (`announcement_not_details`, detail sheet) | `notification_detail_viewed`, `contact_us_telegram` | CTA click with `notification_id`, `cta` |

### 4 A/C/G shipped (T-0064)

Built in T-0064 on branch `feat/tracking-home-discovery`, **not yet merged**. Collection only; the app flow is unchanged and legacy events still dual-fire. Handler-by-handler map (which tap fires what): [Tasks/3-in-progress/App-tracking-home-discovery.md](../../../../Tasks/3-in-progress/App-tracking-home-discovery.md) "Handler map". Code: `lib/analytics/discovery_tracking.dart`.

**Where the ids live.** Element and overlay ids are constants in `DiscoveryIds` (`discovery_tracking.dart`), **not** in `TrackingRegistry` (which has no element table today); this avoids merge conflicts with parallel tracking branches. The rows for backend `ref_app_screen` / `ref_app_element` must be taken from `DiscoveryIds.all` (elements) and `DiscoveryIds.allOverlays` (overlays), plus the screen ids in the handler map (`splash_ad`, `no_internet`, `server_error`, `force_update`, `home`, `home_search`, `see_more`, `banner_detail`, `notifications`, ...). This spec does not repeat the list (one fact, one home). `item_id` accepts only `[A-Za-z0-9_-]{1,40}` (entity ids, server codes); labels, coupon codes, names, URLs and notification text are never sent. `position` is 1-based, like search `rank`.

**Element groups in `DiscoveryIds`:** launch (`splash_ad_cta`, `splash_ad_skip`, `update_now`, `update_later`, `retry_connection`, `retry_server`); home; bottom tabs (`tab_home`, `tab_bookings`, `tab_notifications`, `tab_chat`, `tab_profile`, `tab_locked_guest`); banner detail (`banner_cta`, `banner_image_zoom`, coupon, terms, payment-method and receipt controls); notifications (`notification_card`, `announcement_card`, `mark_all_read*`, `tab_notifications_list`, `tab_announcements`, refresh, permission-banner and announcement sheet controls).

**Overlay names** (`overlay_viewed`): `language_picker`, `promo_popup`, `lucky_draw`, `coming_soon`, `guest_sign_in_prompt`, `banner_payment_methods`, `banner_aba_transfer`, `receipt_source`, `mark_all_read`, `announcement_detail`, `force_update`, `optional_update`. User-opened sheets use `overlaySettings` (`trigger=user`); auto and push popups call `DiscoveryTracking.overlay(trigger: auto|push)` because `overlaySettings` cannot carry `trigger`.

**`app_error` groups (`endpoint_group`):**

| `error_type` | `endpoint_group` | Raised from | Note |
|---|---|---|---|
| `api` | `home_page` | home API failure after load, refresh and retry | |
| `api` | `user_info` | splash, user-info failure branch | `screen_name` is `splash` |
| `network` | `no_internet` | splash, no-internet branch | `screen_name` is `splash`, not `no_internet`: the new screen is not yet on top |
| `update` | `app_version` | update sheet shown (once per variant per run) | `fatal=1` when the update is forced |

**Search** (`home_service_search_screen`; no navigation reaches it today, wired so it reports when the entry returns). `search_performed` fires 600 ms after typing stops, or at once on submit; the same text fires once; `result_count` may be 0. `search_result_clicked` carries `rank` (1-based), `query`, `service_id`.

**Notifications.** `notification_inbox_viewed` (`unread_count`) when the Notifications tab opens. `notification_detail_viewed` with `source=inbox` on an announcement card tap. Tab labels fire `element_clicked tab_notifications_list` / `tab_announcements` (taps only, not swipes). The CTA in the announcement sheet and detail screen is `element_clicked announcement_cta`.

**Tabs.** Bottom tab taps fire `element_clicked` `tab_home | tab_bookings | tab_notifications | tab_chat | tab_profile` (`element_type=tab`, id of the tab being left) beside the legacy `tracking_pages`; `screen_viewed entry_type=tab` is unchanged (from `ScreenTracker`). A guest tap on a locked tab fires `tab_locked_guest` (`target` = `tab_*`) before the sign-in push.

**Not built (widget or control does not exist):**

| Planned | Why |
|---|---|
| `search_bar`, `notification_bell`, `top_up_card`, `history_card`, `greeting_avatar` | widgets commented out or removed in `home_screen.dart` |
| `notification_deleted` | no delete or swipe-to-delete handler |
| `notification_cta_clicked` | would be a new event name and inbox items carry no `send_id`; use `element_clicked announcement_cta` |
| `mark_as_read` per item | not a separate control; a card tap marks read and is `notification_card` |
| `banner_pdf_open` | PDF screen referenced only in commented code |
| `banner_share` | no share control exists |
| `overlay_viewed` for inline offer and Telegram banners | `custom_notification_offer_banner` and `custom_telegram_banner` are cards, not popups; clicks are tracked (`notif_banner_*`, `join_telegram`) |

`banner_image_zoom` fires only on `BannerDetailComponent` (the other screen's tap does nothing).

### H. Profile and settings (task App-tracking-account-profile-chat)
| Screen | Have | Add |
|---|---|---|
| Profile (`profile_screen`, 13 handlers) | `profile_screen`, notification banner events | **Built (T-0067):** `element_clicked` `menu_profile_info`, `menu_my_location`, `menu_my_coupons`, `menu_language`, `menu_share_app`, `menu_help_center`, `menu_contact_support`, `menu_about`, `menu_logout` (+ `overlay_viewed logout_confirm`, `logout_confirm`). **Not built:** `menu_payment_methods`, `menu_notifications`, `menu_terms`, `menu_chat` (rows are commented out or absent; Terms is `help_terms` under Help center); `logout_clicked` not added |
| Profile info, avatar, change password | `profile_info_screen` | **Built (T-0067):** `form_submitted(profile_info)` (`field_count=5`; result inferred from `userInfo` being replaced), `change_avatar` + avatar/gender/DOB/delete-account controls, `permission_result` photos/camera. **Not built:** change password and change avatar screens (dead code, no route reaches them) |
| Location list/edit/search (`location_screen`, `new_location_screen`, `new_search_location`) | `location_screen` | `address_*`, `permission_result(location)`, `search_performed` for place search (no text beyond 30 chars; place query is not a customer address), `element_clicked` `use_current_location` (`set_default` not built) |
| Language | `language_selection` on Home | **Built (T-0067):** `setting_changed(language)` + user property `app_language`, see §4 J2 |
| Payment methods, My coupons | `notification_banner_not_now`, `turn_on_notifications_click` | My coupons: `refresh_coupons`, `permission_result push` + `push_enabled` (built T-0067). Payment methods screen is dead code (not tracked); `coupon_*` events not built here |
| Help centre, tutorials, about | – | **Built (T-0067):** `element_clicked` per row (`help_*`, `about_*`, `tutorial_*`; `item_id` = video id). **Not built:** tutorial `screen_viewed` with `item_id` (the observer cannot carry it; `tutorial_card` carries the id instead) |

### I. Chat (task App-tracking-account-profile-chat)
`chat_feature/views/*` (50 handlers). **Built in T-0067** (branch `feat/tracking-account-profile-chat`, not yet merged): `chat_opened` (`chat_kind`, `entry`), `chat_message_sent` (`message_type` only), and `element_clicked` for chat list, room, story camera, media preview and gallery. Media `chat_message_sent` fires at the Continue tap, text when the MQTT publish is called. No message content or file names. **Not built:** `app_error error_type=network` for MQTT/socket failures and the media upload outcome (code is in `chat_feature/sockets/*` and `message_controller.dart`, outside T-0067; needs its own task); `message_screen.dart` is an unused placeholder.

### J. Top-up (task App-tracking-booking-checkout)
**Built (T-0065):** `top_up_screen`, history, status and ABA payment screen: `topup_started` (screen open), `topup_amount_selected` (`source` preset or typed, `position`), `topup_payment_started` (amount, method code, `value`, `currency`), `element_clicked` (`topup_amount`, `topup_confirm`, `select_payment_method`, `aba_transfer_submit`, `aba_transfer_close`, `topup_status_done`, `topup_history_item`, `topup_new`, `topup_from_balance`, `back`), overlays `topup_payment_sheet`, `topup_aba_transfer`, `topup_detail`, `topup_low_balance`, and `app_error` (`topup_amounts`, `topup_create`, `topup_webview`, `topup_status`). Completion and failure stay **backend-owned** (`topup_completed`, `topup_failed`); the app does not send them. Meta/AppsFlyer: none (not a service purchase) unless marketing asks (O-5). **Not built:** the keypad sheet, `TopUpNote` and the ABA-transfer receipt upload (dead code); `checkTopUpStatus` failures (polled every 5 s, would flood).

### 4 D/E/J shipped (T-0065)

Built in T-0065 on branch `feat/tracking-booking-checkout`, **not yet merged**. Collection only: no screen, navigation, request or amount changed; existing contract events keep name and params (additions only). Handler-by-handler map: [Tasks/3-in-progress/App-tracking-booking-checkout.md](../../../../Tasks/3-in-progress/App-tracking-booking-checkout.md) "Handler map". Code: `lib/analytics/trackers/booking_tracking.dart`.

**Where the ids live.** Element, overlay and `endpoint_group` ids are constants in `BookingIds` (`booking_tracking.dart`), not in `TrackingRegistry`. Backend `ref_app_element` rows must come from `BookingIds.all` (elements) and `BookingIds.allOverlays` (overlays), like `DiscoveryIds` and `ProfileIds`; `BookingIds.allGroups` lists the `endpoint_group` values. This spec does not repeat the list (one fact, one home).

**New events.** `topup_started`, `topup_amount_selected`, `topup_payment_started`, `coupon_list_viewed` (see §3.4). `instant_hours` is added to `checkout_started` and `payment_started` for Instant services only, taken from the last `instant_duration_selected`; it is absent for scheduled services and when the duration is unknown (contract: [Instant_Cleaning_Tracking_Spec.md](Instant_Cleaning_Tracking_Spec.md)).

**Overlay names.** `payment_sheet`, `schedule_sheet`, `service_addons`, `pair_service_sheet`, `cross_sell_sheet`, `task_info`, `equipment_included`, `unavailable_schedule`, `sub_category_detail`, `payment_result_success`, `payment_result_failed`, `topup_payment_sheet`, `topup_aba_transfer`, `topup_detail`, `topup_low_balance`.

**`app_error` groups (`endpoint_group`):** `service_options`, `order_preview`, `coupon_apply`, `order_schedule`, `payment_methods`, `order_create`, `payment_status`, `payment_webview`, `topup_amounts`, `topup_create`, `topup_webview`, `topup_status`. `error_type` is `api`, `network`, `payment` or `webview`. `error_code` is a short snake_case code (backend error code, `load_<code>`, `invalid_url`, `timeout`, `app_launch_failed`), never a message, URL or amount. `order_schedule` uses `no_slots` only; the unavailable-schedule sheet is informational (`overlay_viewed` `unavailable_schedule`), and Instant fully booked is `instant_fully_booked_viewed`.

**Shipped differences from the plan above.** `search_coupon` is one click on the first keystroke and carries no text. `view_order` is not sent. The payment result overlay is split into success and failed. Cross-sell, task-info and equipment sheets use a manual `overlay_viewed` (their `show()` helpers are outside the task's files).

**Not built (widget or control unreachable or out of scope):** `PaymentBottomSheet` (`custom_payment_sheet`, `custom_payment_tile`), `CustomScheduleDateSelector`, `showScheduleTimePicker`, `CustomInstantTimeSelector`, `AppointmentConfirmScreen`, `AppointmentStatusScreen` and `VerifyOrderDetailScreen`, `TaskInfoDetailScreen`, top-up keypad and `TopUpNote`, ABA-transfer receipt upload; `LuckyDrawScreen` / `ChineseNewYearScroll` (seasonal campaign; entry overlay `lucky_draw` is T-0064); `coupon_copied` / `coupon_use_clicked` (Profile coupons).

### J2. Cross-screen controls: language switch and the top-right customer-service (CS) button
Built in T-0067 (branch `feat/tracking-account-profile-chat`, not yet merged). Handler-by-handler map: [Tasks/3-in-progress/App-tracking-account-profile-chat.md](../../../../Tasks/3-in-progress/App-tracking-account-profile-chat.md) "Handler map" and "Customer-service (CS) button: every screen".

**Where the ids live.** Profile, help, about, tutorial, customer-source, support-sheet and chat element and overlay ids are constants in `ProfileIds` (`lib/analytics/profile_tracking.dart`), not in `TrackingRegistry`. Backend `ref_app_element` rows must come from `ProfileIds.all` (elements) and `ProfileIds.allOverlays` (overlays), like `DiscoveryIds`. This spec does not repeat the list.

**Language.** `LocalizationController.selectLanguage` serves both pickers (Home sheet, Profile > Language) and fires `setting_changed` (`setting_id=language`, `setting_value`, `previous_value`, `source`) only when the language really changed. `setting_value` uses the app's real codes `en|km|vi|zh-cn|zh-tw`, else `other`. `source` is the closed set `home_sheet|profile|default_fallback|other` (`default_fallback` = language switched off in the admin portal; the silent sync after sign-in only corrects the server's stored language and sends no `setting_changed`). The `app_language` user property is set in `_setLocale`, so it covers app start, picker, server sync and fallback. The Language screen option also fires `element_clicked language_option`.

**CS button.** `overlay_viewed contact_support` fires when `CustomContactSupportSheet.show` opens (so Sign in, OTP, Sign up and Customer source get it) and from the Profile contact-support action sheet. Channel taps fire `support_contacted` (`channel=telegram|phone|email`, `source_screen`, `element_id` `contact_telegram|contact_phone|contact_email`); the Profile Help center email row fires `support_contacted channel=email`. `contact_us_telegram` is kept (dual-fire). About-screen links are social links (`about_*`), not `support_contacted`.

| Screen | `element_clicked contact_support` |
|---|---|
| Sign in, OTP, Sign up (`account/new_auth/*`) | yes (T-0074) |
| Customer source | yes (T-0067) |
| Profile "Contact supports" row | yes, as `menu_contact_support` (`target=sheet`) |
| See more (Home) | different id `DiscoveryIds.contactSupport` + `support_contacted` (T-0064) |
| Order detail v2 | yes, T-0066: `contact_support` (app bar) and `contact_us` (bottom), both also send `support_contacted channel=chat` with `order_id`. The unreachable v1 screen has no CS button in use |
| Telegram links in `category_building_screen.dart` and `confirm_and_pay_checkout_screen.dart` | **no**; follow-up for T-0065: `support_contacted channel=telegram` |

The order detail follow-up is done (T-0066); the Telegram follow-up stays with T-0065. Sign in / OTP / Sign up could pass `screen:` explicitly (works today through the screen on top).

**Not built in T-0067.** `app_error` for MQTT/socket and media upload outcome; `has_ordered` (belongs to the order/booking slices); `location_permission` (T-0077); tutorial `screen_viewed` with `item_id`; the dead screens `change_avatar_profile_screen`, `change_password_screen`, `payment_method_screen`; Profile rows `menu_payment_methods`, `menu_notifications`, `menu_terms`, `menu_chat`.

### K. Shared widgets and system (assigned to the slice that owns the screen using them)
`custom_show_dialog` (21 handlers, 3 calls), `custom_banner_list`, `custom_contact_support_sheet`, `custom_telegram_banner`, `custom_announcement_detail_sheet`, `custom_promo_overlay`, `custom_notification_offer_banner`, `custom_category_selection_alert_sheet` (see `docs/category-selection-alert-sheet-trigger.md`): each fires `overlay_viewed` once from the widget with a `trigger`, and `element_clicked` with the caller's `screen_name`. Shared widgets receive `screen_name` as an optional constructor argument or read it from `ScreenTrackingObserver.current`; the foundation task decides which (default: observer).

---

## 5. Platform assignment

### 5.1 Meta (App Events)

Existing 7 stay as is (verification in T-0031 / [Meta_App_Events_Setup.md](Meta_App_Events_Setup.md)). **Built in T-0069** (branch `feat/tracking-meta-appsflyer`, not yet merged), next to the Firebase call, fire-and-forget, never throwing. Meta SDK stays initialised after the ATT prompt as before.

| Meta event | Fires from | Params |
|---|---|---|
| `Search` (`fb_mobile_search`) | `home_service_search_screen` `_reportSearch`, service search only | `search_string` (lower-cased, ≤30; omitted when the text looks like a phone number or email), `content_type=service` |
| `AddToCart` (`fb_mobile_add_to_cart`) | `CheckoutTracker.detailsCompleted`, once per checkout screen. It marks address and slot ready, **not** a cart add; marketing must know this | `content_id` (service id), `value`, `currency=USD` |
| `Schedule` | `CheckoutTracker.slotSelected`, first slot per checkout screen only (Firebase `slot_selected` still fires on every change) | `content_id` (service id), `value`, `currency=USD` |
| `Contact` | `DiscoveryTracking.support`, not for push taps (`source=push`) | `content_type=channel` plus `channel` (`telegram`/`phone`/`chat`/`email`); no order id |

Gates: these four new events are sent from release builds only (`Helper.kMode`) and never for staff phones (`PhoneNumbers.instance.isStaff`); the existing 7 events are unchanged and ungated. Meta and AppsFlyer see `InitiateCheckout` before `AddToCart` (spec mapping; marketing should know).

**Not added:** `content_category=instant` (Instant spec §3); place or address search (privacy, location typing; only the service search is sent); any change to the existing 7 events; any change to `CompleteRegistration` timing or the consent/ATT gates; screens, errors, settings, top-up (no ad-optimisation value, O-5). `Lead` and `Subscribe` are not proposed.

### 5.2 AppsFlyer

**Built in T-0069** (not yet merged) through the new generic `AppFlyerHelper.logEvent`. Purpose: in-app funnel per campaign cohort and retention by media source. Revenue stays on `af_purchase` only (`af_revenue`); funnel events carry `af_price`, never `af_revenue`. `af_purchase` and `sign_up`/`CompleteRegistration` timing are unchanged. `AppsFlyer` keeps `timeToWaitForATTUserAuthorization`. Same gates as §5.1 (new events: release builds only and non-staff; `af_purchase` unchanged). Events before SDK init are dropped.

| AppsFlyer event | Fires from | Params |
|---|---|---|
| `af_content_view` | `category_building_screen` `_trackViewContent` | `af_content_id`, `af_content`, `af_content_type=service` |
| `af_search` | `home_service_search_screen` `_reportSearch` (service search only) | search string (lower-cased, ≤30; omitted when it looks like a phone number or email), `content_type=service` |
| `af_add_to_cart` | `CheckoutTracker.detailsCompleted`, once per checkout screen | service id, `af_price`, USD |
| `af_initiated_checkout` | `confirm_and_pay_checkout_screen` `_trackInitiateCheckout` | service id/name, `af_price`, USD |
| `af_add_payment_info` | same file `_trackAddPaymentInfo` | service id/name, `af_price`, USD |
| `af_schedule` | `CheckoutTracker.slotSelected`, first slot per checkout screen | service id, `af_price`, USD |
| `af_contact` | `DiscoveryTracking.support`, not for push taps | `content_type=channel`, `channel`; no order id |
| `af_complete_registration` | `auth_controller` account creation (same spot as Meta `CompleteRegistration`) | `af_registration_method` (`phone`/`telegram`) |
| `af_login` | `auth_controller` OTP success when `fromRegistration == false` | none |

**Not added:** `af_revenue` on funnel events; place/address search; any change to `af_purchase`. `af_login` fires on every successful OTP login of an existing account (no Firebase login event exists); keep or drop is an owner decision.

**Open (G-14):** the existing 7 Meta events and `af_purchase` have no staff exclusion and no release-only gate (only Firebase is guarded by `_sendEnabled`). Left unchanged in T-0069; owner decides whether to apply the exclusion to them.

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
- **O-7** Apply staff exclusion (and release-only?) to the existing 7 Meta events and `af_purchase` (G-14)? *Recommend staff exclusion yes; release-only as for Firebase.* Also open from T-0069: keep `af_login`; `Schedule` once per screen vs every slot change; `Contact` param shape.
