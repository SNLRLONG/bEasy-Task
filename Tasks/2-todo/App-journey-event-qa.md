# App: verify booking events and identity for the journey review

- **ID:** T-0009
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-journey-event-qa/` (only if a fix is needed) · **Branch:** `fix/journey-event-qa`
- **Tier:** T2 · verification first; re-tier to T3 if event names or params change (tracking contract the backend depends on)
- **Claimed by:** none
- **Started / updated:** – / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (DQ-07, T01, T06, T07, T10) · `Docs/tracking/v2/architecture/bEasy-tracking-architecture-guide.md` · `Docs/tracking/v2/backend-response/Journey_V2_Portal_Review.md` §4 "Mobile identity contract"
- **Depends on:** a dev build and Firebase DebugView; backend answer on the stage mapping
- **Touches:** read `lib/analytics/{event_contract,analytics_helper}.dart` and the booking flow screens; edit only if a bug is proven

## Goal
Prove the app fires `booking_details_completed` and `slot_selected` independently (DQ-07: both show 53), and that the identity contract holds. Reviewer: do not claim independent drop-off unless it is real.

## Scope
1. Device test T07: finish details, leave before choosing a slot. Record the events in DebugView. Also T06 (leave before details).
2. Read the code: which user action fires each event (`event_contract.dart:29-30`). If one action fires both, write it down so the funnel can be relabelled as one stage.
3. Ask backend which events map to `details_completed` (the spec lists none; it maps date and time to `booking_started`, summary and promo to `slot_selected`). If the mapping, not the app, makes them equal, send it to `Journey_Tech_Review_Backend_Request.md`.
4. Identity: on login set Firebase `user_id` to the real user id; on logout clear it; never a `GUSTMODE` id; send `x-app-instance-id`; keep `journey_id` across guest login (T01, T10).
5. Write results into the sign-off QA matrix. Fix only proven bugs.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (if code changes)  - [ ] G4 run it (device)
- [ ] G5 code review (if code changes)  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog (if code changes)

## Done when
- [ ] T06, T07, T01, T10 results recorded with event screenshots (personal data masked)
- [ ] Root cause of 53/53 stated: app, backend mapping, or truly one stage
- [ ] Any fix has a test and the tier re-checked
- [ ] Worktree and branch removed after merge (if created)

## Added 2026-10-20: can the app create the order twice?
Backend found no second order creation in the payment callback, so the duplicate may come from repeated `customer/order/v6/create` calls. In `confirm_and_pay_checkout_screen.dart` (call at line 1696) and `payment_controller.dart:2018` check: is the button disabled while the request runs (double tap)? Is there a retry or re-send after a timeout or when the payment sheet returns? Is the same `Idempotency`-style key available to send? Report what you find; add an idempotency key only together with the backend.

## Diagnosis 2026-10-20 (read-only, app code): registration events
- `otp_verified` and `registration_completed` fire back to back in one block (`auth_controller.dart:731-735`, when `fromRegistration`). For registrations they are the same moment, so OTP verified = Completed (135 = 135 on prod) is by construction, not a data bug.
- The current auth flow uses `NewVerificationCodeScreen`, which has **no analytics calls** (0 `AnalyticsHelper`). `otp_viewed` is only in the old `VerificationCodeScreen`, which nothing opens; `otp_submitted` is only in the old `signin_screen.dart:328`; `trackOtpFailed` is never called anywhere. So the current app sends no `otp_viewed`, `otp_submitted` or `otp_failed`. Backend cannot classify OTP failures from events that are not sent.
- `signup_viewed` is sent from `new_signup_screen.dart:53`, so Sign-up started = Completed (135 = 135) is suspicious: anyone who opened the sign-up page and left should make it larger. Needs the raw event counts from backend (request file §2).
- Fix for the app (separate task, T3 because event contract): add `otp_viewed`, `otp_submitted` (attempt_no) and `otp_failed` (error_code) to `NewVerificationCodeScreen`; decide events for "phone checked: existing or new".

## Result 2026-10-20: can the app send create-order twice? (read-only)
- One call site for `customer/order/v6/create` (`confirm_and_pay_checkout_screen.dart:1696`). No automatic retry or re-send found in `lib/data` or `lib/helper`; the payment return does not call create again.
- Double tap: guarded. The method tile sets `loadingIndex` synchronously before the await and returns if it is already set (`:1679`). The local `isProgressing` flag inside `itemBuilder` is reset by rebuilds, so `loadingIndex` is the real guard. It looks adequate for a single session.
- Gap: no idempotency key. If the first request succeeds on the server but the response is lost (timeout, bad network), the user returns to the sheet and taps again: that is a legitimate second request and creates a second order. This fits "checkout creates one order, then another appears" but is not proven.
- Needs: backend real-data check (time apart and `tranId` of each pair). If the pairs are seconds to minutes apart with the same payload, add an idempotency key (backend and app together).
