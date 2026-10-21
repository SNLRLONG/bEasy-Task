# Registration funnel: rework to match the real app flow

- **ID:** T-0006
- **Repo:** cross (bEasy-portal, backend, bEasy-app)
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T3 · changes stage definitions and the tracking contract the backend depends on
- **Claimed by:** none
- **Started / updated:** – / 2026-10-20
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (UX-10) · `Docs/tracking/v2/architecture/bEasy-tracking-architecture-guide.md` (registration events) · `Journey_Tech_Review_Backend_Request.md` item B2
- **Depends on:** `Portal-journey-registration-fixes` merged; product decision on the stages below
- **Touches:** later: backend registration stage mapping, `app/lib/journey/stages.ts`, registration funnel components, locales

## Problem (owner, 2026-10-20)
The Registration funnel stages do not match the real app flow. Today: Registration page viewed, OTP requested, OTP verified, Sign-up started, Completed.

Real app flow:
1. View Registration page
2. Enter phone number; the API checks existing user or new user
3. New user: sign-up page, fill info
4. OTP request
5. OTP verify
6. Register success
7. Select registration source (can probably be excluded)

So sign-up comes **before** OTP, and existing users take a different branch (login, not registration) that the funnel should not count as drop-offs.

## What the code shows (to use when reworking)
- App events today (`bEasy-app/lib/analytics/event_contract.dart`): `registration_started`, `otp_viewed`, `otp_submitted` (with `attempt_no`), `otp_failed` (with `error_code`), `otp_resend_clicked`, `otp_verified`, `signup_viewed`, `registration_completed`. There is no event for the existing-or-new phone check, and none for the registration source.
- Backend maps `registration_viewed` = phone entered and `signup_started` = sign-up form reported (spec §Empty for now); existing customers' logins are excluded.
- The app already sends `otp_failed` with an error code and `otp_submitted` with the attempt number. The handoff says wrong-code is not instrumented, so check whether backend can use these before asking for new OTP events.

## To decide when this is picked up
- New stage list and order, and whether the "phone checked" step needs a new event.
- Whether the registration source step is excluded (owner leans yes).
- How existing users are shown (engagement view, not registration drop-off), consistent with the acquisition `customerKind` default.
- Backward compatibility of past data with the new order.

## Done when
- [ ] Stage list approved by product and matches the app screens
- [ ] Backend mapping, app events (if any) and portal labels agree
- [ ] Test T03, T04, T05 pass on the new flow

## Diagnosis 2026-10-20
Prod screenshot: OTP verified, Sign-up started and Registration completed are all 135. See `App-journey-event-qa.md` (Diagnosis): `otp_verified` and `registration_completed` are fired together, `NewVerificationCodeScreen` sends no OTP events, and `signup_viewed` equal to completions needs backend raw counts. Rework needs the app to send OTP events first.

## Can it be done without an app change? (2026-10-20, from app code)
Mostly yes. The app already sends `X-Reg-Flow-Id` and `X-App-Instance-Id` on the phone status check (`customer/auth/account/{phone}/status/check`), send OTP, OTP/password login and account create (`lib/data/api/tracking_headers.dart`, `wantsRegFlowId`), and already sends `registration_started` and `signup_viewed`. So backend can build the stages from its own API records: phone checked (and the new/existing answer), OTP requested, OTP verified (login success), account created, and OTP failures (wrong code, expired, locked, rate limit, SMS delivery) from the verify and gateway records. Not possible without the app: screens with no API call (someone who opened the page and left before submitting), and the optional registration-source step. Needs from backend: confirm it reads `X-Reg-Flow-Id` on these four calls and stores one row per call with the result.
