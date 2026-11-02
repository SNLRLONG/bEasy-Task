# App location tracking

- **ID:** T-0077
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-location-tracking/` · **Branch:** `feat/location-tracking`
- **Tier:** T3 · new tracking events the backend maps (service area, address); location data rules (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md` §C
- **Depends on:** T-0063; backend contract in `pending-request-backend.md` §10 item 4 (can start before the reply)
- **Touches:** `lib/view/screen/profile/{location_screen,new_location_screen,new_search_location}.dart`, `lib/controller/location_controller.dart` (tracking lines only), `lib/view/screen/home/category_building/confirm_and_pay_checkout_screen.dart` (address region only: `_onAddressActionTap`, `_pickOrCreateAddress`, `_openAddAddress`, `_applyDefaultAddressFromList`), `lib/view/custom/{custom_address_listtile,custom_alert_holiday}.dart`, `test/`

## Goal
Track every action of the location flow (spec C.2): list actions, map pin, current location and its permission outcomes, place search, coverage check and out-of-range sheet, create/update/delete/default, API failures. Verify `address_selected` fires for every way an address reaches checkout. Confirm which of `location_screen` / `new_location_screen` paths are live. No street text, floor, room, note or exact coordinates in any param.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Each handler in the three location screens is mapped to an event or listed as intentionally untracked
- [ ] `service_area_checked` fires with `in_range`, `area_cell`, `result` for list select and save-from-checkout
- [ ] Permission denied / services disabled / denied-forever each produce `permission_result`
- [ ] `address_selected` verified for: default auto-apply, list select, add new, edit, return from profile list
- [ ] A test asserts no location event carries a key outside the allowed set
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
