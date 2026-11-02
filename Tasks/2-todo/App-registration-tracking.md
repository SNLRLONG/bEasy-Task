# App registration tracking

- **ID:** T-0074
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-registration-tracking/` · **Branch:** `feat/registration-tracking`
- **Tier:** T3 · tracking event names/params the backend depends on; identity/OTP flow (tracking lines only, no flow change) (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md` §A.4, A.5, B
- **Depends on:** T-0063
- **Touches:** `lib/view/screen/account/new_auth/*`, `lib/controller/auth_controller.dart` (tracking lines only), `lib/analytics/registration_store.dart`, the top-right CS button on sign in, OTP and sign up (`element_clicked` `contact_support`), one-line `entry:` edits at: `home_screen.dart:931`, `app_screen.dart:418`, `confirm_and_pay_checkout_screen.dart` (`_runGuestSignIn` and its 3 callers), `banner_detail_screen.dart:918`, `firebase_messaging_service.dart:136`, `helper.dart:321`, `custom_show_dialog.dart:124`, `test/`

## Goal
Make the app report the registration the way it really runs and where it started. Pass `entry` / `entry_detail` / `source_screen` from every caller into `NewSignINScreen` (closed set in spec B.2); keep it in `RegistrationStore` for later events. Add `otp_mode`, `flow_type`, `attempt_round` to OTP events; add `otp_code_entered`, `signup_submitted`, `registration_resumed`; fire `otp_submitted` and `otp_failed` for the deferred (held-code) path; add a Firebase `account_created` event at account creation; **leave `sign_up`, Meta `CompleteRegistration` and AppsFlyer timing unchanged** (owner decision 2 Nov: no app-flow change). Dead legacy auth screens are not touched.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Every live entry route reports a non-`other` `entry`; none reports `gate` (checked per call site in the DebugView matrix)
- [ ] New-number flow on a device produces, in order: `registration_started`, `otp_viewed`, `otp_code_entered`, `signup_viewed`, `signup_submitted`, `otp_submitted`, `otp_verified`, `registration_completed`, all with the same `reg_flow_id`
- [ ] Held-code rejection path (wrong/expired code at submit) produces `otp_failed` and a second `otp_viewed` with `attempt_round=2`
- [ ] Existing-number login produces `flow_type=login` and no `registration_completed`
- [ ] `EventContract` and tests updated; no PII in any param
- [ ] Note for marketing: Meta `CompleteRegistration` counts account creation (before OTP verification); the portal counts `registration_completed`. Numbers are not comparable
- [ ] App screens, navigation and request order are identical to before (no flow change)
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
