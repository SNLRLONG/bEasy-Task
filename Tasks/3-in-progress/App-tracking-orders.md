# App tracking orders

- **ID:** T-0066
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-orders/` · **Branch:** `feat/tracking-orders`
- **Tier:** T3 · cancellation/reschedule/extend events the backend depends on (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-03
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 F
- **Depends on:** T-0063; T-0079 merged first (same folder `order_detail/*`)
- **Touches:** `lib/view/screen/activity/**`, `lib/view/custom/{custom_slidable_item,custom_show_dialog}.dart` (order-related callers only), `lib/controller/payment_controller.dart` (tracking lines outside cancel/reschedule)

## Goal
Instrument spec §4 F (orders): bookings list, order detail v2 widgets, extend sheet, review form. **Cancel and reschedule are owned by T-0079** (`Cancellation_And_Reschedule_Spec.md`) and are not touched here.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [ ] G6 security  - [x] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [x] Order detail top-right CS button (`order_detail_app_bar.dart`) fires `element_clicked` `contact_support` and `support_contacted` channel `chat` with the order id
- [x] Extend sheet fires `overlay_viewed` and its element clicks; cancel and reschedule sheets are left to T-0079
- [x] Review submit sends `form_submitted(review)` with counts only, never comment text
- [x] v1 vs v2 detail reachability is decided in T-0079; this task follows its result
- [ ] DebugView matrix recorded
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress (3 Nov 2026)

Branch `feat/tracking-orders` (from `feat/tracking-integration` 4fb989d7). Built, not committed. `fvm flutter analyze --no-pub`: no issues. `fvm flutter test`: 284 pass, only `test/widget_test.dart` fails (fails on main too). New: `test/orders_tracking_test.dart` (27 tests: ids, allow-list with forbidden keys, contract, closed sets, never-throws, observer overlays).

Collection only: no screen, navigation, request, request order, amount or return value changed. Helpers: `lib/analytics/trackers/orders_tracking.dart` (`OrdersIds`, `OrdersTracking`); taps reuse `DiscoveryTracking.click`. Own id lists, `TrackingRegistry` not touched.

### Handler map

| Where | Handler | Event |
|---|---|---|
| `booking_screen` tab labels | tap Active / History (only when the tab changes) | `element_clicked` `tab_active` / `tab_history` (type `tab`) |
| `booking_screen` order card | open detail | `element_clicked` `order_card` (type `card`, `item_id` = bulk order id) |
| `booking_screen` pull to refresh (list and empty state) | `onRefresh` | `element_clicked` `refresh_bookings` |
| `NoBookingFound` button | explore services | `element_clicked` `explore_services` |
| `PaymentController.getBooking` failure | non-true status / exception | `app_error` group `booking_list` |
| `order_detail_screen_v2` app bar back | `_goBack` | `element_clicked` `back` (type `back`, `item_id` order) |
| app bar customer-service icon | `_contactSupportFromAppBar` | `element_clicked` `contact_support` + `support_contacted` channel `chat`, `order_id`, `element_id` `contact_support` |
| bottom "contact us" | `_contactSupportFromBottom` | `element_clicked` `contact_us` + `support_contacted` channel `chat`, `order_id` |
| bottom pay now | `_payNow` | `element_clicked` `pay_now` (legacy `pay_now_order_details` kept) |
| bottom book again | `_bookAgain` | `element_clicked` `book_again` |
| bottom leave a review | `_leaveReview` | `element_clicked` `leave_review` (legacy `leave_review` kept) |
| bottom swipe to complete | `_swipeConfirmed` (drag past threshold) | `element_clicked` `swipe_complete` |
| `showCleanersNotFinishedDialog` | opens | `overlay_viewed` `cleaners_not_finished` (dialog); buttons `cleaners_not_yet`, `cleaners_done` (screen = the dialog) |
| `PaymentController.completeOrder` failure | non-true / exception | `app_error` group `order_complete` |
| services card add-on button | `_openServiceAddOn` | `element_clicked` `service_add_on` |
| add-on picker sheet (`MainServiceScreen`) | opens; item tap; close | `overlay_viewed` `addon_services`; `addon_service_item` (position), `addon_close` |
| add-on sheet (`AddOnScreen`) | opens; add / plus / minus; checkout | `overlay_viewed` `order_addon`; `addon_add`, `addon_increase`, `addon_decrease` (position = row), `addon_checkout` |
| order pay sheet (`showBottomSheetPaymentList` / `PaymentListWidget`) | opens; method row | `overlay_viewed` `order_payment_sheet`; `element_clicked` `select_payment_method` (`item_id` = method code, position) |
| instant card extend button | `onExtend` in the screen | `element_clicked` `extend_hours` (the card file itself is T-0082) |
| extend sheet | opens (T-0082 name `extend_hours`); close X (3 steps), pay amount, done | `overlay_viewed` `extend_hours`; `extend_close`, `extend_confirm`, `extend_done` (`screen_name` `extend_hours`, `section` = step); payment row stays `extend_payment_selected` (T-0082) |
| gift-coupon confirm dialog (`canUseNow`) | opens; continue; cancel | `overlay_viewed` `gift_coupon_confirm` (dialog); `gift_apply`, `gift_cancel` |
| notification banner | open settings; not now | no new click lines: the shared `CustomNotificationOfferBanner` already sends `notif_banner_open_settings` / `notif_banner_not_now` (`section: offer_banner`); legacy events kept |
| pull to refresh on detail | `_onPullRefresh` (not the resume refresh) | `element_clicked` `refresh_order` |
| `PaymentController.getBookingDetail` failure | non-true / exception | `app_error` group `order_detail` |
| `leave_review` back | back arrow | `element_clicked` `back` |
| `leave_review` stars | tap star | `element_clicked` `review_star` (position 1-5; legacy `rating_selected` kept) |
| `leave_review` tags | toggle option | `element_clicked` `review_tag` (type `chip`, position only, no tag name) |
| `leave_review` submit | tap submit | `element_clicked` `review_submit` (legacy `leave_review_submitted` kept) |
| `AuthController.submitLeaveReview` | response | `form_submitted` `review`, `result` success / error, `tags_count`, `has_comment` (0/1), `error_code` `http_<n>` / `network`; `review_submitted` (stars) unchanged |
| `review_done_screen` | close X; close button | `element_clicked` `review_done_close_icon`, `review_done_close` |

### Intentionally untracked
- Order detail v1 `order_detail_screen.dart` (2037 lines) and `cleaner_widget.dart`: **unreachable** (`OrderDetailScreen(` is constructed nowhere; all entries use `OrderDetailScreenV2`). Not instrumented and not changed; its CS button and legacy `reschedule_button_*` are dead. Delete the file in a separate cleanup (owner decision).
- Legacy `reschedule_button_click` / `reschedule_button_confirm` in `order_detail_screen_v2.dart` **kept** (dual-fire, may feed Meta/GA audiences, O-3 in the spec). Cancel and reschedule sheets and their events stay with T-0079.
- Bookings list: horizontal swipe between tabs and the hidden empty-state swipe (alternate gesture for the tabs; the page change also fires after a tab tap, so reporting it would duplicate).
- Order detail: read-only cards (id and status, where and when, partners, payment, services list, header image), the Telegram banner (shared widget, T-0064), "read full policy" and "reschedule this booking" rows (`cancellation_policy_viewed` / `reschedule_started`, T-0079), the commented-out where-and-when reschedule link and cancel row.
- Not in the UI, so nothing to track: `call_cleaner`, `chat_cleaner`, `view_cleaner`, `services_card_expand`, `where_when_edit` (spec section F lists them).
- Add-on sheets: swipe or barrier dismissal; review: comment text field; order pay sheet: dismissal and the empty "try again" text (no handler).
- `custom_slidable_item.dart`: only used by checkout (T-0065), not order related.
- Instant progress card, `instant_progress_viewed`, `extend_started` / `extend_payment_selected` / `extend_closed`: T-0082.

### Differences from the spec (section 4 F)
- Review `form_submitted` also needed typed optional `tagsCount` / `hasComment` on `trackFormSubmitted` (additive in `analytics_helper.dart`) and the tracking lines are in `AuthController.submitLeaveReview`, because the result is only known there. Tag count and comment flag are derived inside the controller; no signature change.
- `showCustomizeSheet` (`lib/view/custom/customize_bottom_sheet.dart`) and `CustomDialog.showInfoDialog2` (`custom_show_dialog.dart`) got an optional `routeSettings` so the order sheets are named overlays. Outside the stated Touches by one file, additive, no behaviour change.
- `auth_controller.dart` (review) and `analytics_helper.dart` are outside Touches (tracking lines only).
- Spec lists `confirm_new_time` / `try_again` for the reschedule sheet; left to T-0079 (its sheet file).
- Spec lists `cancellation_fee_shown`; not built (placeholder fees, see Cancellation_And_Reschedule_Spec).
- `contact_us` (bottom) and `contact_support` (app bar) are separate ids; both also send `support_contacted` (chat).
- `app_error` groups added: `booking_list`, `order_detail`, `order_complete` (`review_options` is sent by `getRatingOption` on load failure).

### Still open
- G4 device matrix (DebugView): list tabs and card, detail CS buttons, pay now sheet, add-on sheets, extend sheet, gift dialog, swipe-to-complete with and without the cleaners dialog, review submit success and failure.
- G6 security, G8 PR message, G9 human review, G10 release plan, G11 root `CHANGELOG.md`.
- Owner decisions: delete dead order detail v1; confirm `support_contacted` should also fire from the bottom contact button; backend ingest must accept `tags_count` and `has_comment` on `form_submitted`.

## Review round 1 (fresh agent, 3 Nov 2026)
Fixed
- Notification banner double count: removed `DiscoveryTracking.click` in `_handleOpenSettings` / `_handleNotNow` (the shared banner already reports them); handler map and spec section F corrected.
- `getBooking` / `getBookingDetail` / `completeOrder`: `body['code']` read is now `body is Map ? body['code'] : null`, so control flow is unchanged when body is null.
- `trackFormSubmitted`: free `extra` replaced by typed `tagsCount` / `hasComment`; `OrdersTracking.reviewForm` updated; callers cannot override `form_id` / `result` or add keys.
- `review_options`: wired `OrdersTracking.error` in `AuthController.getRatingOption` failure branch (tracking-only line, safe body read); constant kept.
- Extend sheet `section` now snake_case (`choose_pay`; `confirm`, `success` unchanged).

Accepted
- `OrdersTracking.support` calls `trackSupportContacted` directly, so order-detail support taps do not send the Meta/AppsFlyer `Contact` event (in-app order chat, not an external channel).
- A successful review sends three events (`form_submitted(review)`, legacy `review_submitted`, `leave_review_submitted`); dashboards must not sum them.
- Order detail v1 is unreachable.

Left
- Tests cover helpers; call sites and controllers are covered by the G4 device matrix.

After fix: `fvm flutter analyze --no-pub` clean; `fvm flutter test` 286 pass, only `test/widget_test.dart` fails (pre-existing).
