# Customer Journey: backend response on OTP reasons

**Date:** 30 Oct 2026 (Asia/Phnom_Penh) · **For:** Portal task T-0020 · **Answers:** the old item 1 of `Journey_Tech_Review_Backend_Request.md` (removed now it is answered)

Source-and-test review by backend, not a captured response from the deployed dev API or a production audit. No application code was changed. It is newer than the 19 Oct handoff: OTP send and verification now write diagnostic events to `tracking_otp_events`.

## 1. Which OTP reasons are recorded today

| Availability key | Supported now? | Meaning |
| --- | --- | --- |
| `delivery` | No (`false`) | No SMS delivery-receipt ingestion produces `delivery_failed`. Gateway acceptance does not confirm delivery to the handset. |
| `gateway` | Yes, if the diagnostic read works | `sendOtp.js` records `gateway_error` when the PlasGate wrapper reports an error (including timeouts). Also groups `OTP_LIMITER_UNAVAILABLE`, so it is not only provider failures. |
| `wrongCode` | Yes, if the diagnostic read works | `login.js` records `wrong_code` for `OTP_MISMATCH`, and also for `OTP_NOT_FOUND`, so it is broader than a confirmed mismatch. |
| `rateLimit` | Yes, if the diagnostic read works | `sendOtp.js` records `rate_limited` when the limiter denies a request or the resend cooldown rejects it (`OTP_REQUEST_WAIT`). |
| `blocked` | Yes, if the diagnostic read works | `login.js` records `blocked` (`OTP_ATTEMPTS_EXCEEDED`) at three failed attempts and disables that OTP. It is about the OTP challenge, not an account ban. |
| `expiredOrLeft` | Yes (`true`), with an inference caveat | `login.js` records `expired` when an expired OTP is submitted. The report also infers expiry from a matched OTP whose `expDate` is past. It does not prove the person left or never got the SMS. |

Sources: `controllers/customer/auth/sendOtp.js`, `controllers/customer/auth/login.js`, `apis/plasgate/sendOtpApi.js`, `services/customer/otpTrackingService.js`, `services/journey/registrationFunnelService.js`.

## 2. Expected availability object (code-derived, not captured live)

Diagnostic query succeeds:

```json
{ "otpReasonAvailability": { "delivery": false, "gateway": true, "wrongCode": true, "rateLimit": true, "blocked": true, "expiredOrLeft": true } }
```

Diagnostic table fails to load (the four dependent flags turn `false`):

```json
{ "otpReasonAvailability": { "delivery": false, "gateway": false, "wrongCode": false, "rateLimit": false, "blocked": false, "expiredOrLeft": true } }
```

A successful query with zero rows still counts as available. The flag does not prove complete historical tracking or that every write succeeded; diagnostic writes are best-effort. Edge case: when there are no registration flows the loader returns early, so the four dependent keys can be omitted instead of `false`. If the contract needs all six keys on every response, backend needs a small normalization fix (not applied; no portal change needed).

## 3. How the portal should read these values

- Keep SMS delivery as "Not tracked yet", not zero.
- The other reasons may show numbers when their flag is `true`.
- A zero means no matching classified person/flow in the selected cohort, not that no such event happened anywhere.
- Counts are people/flows stopped at OTP requested (latest matching failure, then inferred expiry, then unknown), not raw attempts.
- Keep "unknown" when there is no diagnostic or expiry evidence. Do not infer a failure because verification did not finish.
- Data from before instrumentation is not backfilled.

## 4. Verification and limits

Reviewed API commit `7e94d1ec` (branch `feature/bunlong-chat-telegram`); local dev reference `d027ca04` has the same instrumentation. This does not show which commit or migrations are deployed. Focused checks passed (3 suites, 50 tests, mocks only): `sendOtpTracking`, `loginOtpDiagnostics`, `journeyRegistrationDiagnostics`. These show code behaviour, not live telemetry completeness. The 19 Oct statement that these are not instrumented no longer describes the code.

## Portal note (not from backend)
Dev matches this answer: delivery shows "Not tracked yet" and the other reasons show numbers. The portal needs no change. An omitted flag with a zero count shows "Not tracked yet", which is the safe reading for the no-flows edge case.
