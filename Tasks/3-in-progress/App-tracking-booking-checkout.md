# App tracking booking checkout

- **ID:** T-0065
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-booking-checkout/` · **Branch:** `feat/tracking-booking-checkout`
- **Tier:** T3 · booking/checkout/payment/top-up events the backend maps; money-adjacent flow (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-03
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 D, E, J
- **Depends on:** T-0082 merged first (owns the Instant parts of `category_building_screen.dart`); T-0063; T-0074 and T-0077 merged first (they own the guest sign-in gate and the address region of `confirm_and_pay_checkout_screen.dart`)
- **Touches:** `lib/view/screen/home/category_building/**`, `lib/view/screen/home/top_up/**`, `lib/view/screen/home/web_view/payment_aba_webview_screen.dart`, `lib/view/custom/{custom_payment_*,custom_schedule_*,custom_service_addon_group,custom_instant_*,custom_unavailable_schedule_sheet,custom_apply_coupon_section,custom_coupon_card}.dart`, `lib/analytics/trackers/*`, `lib/controller/payment_controller.dart` (tracking lines only)

## Goal
Instrument spec §4 D (service and booking), E (checkout and payment) and J (top-up): element clicks, overlays (policy, schedule, payment result), `app_error` on price/coupon/payment failures, `topup_*` funnel events, `coupon_*`. Do not change money math or existing contract events.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [ ] G6 security  - [x] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [x] No existing contract event changes name or params (diff shows additions only)
- [x] Checkout/payment errors emit `app_error` with `endpoint_group`, never URLs or amounts of other users
- [x] Top-up funnel events fire; completion stays backend-owned
- [x] Unreachable screens (`lucky_draw*`, `appointment_status_*` duplicates) confirmed and skipped or listed
- [ ] DebugView matrix recorded; `EventContract` tests updated
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress (3 Nov 2026)

Built on `feat/tracking-booking-checkout` (cut from `feat/tracking-integration` 01786197). Collection only: no screen, navigation, request, request order or amount changed; every call is fire-and-forget through `DiscoveryTracking` / `BookingTracking` / `AnalyticsHelper`. `fvm flutter analyze --no-pub` clean; `fvm flutter test` 233 pass, only `test/widget_test.dart` fails (fails on main too). New tests: `test/booking_checkout_tracking_test.dart` (27).

New code: `lib/analytics/trackers/booking_tracking.dart` (`BookingIds` element, overlay and `endpoint_group` ids; `BookingTracking.error/safeCode/amountFromText`); `AnalyticsHelper.trackTopupStarted/AmountSelected/PaymentStarted`, `trackCouponListViewed`; `instant_hours` on `checkout_started` and `payment_started` (remembered per service from `instant_duration_selected`, sent only for Instant services with a known duration); `EventContract` rows for `topup_amount_selected` and `topup_payment_started`.

### Handler map

| Where | Handler | Event |
|---|---|---|
| category_building_screen | back (app bars) `_goBack` | `element_clicked` `back` |
| category_building_screen | CS icon `_contactSupport` | `element_clicked` `contact_support` (target telegram) + `support_contacted` telegram; legacy `contact_us_telegram` kept |
| category_building_screen | primary CTA, non-qty and qty "Next" | `element_clicked` `schedule_appointment` (fires on every tap, also when the button only scrolls because options are missing) |
| category_building_screen | primary CTA, qty "Add" | `element_clicked` `add_to_booking` |
| category_building_screen | task information row `showCleaningInstructions` | `element_clicked` `showcase_task_info` + `overlay_viewed` `task_info` (manual: sheet is in a file outside Touches) |
| category_building_screen | included/equipment row `buttonSheetIncluded` | `element_clicked` `equipment_included` + `overlay_viewed` `equipment_included` |
| category_building_screen | retry (options failed) | `element_clicked` `retry`; `app_error` api `service_options` when the product call fails (both load paths) |
| category_building_screen | add-on sheet | `overlay_viewed` `service_addons` (routeSettings); Add button `addon_done` click |
| category_building_screen | fully-booked sheet (via `CustomUnavailableScheduleSheet.show`) | `overlay_viewed` `unavailable_schedule` only (informational sheet, no `app_error`) |
| product_option_selector | "view more" toggle | `element_clicked` `show_all_options` |
| category_product_selector | sub-category chip | `element_clicked` `sub_category_selected` (item_id = product id, position) |
| sub_category_screen | info icon, detail sheet, close | `element_clicked` `option_info`, `overlay_viewed` `sub_category_detail`, `sheet_close` |
| confirm_and_pay_checkout_screen | apply coupon row `_onApplyCouponTap` | `element_clicked` `apply_coupon` |
| confirm_and_pay_checkout_screen | "use other promo" (alternate offer) | `element_clicked` `use_other_promo` |
| confirm_and_pay_checkout_screen | refresh price | `element_clicked` `refresh_price` |
| confirm_and_pay_checkout_screen | footer button | `element_clicked` `confirm_pay` (signed in) or `sign_up_gate` (guest); legacy `begin_checkout`, `add_payment_info` untouched |
| confirm_and_pay_checkout_screen | change schedule ("Choose") | `element_clicked` `change_schedule`; sheet = `overlay_viewed` `schedule_sheet`; empty calendar snackbar = `app_error` `order_schedule` code `no_slots` |
| confirm_and_pay_checkout_screen | read full policy | `element_clicked` `read_full_policy`; sheet already `overlay_viewed` `checkout_cancellation_policy` (T-0079) |
| confirm_and_pay_checkout_screen | cross-sell sheet and its Next | `overlay_viewed` `cross_sell_sheet` (manual, helper file outside Touches); `element_clicked` `cross_sell_next` |
| confirm_and_pay_checkout_screen | payment sheet and each method row | `overlay_viewed` `payment_sheet`; `element_clicked` `select_payment_method` (item_id = method code, position) |
| confirm_and_pay_checkout_screen | create-order result | controller: `app_error` payment `order_create` with code and http status (also exception); existing `payment_failed` stays |
| confirm_and_pay_checkout_screen | pair-service sheet, cancel, Add/Next | `overlay_viewed` `pair_service_sheet`; `element_clicked` `pair_sheet_cancel`, `pair_sheet_confirm`; pair events already existed |
| confirm_and_pay_checkout_screen | app bar back, CS icon | `back`; `contact_support` + `support_contacted` telegram |
| confirm_and_pay_checkout_screen | ABA app launch fails (`openOtherApp`) | `app_error` payment `payment_webview` code `app_launch_failed` / `app_launch_error` |
| checkout_booking_schedule_sheet | close, Confirm | `schedule_close`, `schedule_confirm` clicks (screen `schedule_sheet`); date/time/slot events still come from `slot_selected` |
| apply_coupon_list_screen | screen open | `coupon_list_viewed` (service, count) after the list loads |
| apply_coupon_list_screen | Apply, back, search box (first keystroke only), coupon card | `apply_coupon`, `back`, `search_coupon`, `coupon_card` (item_id = coupon id, position); never the code or search text |
| apply_coupon_list_screen | notification offer Not now / Open settings | `notif_banner_not_now`, `notif_banner_open_settings` (T-0064 ids) |
| payment_controller | bulk preview, coupon apply, schedule, payment methods, create-order failures | `app_error` `order_preview`, `coupon_apply`, `order_schedule`, `payment_methods`, `order_create` (api / network / payment; code snake_case, no URL or message) |
| custom_payment_result_sheet | success and failure sheets | `overlay_viewed` `payment_result_success` / `payment_result_failed`; clicks `back_home`, `try_again`, `result_close` |
| custom_unavailable_schedule_sheet | got it, close | `sheet_close` |
| payment_aba_webview_screen | back (app bar and system back) | `back` |
| payment_aba_webview_screen | load error, invalid URL, final non-paid status, 3-minute timeout | `app_error` webview `payment_webview` (`load_<code>`, `invalid_url`); payment `payment_status` (status code, `timeout`) |
| appointment_status_v2_screen | Done button | `back_home` |
| appointment_status_screen_v3 | close, Start booking | `back`, `start_booking` |
| sub_topup_point_screen (low balance sheet) | Top up, Cancel | `topup_from_balance`, `sheet_close`; `overlay_viewed` `topup_low_balance` (routeSettings in payment_controller) |
| top_up_screen | open | `topup_started` |
| top_up_screen | amount chip | `topup_amount_selected` (source preset, position) + `element_clicked` `topup_amount` |
| top_up_screen | Top up button | `element_clicked` `topup_confirm`; `topup_amount_selected` source typed when no chip is selected |
| top_up_screen | payment sheet, method row | `overlay_viewed` `topup_payment_sheet`; `topup_payment_started` (amount, method code, value, currency) + `select_payment_method` click |
| top_up_screen | ABA transfer sheet, close, submit | `overlay_viewed` `topup_aba_transfer`; `aba_transfer_close`, `aba_transfer_submit` |
| top_up_screen | ABA app launch fails | `app_error` payment `topup_webview` |
| top_up_screen | back | `back` |
| top_up_controller | amount list or create-payment fails | `app_error` api `topup_amounts`; payment `topup_create` |
| top_up_aba_payment_screen | web load error, invalid URL, 3-minute timeout, back | `app_error` `topup_webview` / `topup_status`; `back` |
| top_up_status_screen | Done / pending button | `topup_status_done` |
| top_up_history_screen | row, detail sheet, close, add button, back | `topup_history_item` (item_id = id), `overlay_viewed` `topup_detail`, `sheet_close`, `topup_new`, `back` |

### Intentionally untracked

- Covered by an existing event: product option tap (`service_option_selected`, `duration_selected`), add-on plus / minus / remove / quantity (`addon_*`), increase / decrease quantity (legacy), pair-service select / remove / add (`pair_service_*`), "Need more than 2 hours?" and rooms section (T-0082 `instant_upsell_clicked`, `instant_room_toggled`), address row and picker (T-0077 `address_selected`), guest sign-in gate lines (T-0074), cancellation policy sheet contents (T-0079), banner carousel on the coupon list (T-0064).
- Inside the schedule sheet: week calendar, date selector and slot chips (the choice is reported on Confirm by `date_selected`, `time_selected`, `slot_selected`).
- Noise or no decision: pull-to-refresh, text fields (note, coupon search text, top-up amount keystrokes), scroll controllers, keyboard dismiss, loading dialogs, snackbars.
- Dead code, not wired: `PaymentBottomSheet` (`custom_payment_sheet.dart`, `custom_payment_tile.dart`), `CustomScheduleDateSelector`, `showScheduleTimePicker`, `CustomInstantTimeSelector`, `AppointmentConfirmScreen`, `AppointmentStatusScreen` and `VerifyOrderDetailScreen` (only reach each other), `TaskInfoDetailScreen`, top-up `buildButtonSheet` keypad, `_buildInputAmount`, `TopUpNote` (commented out), ABA-transfer receipt upload (`isSubmitEnabled` is a constant false).
- Reachable but skipped: `LuckyDrawScreen` / `ChineseNewYearScroll` (seasonal campaign outside the booking funnel; T-0064 already reports its entry as `overlay_viewed` `lucky_draw`).
- `checkTopUpStatus` failures (polled every 5 s, would flood); the 3-minute timeout is reported instead.
- Meta / AppsFlyer timing: T-0069.

### Differences from the spec

- Payment result overlay is split: `payment_result_success` and `payment_result_failed` instead of one `payment_result`, so the outcome is visible without joining other events.
- `view_order` is not sent: the success sheet has no "view order" button (Done goes home, tracked as `back_home`).
- `search_coupon` is one click on the first keystroke, not per search; no text is sent.
- `coupon_copied` / `coupon_use_clicked` belong to Profile coupons, not built here.
- `app_error` for "no slots" uses error_code `no_slots` (empty calendar) only; the unavailable sheet is informational (`overlay_viewed` `unavailable_schedule`) and Instant fully-booked is T-0082's `instant_fully_booked_viewed`.
- Cross-sell, task-info and equipment sheets are reported with a manual `overlay_viewed` because their `show()` helpers live outside Touches; every other sheet uses `overlaySettings`.
- Tracking lines were added to `lib/controller/top_up_controller.dart` and `lib/view/custom/custom_unavailable_schedule_sheet.dart`, which are not in `Touches` (the top-up controller swallows failures, so the screen cannot see them). No logic changed.
- `instant_hours` is derived inside `AnalyticsHelper` from the last `instant_duration_selected`, not passed by the checkout screen, so the checkout code did not change for it.

### Still open

- G4: Firebase DebugView / device matrix (Android + iOS, dev backend): service detail to payment sheet (wallet, KHQR deeplink, card, cash), coupon list, schedule sheet, payment failure and webview timeout, top-up preset and typed amount, Instant service for `instant_hours`.
- G6 security, G7 docs sync (Instant spec note "instant_hours not sent yet" is now outdated; coverage spec rows D, E, J; backend must accept `topup_*`, `coupon_list_viewed`, new `endpoint_group` values), G8 PR message, G9 human review, G10 release plan, G11 changelog entry.
- Done-when still open: DebugView matrix recorded; root `CHANGELOG.md`; worktree removal after merge.
- Owner decisions: (1) keep the success/failure overlay split; (2) accept tracking lines in the two files outside `Touches`; (3) whether Lucky Draw needs its own slice.

## Review round 1 (fresh agent, 3 Nov 2026)

Fixed
- Webview errors (payment and top-up ABA): one `app_error` per screen (`_errorTracked`), main-frame only (`isForMainFrame` already checked), and codes -10, -1002, -999 (custom-scheme redirects, cancelled) are skipped. Flow unchanged.
- `schedule_unavailable` removed: `CustomUnavailableScheduleSheet.show` no longer fires `app_error`; only `overlay_viewed unavailable_schedule` remains. Docs fixed (coverage spec, backend handoff, this task). `no_slots` kept: it is the empty calendar after a successful load, a real dead end the customer sees.
- `_instantHours` cleared in `setBookingMode` and in `trackOrderCreated`; two tests added. Note: `setBookingMode` runs after the first product selection in `category_building_screen`; a duration auto-picked before that call would be dropped (device matrix covers this).
- `getOrderSchedule` uses a local `code` from this response only.
- `safeCode` collapses repeated underscores; tests for negative and empty code.
- `coupon_list_viewed` only sent when the load succeeded (`getApplyCouponList` now returns a bool; existing callers ignore it).
- Typed `topup_amount_selected` de-duplicated on the last sent amount.
- Comment on `topupMethod` (duplicates `selectPaymentMethod`, stays out of `all`).

Accepted: `payment_failed` plus `app_error` for the same failure; test layer is helper-level.

Left: tests cover the helper layer; webview, controllers and widgets are covered by the device matrix (G4).

After fix: `fvm flutter analyze --no-pub` clean; `fvm flutter test` all pass except the known `test/widget_test.dart`.
