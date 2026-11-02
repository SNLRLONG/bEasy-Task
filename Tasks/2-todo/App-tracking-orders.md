# App tracking orders

- **ID:** T-0066
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-orders/` · **Branch:** `feat/tracking-orders`
- **Tier:** T3 · cancellation/reschedule/extend events the backend depends on (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 F
- **Depends on:** T-0063; T-0079 merged first (same folder `order_detail/*`)
- **Touches:** `lib/view/screen/activity/**`, `lib/view/custom/{custom_slidable_item,custom_show_dialog}.dart` (order-related callers only), `lib/controller/payment_controller.dart` (tracking lines outside cancel/reschedule)

## Goal
Instrument spec §4 F (orders): bookings list, order detail v2 widgets, extend sheet, review form. **Cancel and reschedule are owned by T-0079** (`Cancellation_And_Reschedule_Spec.md`) and are not touched here.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Order detail top-right CS button (`order_detail_app_bar.dart`) fires `element_clicked` `contact_support` and `support_contacted` channel `chat` with the order id
- [ ] Extend sheet fires `overlay_viewed` and its element clicks; cancel and reschedule sheets are left to T-0079
- [ ] Review submit sends `form_submitted(review)` with counts only, never comment text
- [ ] v1 vs v2 detail reachability is decided in T-0079; this task follows its result
- [ ] DebugView matrix recorded
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
