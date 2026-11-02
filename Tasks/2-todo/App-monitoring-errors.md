# App monitoring errors

- **ID:** T-0068
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-monitoring-errors/` · **Branch:** `feat/monitoring-errors`
- **Tier:** T3 · new dependency and native config (Crashlytics); error telemetry contract (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §5.3
- **Depends on:** T-0062; answer to O-1
- **Touches:** `pubspec.yaml` (dependency only, not version), `android/`, `ios/` Crashlytics config, `lib/main.dart`, `lib/data/api/api_client.dart` (error hook), `lib/view/screen/splash/{no_internet_screen,handle_server_error_screen}.dart`, `lib/view/custom/custom_button_sheet_update_app.dart`

## Goal
Add Firebase Crashlytics (fatal + non-fatal), wire `FlutterError.onError` and `PlatformDispatcher.onError`, emit `app_error` for API, network, update and exception cases, tag Clarity with `screen_name`. Releases: Shorebird/OTA impact written in the task before merge.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Test crash and non-fatal visible in Crashlytics for dev flavor
- [ ] `app_error` fires for API failure, no-internet, server-error screen, force-update, with no URLs or PII
- [ ] Staff and non-release builds do not send
- [ ] Release note covers native change (needs store build, not an OTA patch)
- [ ] Rollback: remove dependency in a patch release; documented
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
