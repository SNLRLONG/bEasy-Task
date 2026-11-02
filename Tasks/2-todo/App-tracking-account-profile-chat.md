# App tracking account profile chat

- **ID:** T-0067
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-account-profile-chat/` · **Branch:** `feat/tracking-account-profile-chat`
- **Tier:** T2 · new events using the generic contract from T-0063 (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 B, H, I
- **Depends on:** T-0063, T-0074, T-0077 merged first (rebase)
- **Touches:** `lib/view/screen/{profile,message}/**` (not `profile/*location*.dart`, owned by T-0077; not `account/new_auth/*`, owned by T-0074), `lib/chat_feature/{views,components}/**`, `lib/controller/auth_controller.dart` (tracking lines only), `lib/controller/localization_controller.dart` (`selectLanguage`, `syncServerLanguage`, tracking lines only), `lib/view/screen/account/customer_source_screen.dart`, `lib/view/custom/{custom_code_picker_widget,custom_country_code_sheet,custom_contact_support_sheet,custom_argree_tc,custom_text*}.dart`

## Goal
Instrument spec §4 H (profile, language, help, tutorials; address and location moved to T-0077) and I (chat); auth and registration moved to T-0074: menu rows, forms, address management events, language `setting_changed`, permission results, chat events without any content.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] All 13 profile menu rows map to `element_id`s
- [ ] `address_*` events never carry street text
- [ ] Chat events carry message type only
- [ ] Language switch from every picker (Home sheet, Profile) and the silent server sync fires `setting_changed` with `source` and sets user property `app_language`, also at app start
- [ ] `CustomContactSupportSheet`: sheet view, Telegram, hotline, email each fire `support_contacted` with `source_screen`; Customer source CS button tracked
- [ ] DebugView matrix recorded
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
