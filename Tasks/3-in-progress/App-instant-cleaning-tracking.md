# App instant cleaning tracking

- **ID:** T-0082
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-instant-cleaning-tracking/` · **Branch:** `feat/instant-cleaning-tracking`
- **Tier:** T3 · new tracking params/events the backend depends on; common param on money-adjacent events (tracking lines only, no flow change) (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** 2026-11-02 / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Instant_Cleaning_Tracking_Spec.md`
- **Depends on:** T-0063; T-0083 agreed names
- **Touches:** `lib/view/screen/home/category_building/{category_building_screen,product_option_selector,instant_room_suggestion_logic,checkout_booking_schedule_sheet,sub_category_screen}.dart` (Instant parts and tracking lines), `lib/view/custom/{custom_instant_rooms_section,custom_instant_time_selector,custom_explore_other_options_section,custom_excludes_section,custom_how_instant_cleaning_works_section,custom_why_choose_instant_cleaning_section,custom_estimated_cleaning_time_section,custom_category_selection_alert_sheet}.dart`, `lib/view/screen/activity/order_detail/{widgets/order_detail_instant_progress_card,widgets/order_detail_extend_sheet}.dart`, `lib/util/booking_grace_logic.dart` + `payment_controller.dart` (`requestCheckoutGrace` tracking only), `lib/analytics/trackers/service_detail_tracker.dart`, `test/`

## Goal
Make every click, tap and choice in Instant Cleaning tracked with no change to what the app shows or does: `booking_mode` on all service/order events, the §3 events, `selection_method` so auto-selection is not logged as a choice.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [ ] G6 security  - [x] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [x] Every handler in the listed Instant files is mapped to an event or listed as intentionally untracked (table below)
- [ ] `booking_mode` present on journey, checkout, payment, add-on and order events for Instant; absent/`scheduled` for others
- [ ] Auto default, auto from rooms and restore-after-Deep-Cleaning are flagged via `selection_method`; no extra duplicate events
- [ ] Fully-booked sheet, availability result, grace errors (each code) and upsell round trip each fire once
- [x] No free text or labels in params; `room_key` is a catalogue id; tests assert the allow-list (`test/instant_tracking_test.dart`)
- [ ] App screens, navigation and requests identical to before (screenshot comparison)
- [ ] DebugView matrix: normal, fully booked, same-day too short, slot unavailable, rooms to 3h bigger-job, Deep Cleaning round trip, progress card stages
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge


## Progress (2 Nov 2026)
Branch `feat/instant-cleaning-tracking`, cut from `feat/tracking-foundation` (d08204b1). Nothing committed. Analyze clean; 150 tests pass (only the old `widget_test.dart` fails). Collection only: no screen, navigation or request changed.

| Where | Handler | Event |
|---|---|---|
| `AnalyticsHelper` / `ServiceDetailTracker` | any event with a `service_id` | `booking_mode` (`instant` / `scheduled`), remembered per service when the detail screen loads (`setBookingMode`); events fired before the mode is known, and order-only events (no `service_id`), have none |
| `category_building_screen` | availability check on load and quiet refresh | `instant_availability_checked` (trigger `load` / `refresh`; skipped when there are no dates) |
| | fully-booked sheet | `instant_fully_booked_viewed` |
| | hour card tap / auto default / restore after Deep Cleaning | `service_option_selected`, `duration_selected` (+`selection_method`) and `instant_duration_selected` (`user` / `auto_default` / `restored`, `was_suggested`) |
| | room tick | `instant_room_toggled` (`room_key` slug of `sid:<id>`, count, minutes, band, suggested hours) |
| | under-pick warning appears | `instant_underpick_warning_viewed` (once per appearance) |
| | "Need more than 2 hours?" / "See Deep Cleaning" | `instant_upsell_clicked` (source `text_link` / `rooms_section`), `instant_upsell_result` (`returned=1`), legacy event keeps firing with `source` |
| `custom_category_selection_alert_sheet` | sheet shown / closed / CTA | `category_alert_viewed`, `category_alert_action`, `overlay_viewed category_alert` |
| `custom_instant_time_selector` | time chosen | `instant_slot_selected` (slot_source `selector`) |
| `payment_controller.requestCheckoutGrace` | grace ok / error | `schedule_grace_result` (error codes closed set, `lead_minutes`, `same_day`, `action`) |
| `order_detail_instant_progress_card` | stage shown | `instant_progress_viewed` (once per stage per card) |
| `order_detail_extend_sheet` | open / pay method / close | `extend_started`, `extend_payment_selected`, `extend_closed`, `overlay_viewed extend_hours` (no price: the quote is a placeholder) |

Intentionally untracked / differences from the spec:
- `custom_excludes_section`, `custom_how_instant_cleaning_works_section`, `custom_why_choose_instant_cleaning_section`, `custom_estimated_cleaning_time_section`: static content with no handler, so there is nothing to tap; `element_clicked` for them is not sent.
- `instant_upsell_result.converted` is not sent (needs state shared with the Deep Cleaning screen); `returned=1` only. The funnel can be read from `checkout_started` of the target service.
- `instant_slot_selected` is sent from the time selector only. The checkout schedule sheet serves every service and already feeds `date_selected` / `time_selected` / `slot_selected`, which now carry `booking_mode`.
- `instant_progress_viewed` has no `order_id`: the card does not know it and the order detail screen is owned by T-0079.
- `instant_hours` on `checkout_started` / `payment_started` is not added yet (those trackers belong to the checkout slice, T-0065).
- `auto_rooms` exists in the closed set but is never sent: room ticks do not change the hour selection in the app.
- `extend_hour_pay` (existing, UI-only) is unchanged.

Still open: G4 (combined device matrix: normal, fully booked, same-day too short, slot unavailable, rooms to 3h bigger-job, Deep Cleaning round trip, progress card stages), G6, G8-G11.

## Review round 1 (fresh agent, 2 Nov 2026)
Fixed: (1, high) `instant_slot_selected` never fired: the time picker returns a `TimeOfDay`, the tracker parsed its `toString()`; it now reads `hour` / `minute`. (2) `booking_mode` was registered after the first availability and fully-booked events and skipped when the service name was empty: it is now registered right after the details load and before the name guard. (3) The available-date count could throw on an odd schedule item: counted in a safe helper.
Accepted and documented: `category_alert_*` fire from the home grid before the service screen exists, so they carry no `booking_mode`; `overlay_viewed` and the named event both fire for the alert and extend sheets (dashboards must count one of them); the return from Deep Cleaning is labelled `trigger=refresh` (the `return` value is allowed but not sent); `_underpickReported` can miss a re-appearance after the stash restore (under-count); `same_day` uses the device time zone; `schedule_grace_result` has no `service_id`.
Left: tests cover the helper layer, not `ServiceDetailTracker` / `setBookingMode` through `track()`, nor the widgets; the device matrix covers those.
After fixes: analyze clean; 150 tests pass (only the old `widget_test.dart` fails).

## Status (3 Nov 2026)
Committed as c0786147 on `feat/instant-cleaning-tracking`. **Not merged to `main`: the owner holds all merges until the whole app side is done; branches are merged into `feat/tracking-integration` for the combined device check (G4).** Open: G4, G6, G8-G11.
