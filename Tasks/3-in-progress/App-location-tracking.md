# App location tracking

- **ID:** T-0077
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-location-tracking/` · **Branch:** `feat/location-tracking`
- **Tier:** T3 · new tracking events the backend maps (service area, address); location data rules (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** 2026-11-02 / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md` §C
- **Depends on:** T-0063; backend contract in `pending-request-backend.md` §10 item 4 (can start before the reply)
- **Touches:** `lib/view/screen/profile/{location_screen,new_location_screen,new_search_location}.dart`, `lib/controller/location_controller.dart` (tracking lines only), `lib/view/screen/home/category_building/confirm_and_pay_checkout_screen.dart` (address region only: `_onAddressActionTap`, `_pickOrCreateAddress`, `_openAddAddress`, `_applyDefaultAddressFromList`), `lib/view/custom/{custom_address_listtile,custom_alert_holiday}.dart`, `test/`

## Goal
Track every action of the location flow (spec C.2): list actions, map pin, current location and its permission outcomes, place search, coverage check and out-of-range sheet, create/update/delete/default, API failures. Verify `address_selected` fires for every way an address reaches checkout. Confirm which of `location_screen` / `new_location_screen` paths are live. No street text, floor, room, note or exact coordinates in any param.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [ ] G6 security  - [x] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [x] Each handler in the three location screens is mapped to an event or listed as intentionally untracked (table below)
- [x] `service_area_checked` fires with `in_range`, `area_cell`, `result` for list select and save-from-checkout
- [x] Permission denied / services disabled / denied-forever each produce `permission_result`
- [x] `address_selected` verified for: default auto-apply, list select, add new, edit, return from profile list (labels: `default_auto`, `saved_list`, `existing`; `new_address` not distinguishable, see Progress)
- [x] A test asserts no location event carries a key outside the allowed set (`test/location_tracking_test.dart`)
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge


## Progress (2 Nov 2026)
Branch `feat/location-tracking`, cut from `feat/tracking-foundation` (d08204b1). Nothing committed. Analyze clean; 149 tests pass (only the old `widget_test.dart` fails).

L-7 answered: **both list screens are live**. Profile "My location" opens `NewLocationScreen` when there are no saved addresses, else `LocationScreen`; checkout opens `LocationScreen(returnSelectedLocation)` or `NewLocationScreen(fromCheckoutScreen)`.

Handler map (events go through `AnalyticsHelper`; no street text, floor, room, note or exact coordinates):

| Where | Handler | Event |
|---|---|---|
| `location_screen` | back | `element_clicked back` |
| | "add location" button | `element_clicked add_address` |
| | current-location tile (with / without permission) | `element_clicked use_current_location`; no permission: `permission_result` (services_disabled / denied_forever / denied / granted, context `use_current_location`) |
| | address card tap | `element_clicked address_card` (item_id) |
| | coverage check on select (checkout) | `service_area_checked` (in `checkAddressV2`); UI timeout: `app_error` (coverage / timeout); out-of-range sheet: `overlay_viewed out_of_range` |
| | edit / delete icons | `element_clicked edit_address` / `delete_address`; delete result: `address_deleted` |
| `new_location_screen` | back, save | `element_clicked back` / `save_address`; save result: `address_added` / `address_edited` (flags, method, source) |
| | search bar / icon, search sheet, use-current-location (3 places) | `element_clicked search_bar` / `use_current_location`; sheet: `overlay_viewed address_search` |
| | map tap / drag / current location / place picked | `location_pin_moved` (method tap / drag / current_location / search, area_cell) |
| | map opens (auto current location) | `permission_result` with context `auto_open` |
| `new_search_location` + sheet | typing (debounced in the controller) | `search_performed` (scope place, no query) |
| | pick a suggestion | `search_result_clicked` (rank) |
| controller | Places / address API failures | `app_error` (places / address) |
| checkout | `address_selected` | now carries `source=checkout` and `method` (default_auto / saved_list / existing) |

Intentionally untracked / not built: `address_default_set` (the set-default feature is commented out in the app); `form_submitted address` (covered by `save_address` + `address_added`); `custom_address_listtile` has no handler of its own.
Coverage check failures: `checkAddressV2` still returns false on error as before; the event reports `result=error` with no `in_range`, so failures are not counted as out of range.

Merge note: T-0074 also edits `confirm_and_pay_checkout_screen.dart` (`_onAddressActionTap`, `_runGuestSignIn`, one schedule line). This task only changes the four `_tracker.addressSelected(...)` lines, so the hunks do not overlap.

Still open: G4 device matrix, G6, G8-G11.

## Review round 1 (fresh agent, 2 Nov 2026)
Fixed: (1) `address_selected` never fired for an address added at checkout (the add flow pops `true`, not a map): the `true` branch now sends `method=new_address`. (2) Drag detection: the first idle after the map is created is ignored, and the programmatic flag is cleared 800 ms after our own animation so a no-op animation cannot swallow the next real drag. (3) `pinMethod` resets when the form opens and is `current_location` when the background locate moves the pin. (4) A failure after a successful save no longer reports `app_error save_failed`. (7) `permission_result granted` fires only when the OS prompt was shown in that call (denied results fire every time).
Accepted and documented: a slow coverage check that outlives the 20 s UI timeout still sends its own `service_area_checked` later (`duration_ms` over 20000); `address_id` is omitted when the id is not numeric.
Left: `_buildMap` in `new_location_screen.dart` is dead code (edits there are harmless); controller logic (`reportSave` guard, idle flag, `pinMethod`) is covered by review and the device matrix, not unit tests.
After fixes: analyze clean; 149 tests pass (only the old `widget_test.dart` fails).

## Status (2 Nov 2026)
Committed as dfb024cd and pushed on `feat/location-tracking` (on top of d08204b1). **Not merged: owner holds all merges to `main` until the whole app side is done.** Open: G4 (combined device matrix), G6, G8-G11.

## Backend note (to send)
The 2 Nov handoff is out of date on the location item (docs in the repo are already aligned; this is the message). Corrections:
1. `service_area_checked`: a failed check has `result=error` and no `in_range`; do not count it as out of range. A UI timeout (20 s) is an `app_error` (`endpoint_group=coverage`, `error_code=timeout`); a slow check can still send its own late `service_area_checked` with `duration_ms` over 20000. `address_id` is omitted when not numeric. Table columns `address_id` and `in_range` are nullable.
2. `address_default_set` is not sent (the set-default feature is disabled in the app). `address_added|edited` carry `source`, `result`, `method`, `has_floor|room|note`, `area_cell`, `error_code`, no `address_id` or `in_range`.
3. `address_selected` gains `source=checkout` and `method` (`default_auto|saved_list|new_address|existing`).
4. `permission_result` (location): `granted` only after the OS prompt was shown; `denied|denied_forever|services_disabled` on every attempt; `context` is `use_current_location|auto_open`.
5. `search_performed` for places has no `query`; `location_pin_moved` method `drag|tap|current_location|search`.
Status: no longer needed. The owner withdrew the earlier handoff (3 Nov); the final handoff will include these facts.
