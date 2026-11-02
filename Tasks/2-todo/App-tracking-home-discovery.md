# App tracking home discovery

- **ID:** T-0064
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-home-discovery/` · **Branch:** `feat/tracking-home-discovery`
- **Tier:** T2 · new events using the generic contract from T-0063; no new names (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 A, C, G
- **Depends on:** T-0063; T-0074 merged first (it adds one `entry:` line in `home_screen.dart`, `app_screen.dart`, `banner_detail_screen.dart`, `helper.dart`)
- **Touches:** `lib/view/screen/{splash,home,notification}/**`, `lib/view/custom/{custom_banner_list,custom_telegram_banner,custom_announcement_*,custom_notification_*,custom_promo_*,custom_button_sheet_update_app}.dart`, `lib/util/local_notifications_service.dart`, `lib/view/app/app_screen.dart`

## Goal
Home language icon fires `element_clicked` `language_picker` (the choice itself is tracked in T-0067's controller hook). Instrument spec §4 A (launch), C (home and discovery) and G (notifications): element clicks, home search events, banner clicks, notification inbox/detail actions, tab navigation, dual-fire legacy events with `screen_name`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Every live handler in the listed files is mapped to an `element_id` or listed as intentionally untracked with a reason
- [ ] Search fires `search_performed`/`search_result_clicked` (debounced, ≤30 chars)
- [ ] Legacy events still fire
- [ ] DebugView matrix (screen, action, event, params) recorded in this file
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
