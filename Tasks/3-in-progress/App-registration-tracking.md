# App registration tracking

- **ID:** T-0074
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-registration-tracking/` · **Branch:** `feat/registration-tracking`
- **Tier:** T3 · tracking event names/params the backend depends on; identity/OTP flow (tracking lines only, no flow change) (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** 2026-11-02 / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md` §A.4, A.5, B
- **Depends on:** T-0063
- **Touches:** `lib/view/screen/account/new_auth/*`, `lib/controller/auth_controller.dart` (tracking lines only), `lib/analytics/registration_store.dart`, the top-right CS button on sign in, OTP and sign up (`element_clicked` `contact_support`), one-line `entry:` edits at: `home_screen.dart:931`, `app_screen.dart:418`, `confirm_and_pay_checkout_screen.dart` (`_runGuestSignIn` and its 3 callers), `banner_detail_screen.dart:918`, `firebase_messaging_service.dart:136`, `helper.dart:321`, `custom_show_dialog.dart:124`, `test/`

## Goal
Make the app report the registration the way it really runs and where it started. Pass `entry` / `entry_detail` / `source_screen` from every caller into `NewSignINScreen` (closed set in spec B.2); keep it in `RegistrationStore` for later events. Add `otp_mode`, `flow_type`, `attempt_round` to OTP events; add `otp_code_entered`, `signup_submitted`, `registration_resumed`; fire `otp_submitted` and `otp_failed` for the deferred (held-code) path; add a Firebase `account_created` event at account creation; **leave `sign_up`, Meta `CompleteRegistration` and AppsFlyer timing unchanged** (owner decision 2 Nov: no app-flow change). Dead legacy auth screens are not touched.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [ ] G6 security  - [x] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Every live entry route reports a non-`other` `entry`; none reports `gate` (checked per call site in the DebugView matrix)
- [ ] New-number flow on a device produces, in order: `registration_started`, `otp_viewed`, `otp_code_entered`, `signup_viewed`, `signup_submitted`, `otp_submitted`, `otp_verified`, `registration_completed`, all with the same `reg_flow_id`
- [ ] Held-code rejection path (wrong/expired code at submit) produces `otp_failed` and a second `otp_viewed` with `attempt_round=2`
- [ ] Existing-number login produces `flow_type=login` and no `registration_completed`
- [x] `EventContract` and tests updated; no PII in any param
- [ ] Note for marketing: Meta `CompleteRegistration` counts account creation (before OTP verification); the portal counts `registration_completed`. Numbers are not comparable
- [x] App screens, navigation and request order are identical to before (no flow change) (reviewer checked: only `unawaited` tracking lines on the auth path)
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge


## Progress (2 Nov 2026)
Branch `feat/registration-tracking`, cut from `feat/tracking-foundation` (d08204b1). Nothing committed.

Done (collection only, no flow change):
- `RegistrationStore`: entry, entry_detail, source_screen, OTP and sign-up round counters (memory only; an app restart reports `other`).
- `AnalyticsHelper`: `registration_started` validates `entry` against the closed set (unknown -> `other`); `otp_viewed` (+`flow_type`, `otp_mode`, `attempt_round`), `otp_code_entered`, `otp_submitted`/`otp_failed`/`otp_verified` (+`otp_mode`), `signup_viewed` (+`attempt_round`), `signup_submitted` (flags only), `account_created`, `registration_completed` (+`entry`), user property `registration_entry`.
- `auth_controller.dart` (tracking lines only): `signup_submitted` and `account_created` right after the create call returns (so the order is signup_submitted, otp_submitted, otp_verified, registration_completed); held code now fires `otp_submitted` and, when rejected, `otp_failed` with `otp_mode=deferred`; `loginWithOTP` got a tracking-only `otpMode` argument. `sign_up`, Meta `CompleteRegistration` and AppsFlyer untouched.
- `entry:` passed at 7 call sites: home, tab_gate (tab name), checkout_address, checkout_schedule, banner (id), push (type), deeplink, dialog. `NewSignINScreen` without an entry now reports `other` (debug log).
- Support button on sign in, OTP and sign up: `element_clicked` `contact_support`.
- Tests: `test/registration_tracking_test.dart` (8), `event_contract_test.dart` updated. Analyze clean; full suite passes except the pre-existing `widget_test.dart`.

Differences from the spec to settle:
- `checkout_summary` has no separate call site: the checkout "sign_up" text and the address row use the same handler (`_onAddressActionTap`), so both report `checkout_address`.
- `registration_resumed` is not built yet (needs a hook in `_completeGuestAuth`); do it in a follow-up commit of this task.
- `signup_submitted` is not fired on the "retry SMS only" branch (no create call) or for form-invalid taps.
- `error_code` values stay as the app has them today (`network`, `invalid_or_expired`, `http_<n>`, ...); the spec's `wrong_code`/`blocked`/`expired` need the backend reply first.

Still open: G4 (device DebugView matrix), G5, G6, G7 (spec edits above), G8-G11.

## Review round 1 (fresh agent, 2 Nov 2026)
Fixed: (1, high) first OTP screen of a new number reported `flow_type=login` (deferred screen opens with `isNewUser` 0): now `isNewUser || openSignupAfterCode`. (2) `attempt_round` over-counted after Back: round 2 now only after a rejected held code, a new deferred screen restarts the code and form counters. (3) `signup_submitted` could fire twice for one create (success then catch): guarded. (5) `tab_gate` detail now `bookings`/`profile` as in the spec; `entry_detail` cut to 40; `source_screen` is the screen before sign-in (the observer has already moved to `sign_in`; `CurrentScreen.previous` added).
Left (low): Telegram `account_created` is unawaited and could in theory get a new `reg_flow_id` if `clear()` wins the race (entry itself is read synchronously, so safe); the controller wiring (held-code failure path, event order) is covered by review and the device matrix, not unit tests.
After fixes: analyze clean; 153 tests pass (only the pre-existing `widget_test.dart` fails).

## Backend note
Correction note for the 2 Nov handoff sent to backend by the user (checkout_summary unused, signup_submitted timing, error_code values, account_created, registration_resumed pending). Waiting for their reply on telling wrong from expired codes.

## Status (2 Nov 2026)
Committed as 69076a64 on `feat/registration-tracking` (on top of d08204b1). **Not merged: owner holds all merges to `main` until the whole app side is done.** Open: G4 (combined device matrix), G6, G8-G11; `registration_resumed` still to build.

Update (3 Nov): the owner withdrew the handoff sent to backend; the correction note above is superseded by the final handoff sent after the last app slice.
