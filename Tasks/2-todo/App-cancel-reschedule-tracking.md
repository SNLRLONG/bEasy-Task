# App cancel reschedule tracking

- **ID:** T-0079
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-cancel-reschedule-tracking/` · **Branch:** `feat/cancel-reschedule-tracking`
- **Tier:** T3 · tracking events and request fields the backend depends on; order state flows (tracking lines only) (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Cancellation_And_Reschedule_Spec.md` §1, §2
- **Depends on:** T-0063; T-0080 (agreed field names for the optional request fields); product sign-off O-11
- **Touches:** `lib/view/screen/activity/order_detail_screen_v2.dart` (cancel and reschedule methods only), `lib/view/screen/activity/order_detail/widgets/{order_detail_cancel_sheet,order_detail_reschedule_sheet,order_detail_reschedule_cancel_card,order_detail_bottom_actions}.dart`, `lib/view/screen/activity/order_detail/order_detail_reschedule_logic.dart`, `lib/controller/payment_controller.dart` (`paymentCancelOrder`, `rescheduleOrder`, `requestRescheduleGrace`), `lib/data/repository/payment_repository.dart` (optional body fields), `lib/view/screen/home/category_building/checkout_cancellation_policy_sheet.dart`, `lib/analytics/analytics_helper.dart` (only the methods listed in the foundation task), `test/`

## Goal
Collect cancellation and reschedule data correctly with **no change to what the app shows or does**. Add the events in spec §2.1 and §2.2, send the cancel reason and source in the existing cancel call as optional fields, stop sending the customer's free text to Firebase, never send placeholder fee amounts. Order detail v1: confirm reachability; if dead, remove its duplicate `reschedule_button_*` events.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Every step of the cancel sheet (open, reason, redirect to reschedule, policy, confirm, result, dismiss) and reschedule sheet (open, slot, grace result, confirm, result, dismiss) fires the right event once
- [ ] `cancellation_result` / `reschedule_completed` / `reschedule_failed` fire after the API returns with the backend error code; the intent events fire before
- [ ] Cancel request body carries `reasonCode`, `reasonText`, `source` and old backends still return success; edit-schedule carries `source`, `entry`
- [ ] Firebase never receives free text or a placeholder fee: a test asserts the allow-list and the 100-char limit
- [ ] Screens, navigation, copy and amounts shown are identical to before (screenshot comparison in the PR)
- [ ] `order_id` equals `bulkOrderId` everywhere; `service_id` is a plain param, not the `serviceId:` argument
- [ ] DebugView matrix: customer cancel success, cancel API failure, dismissed at reason and confirm, cancel-to-reschedule, reschedule success, slot unavailable, reschedule failure
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
