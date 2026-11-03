# App cancel reschedule tracking

- **ID:** T-0079
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-cancel-reschedule-tracking/` · **Branch:** `feat/cancel-reschedule-tracking`
- **Tier:** T3 · tracking events and request fields the backend depends on; order state flows (tracking lines only) (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-03
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Cancellation_And_Reschedule_Spec.md` §1, §2
- **Depends on:** T-0063; T-0080 (agreed field names for the optional request fields); product sign-off O-11
- **Touches:** `lib/view/screen/activity/order_detail_screen_v2.dart` (cancel and reschedule methods only), `lib/view/screen/activity/order_detail/widgets/{order_detail_cancel_sheet,order_detail_reschedule_sheet,order_detail_reschedule_cancel_card,order_detail_bottom_actions}.dart`, `lib/view/screen/activity/order_detail/order_detail_reschedule_logic.dart`, `lib/controller/payment_controller.dart` (`paymentCancelOrder`, `rescheduleOrder`, `requestRescheduleGrace`), `lib/data/repository/payment_repository.dart` (optional body fields), `lib/view/screen/home/category_building/checkout_cancellation_policy_sheet.dart`, `lib/analytics/analytics_helper.dart` (only the methods listed in the foundation task), `test/`

## Goal
Collect cancellation and reschedule data correctly with **no change to what the app shows or does**. Add the events in spec §2.1 and §2.2, send the cancel reason and source in the existing cancel call as optional fields, stop sending the customer's free text to Firebase, never send placeholder fee amounts. Order detail v1: confirm reachability; if dead, remove its duplicate `reschedule_button_*` events.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [ ] G6 security  - [x] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Every step of the cancel sheet (open, reason, redirect to reschedule, policy, confirm, result, dismiss) and reschedule sheet (open, slot, grace result, confirm, result, dismiss) fires the right event once
- [ ] `cancellation_result` / `reschedule_completed` / `reschedule_failed` fire after the API returns with the backend error code; the intent events fire before
- [ ] Cancel request body carries `reasonCode`, `reasonText`, `source` and old backends still return success; edit-schedule carries `source`, `entry`
- [x] Firebase never receives free text or a placeholder fee: a test asserts the allow-list and the 100-char limit
- [ ] Screens, navigation, copy and amounts shown are identical to before (screenshot comparison in the PR)
- [ ] `order_id` equals `bulkOrderId` everywhere; `service_id` is a plain param, not the `serviceId:` argument
- [ ] DebugView matrix: customer cancel success, cancel API failure, dismissed at reason and confirm, cancel-to-reschedule, reschedule success, slot unavailable, reschedule failure
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress (3 Nov 2026)

Branch `feat/cancel-reschedule-tracking`, nothing committed. `fvm flutter analyze --no-pub`: no issues. `fvm flutter test`: 164 pass, only `test/widget_test.dart` fails (fails on main too). New tests: `test/cancel_reschedule_tracking_test.dart` (helpers, builders, allow-list, 100-char limit, no free text, contract validation, cancel sheet widget test).

### Handler map

| Where | Handler | Event |
|---|---|---|
| `order_detail_screen_v2.dart` `_openCancelSheet` | cancel sheet opens | `cancellation_started` (entry `order_detail`, `hours_to_start`, `order_status`, `value`, `currency`, `partner_assigned`) + `overlay_viewed` (`cancel_sheet`) |
| `order_detail_cancel_sheet.dart` `_selectReason` | reason row tapped | `cancellation_reason_selected` (once per change) |
| same, `_selectReason` (want_reschedule) | "I want to reschedule instead" | `cancel_to_reschedule_clicked`, then screen opens reschedule with `entry=cancel_sheet` |
| `order_detail_screen_v2.dart` `onViewDetails` / `onReadPolicy` (cancel sheet) | policy link | `cancellation_policy_viewed` (`source_screen=cancel_sheet`) |
| `order_detail_screen_v2.dart` `onConfirmCancel` | confirm tapped, before API | `cancellation_submitted` (+ legacy `cancel_booking_confirm`, free text removed) |
| `payment_controller.dart` `paymentCancelOrder` | API returned or threw | `cancellation_result` (`result`, `error_code`, `http_status`, `duration_ms`) |
| `order_detail_cancel_sheet.dart` `dispose` | sheet closed (close, swipe, back) before success | `cancellation_abandoned` (`step` reason/confirm, `reason_code` if chosen) |
| `order_detail_screen_v2.dart` `_openRescheduleSheet` (from Where&When card, reschedule card, cancel sheet) | sheet opens | `reschedule_started` (+ `entry`, `hours_to_start`, ...) + `overlay_viewed` (`reschedule_sheet`) + legacy `reschedule_button_click` |
| same, `onRequestGrace` | slot continue | `reschedule_slot_selected` (+ `shift_hours`) |
| `payment_controller.dart` `requestRescheduleGrace` | grace API returned or threw | `reschedule_grace_requested` (`ok`/`slot_unavailable`/`error`, `error_code`, `new_slot`); not fired when a still-valid cached token is reused |
| `order_detail_screen_v2.dart` `onConfirm` | confirm tapped, before API | `reschedule_submitted` (+ legacy `reschedule_button_confirm`) |
| `payment_controller.dart` `rescheduleOrder` | API returned or threw | `reschedule_completed` (+ legacy `order_rescheduled`) / `reschedule_failed` (+ `http_status`, code sanitised) |
| `order_detail_reschedule_sheet.dart` `dispose` | sheet closed before success | `reschedule_abandoned` (`step` pick/confirm/choose_pay/payment_failed) |
| `order_detail_screen_v2.dart` `_openReschedulePolicy` | "read full policy" (card, reschedule sheet) | `cancellation_policy_viewed` (`source_screen` order_detail / reschedule_sheet) |
| `checkout_cancellation_policy_sheet.dart` `show` | checkout policy sheet | `cancellation_policy_viewed` (`source_screen=checkout`, no order_id) + `overlay_viewed` (`checkout_cancellation_policy`) |

### Intentionally untracked
- Cancel sheet: close buttons on reason/confirm/success steps, "Next", "Done", back from confirm to reason (dismissal is covered by `cancellation_abandoned`; success has `cancellation_result`).
- Reschedule sheet: day and slot chips (slot is tracked at continue via `reschedule_slot_selected`, as before), "Change" payment, payment method rows, "Try again", back steps (paid path is not reachable: `_isFreeWindow => true`; `choose_pay` and `payment_failed` are only used as `reschedule_abandoned` steps).
- `order_detail_bottom_actions.dart` (contact us, pay now, book again, leave review, swipe) and `order_detail_reschedule_cancel_card.dart` `onCancel`: not cancel/reschedule events. The cancel row in the card is commented out, so the cancel sheet is not reachable from the UI today. Pay now and review keep their legacy events in the screen.
- `cancellation_fee_shown` and `reschedule_fee_shown`: not built (placeholder fees, O-10). `cancel/status` checks: no event.

### Differences from the spec
- Request fields (`reasonCode`, `reasonText`, `source`, `entry`) and the repository: no change (owner rule, waits for T-0080 and O-11). So two Done-when boxes stay open.
- `service_id` is not sent: `bookingDetail` has no reliable service id. `reschedule_count` not sent: not in the detail. `value`/`currency` use `totalAmount`/`totalPayableAmount` with currency fixed `USD` (same as purchase tracking).
- `reschedule_completed` `is_free`/`fee_amount`: omitted (server does not say yet).
- Spec says `cancellation_abandoned` fires on dismissal without cancel; when the customer is sent to reschedule it does not fire (redirect has its own event).
- Slots in the reschedule events are now ISO `+07:00` (old values were raw `dd-MM-yyyy h:mm a` and `yyyy-MM-dd HH:mm:ss`); `failure_reason`/`error_code` are passed through `safeErrorCode` (code-shaped, max 40 chars, else `unknown`).
- Free text removed from legacy `cancel_booking_confirm` (`other` param). `reschedule_started` etc. contract entries added to `EventContract.required`.
- `order_detail_screen.dart` (v1) not touched (not in Touches); its duplicate `reschedule_button_*` events and reachability stay with T-0066.

### Still open
- G4 device matrix (DebugView): cancel success, cancel API failure, dismissed at reason and confirm, cancel-to-reschedule, reschedule success, slot unavailable, reschedule failure. Note the cancel entry is hidden in the UI, so cancel events need a temporary enable on dev or are untestable until it is shown.
- Done-when still open: right-event-once check on device, optional request fields (T-0080, O-11), screenshot comparison in the PR, service_id.
- G6 security, G8 PR message, G9 human review, G10 release plan, G11 changelog; backend whitelist for the new events (pending request 3.4).

## Review round 1 (fresh agent, 3 Nov 2026)
Fixed: (M1) a sheet dismissed (swipe or barrier tap) while the request is in flight no longer fires `*_abandoned`, so a result is never preceded by an abandon for the same attempt; (M2) `rescheduleOrder` fires one completed or failed event per attempt (a throw after success no longer adds `reschedule_failed`); (L2) backend error codes pass only if they look like codes (`UPPER_SNAKE`, or `exception` / `network` / `unknown`), so a bare word from the server cannot leak.
Accepted: (L1) `hours_to_start` uses the device clock against the ICT wall time, like the existing free-window logic: wrong by the zone difference on a phone outside +07:00 (slot strings are unaffected); (L3) a network failure is reported as `exception` without `http_status`; (L4) no `reschedule_grace_requested` when a cached grace token is reused.
Left: tests cover helpers and the cancel sheet; the controller methods and the reschedule sheet's abandon mapping are covered by review and the device matrix.
After fixes: analyze clean; 164 tests pass (only the old `widget_test.dart` fails).

## Status (3 Nov 2026)
Committed as 3fb53aa9 on `feat/cancel-reschedule-tracking`. **Not merged to `main`: the owner holds all merges until the whole app side is done; branches are merged into `feat/tracking-integration` for the combined device check (G4).** Open: G4, G6, G8-G11.
