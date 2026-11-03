# App tracking foundation

- **ID:** T-0063
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-foundation/` · **Branch:** `feat/tracking-foundation`
- **Tier:** T3 · new tracking event names/params the backend depends on; shared analytics wrapper (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §2, §3, §8
- **Depends on:** T-0062 approved
- **Touches:** `lib/analytics/{analytics_helper,event_contract,screen_observer}.dart`, new `lib/analytics/tracking_registry.dart`, `lib/my_app.dart` (lifecycle pause/resume, 2 lines), `lib/view/app/app_screen.dart` (tab hook, 2 lines; T-0064 and T-0074 also edit this file at other lines), `test/tracking_registry_test.dart`. `next_screen.dart` needed no change (route names already `/<WidgetClass>`).

## Goal
Add the shared pieces every slice needs: registry of `screen_name`/`element_id`, helper methods (`trackElementClicked`, `trackOverlayViewed`, `trackFormSubmitted`, `trackSearch*`, `trackSettingChanged`, `trackPermissionResult`, `trackSupportContacted`, `trackAppError`), extended `screen_viewed` with `screen_group`/`entry_type`, `screen_exited` with duration, tab and overlay observation, user properties, contract rules and tests. No screen edits except what is needed to prove it.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [x] Registry and helper methods exist; call sites in other tasks need no edits to `analytics_helper.dart`
- [x] `screen_viewed` fires for tabs, sheets, dialogs and web views, debounced; `screen_exited` only when ≥1 s
- [x] `EventContract` covers every new event; unit tests per builder and a registry-name test pass
- [x] Legacy events untouched (dual-fire handled by slices)
- [ ] User properties `app_language`, `push_enabled`, `location_permission`, `has_ordered` set and cleared on logout
- [x] `fvm flutter analyze` and `fvm flutter test` pass
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress (2026-11-02)
- Built in worktree `bEasy/bEasy-app-tracking-foundation/`, branch `feat/tracking-foundation`, not committed (AGENTS.md).
- New: `lib/analytics/tracking_registry.dart` (screens, groups, route class to id map, element/entry/reason sets). Changed: `analytics_helper.dart` (`trackScreenViewed` extended with `screen_group`/`entry_type`; new `trackScreenExited`, `trackOverlayViewed`, `trackElementClicked`, `trackFormSubmitted`, `trackSearchPerformed`, `trackSearchResultClicked`, `trackSettingChanged`, `trackPermissionResult`, `trackSupportContacted`, `trackAppError`, `setContextProperty`, has_ordered reset on sign-out), `screen_observer.dart` (`ScreenTracker`, `overlaySettings()`, named popups become `overlay_viewed`, `AppScreen` route resolves to the active tab), `event_contract.dart` (rules for the new events, id format check).
- Tests: `test/tracking_registry_test.dart` (new, 15 cases). Full suite 134 pass; `widget_test.dart` (Flutter counter template) fails on `main` as well, unrelated.
- `fvm flutter analyze`: no issues.
- Not done / needs a person: **G4 on-device check** (Firebase DebugView: tab switch, back navigation, background/foreground, a named sheet). Not run, no device here. Sheets/dialogs only report when opened with `overlaySettings('<name>')`, so slices add that per sheet. `setContextProperty` has no callers yet (slices call it: language, push, location, first order).
- Gates not yet run: G5 code review, G6 security, G9, G10, G11.
- Release note: `screen_viewed.screen_name` values change from class names to registry ids (`HomeScreen` -> `home`). Backend skips `screen_viewed` today, so no live consumer; tell backend when §9 is agreed.

## Review round 1 (fresh review agent, 2026-11-02): G5 done, all actionable findings fixed
- Fixed: stale tab on a fresh `AppScreen` (observer resets to `home` on push/replace); `resume()` only restarts a visit after a pause; `enterTabIndex` never throws on the navigation path; unnamed page routes report `unnamed_page` instead of leaving a stale screen; overlay type travels in `overlaySettings(name, overlayType:)`; closed sets for `trigger`, `result`, `permission`, `channel`; search `query` dropped when it has `@` or 6+ digits; circular import removed (`current_screen.dart`); `setting_changed.value` renamed `setting_value` (value is numeric elsewhere); spec updated.
- Tests: 18 cases now assert emitted events through `AnalyticsHelper.debugSink` (1 s minimum, 30 min cap, order exit-before-view, pause/resume, AppScreen tab cases, overlay, privacy, closed sets). Full suite 143 pass; only `widget_test.dart` fails (same on `main`). Analyze: no issues.
- Left as is (low): tab switch while another route covers AppScreen (no call site exists); `has_ordered` carries over on a direct A-to-B account switch with no sign-out.
- Still open: G4 on-device check, G6 security, G9 human review, G10 release plan, G11 changelog.

## Status (2 Nov 2026)
Committed and pushed as d08204b1 on `feat/tracking-foundation`. **Not merged: the owner holds all merges to `main` until the whole app side of tracking is done.** T-0074 and T-0077 branch from it. G4 (device matrix) is run once on a combined build from an integration branch, not per task. Open: G4, G9, G10, G11.
