# App instant cleaning tracking

- **ID:** T-0082
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-instant-cleaning-tracking/` · **Branch:** `feat/instant-cleaning-tracking`
- **Tier:** T3 · new tracking params/events the backend depends on; common param on money-adjacent events (tracking lines only, no flow change) (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Instant_Cleaning_Tracking_Spec.md`
- **Depends on:** T-0063; T-0083 agreed names
- **Touches:** `lib/view/screen/home/category_building/{category_building_screen,product_option_selector,instant_room_suggestion_logic,checkout_booking_schedule_sheet,sub_category_screen}.dart` (Instant parts and tracking lines), `lib/view/custom/{custom_instant_rooms_section,custom_instant_time_selector,custom_explore_other_options_section,custom_excludes_section,custom_how_instant_cleaning_works_section,custom_why_choose_instant_cleaning_section,custom_estimated_cleaning_time_section,custom_category_selection_alert_sheet}.dart`, `lib/view/screen/activity/order_detail/{widgets/order_detail_instant_progress_card,widgets/order_detail_extend_sheet}.dart`, `lib/util/booking_grace_logic.dart` + `payment_controller.dart` (`requestCheckoutGrace` tracking only), `lib/analytics/trackers/service_detail_tracker.dart`, `test/`

## Goal
Make every click, tap and choice in Instant Cleaning tracked with no change to what the app shows or does: `booking_mode` on all service/order events, the §3 events, `selection_method` so auto-selection is not logged as a choice.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Every handler in the listed Instant files is mapped to an event or listed as intentionally untracked
- [ ] `booking_mode` present on journey, checkout, payment, add-on and order events for Instant; absent/`scheduled` for others
- [ ] Auto default, auto from rooms and restore-after-Deep-Cleaning are flagged via `selection_method`; no extra duplicate events
- [ ] Fully-booked sheet, availability result, grace errors (each code) and upsell round trip each fire once
- [ ] No free text or labels in params; `room_key` is a catalogue id; tests assert the allow-list
- [ ] App screens, navigation and requests identical to before (screenshot comparison)
- [ ] DebugView matrix: normal, fully booked, same-day too short, slot unavailable, rooms to 3h bigger-job, Deep Cleaning round trip, progress card stages
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
