# App tracking booking checkout

- **ID:** T-0065
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-booking-checkout/` · **Branch:** `feat/tracking-booking-checkout`
- **Tier:** T3 · booking/checkout/payment/top-up events the backend maps; money-adjacent flow (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 D, E, J
- **Depends on:** T-0082 merged first (owns the Instant parts of `category_building_screen.dart`); T-0063; T-0074 and T-0077 merged first (they own the guest sign-in gate and the address region of `confirm_and_pay_checkout_screen.dart`)
- **Touches:** `lib/view/screen/home/category_building/**`, `lib/view/screen/home/top_up/**`, `lib/view/screen/home/web_view/payment_aba_webview_screen.dart`, `lib/view/custom/{custom_payment_*,custom_schedule_*,custom_service_addon_group,custom_instant_*,custom_unavailable_schedule_sheet,custom_apply_coupon_section,custom_coupon_card}.dart`, `lib/analytics/trackers/*`, `lib/controller/payment_controller.dart` (tracking lines only)

## Goal
Instrument spec §4 D (service and booking), E (checkout and payment) and J (top-up): element clicks, overlays (policy, schedule, payment result), `app_error` on price/coupon/payment failures, `topup_*` funnel events, `coupon_*`. Do not change money math or existing contract events.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] No existing contract event changes name or params (diff shows additions only)
- [ ] Checkout/payment errors emit `app_error` with `endpoint_group`, never URLs or amounts of other users
- [ ] Top-up funnel events fire; completion stays backend-owned
- [ ] Unreachable screens (`lucky_draw*`, `appointment_status_*` duplicates) confirmed and skipped or listed
- [ ] DebugView matrix recorded; `EventContract` tests updated
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
