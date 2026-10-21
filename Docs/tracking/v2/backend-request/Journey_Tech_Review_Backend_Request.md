# Customer Journey: open requests for backend

**To:** Backend · **From:** Portal · **Updated:** 20 Oct 2026
Backend's answers: [Journey_Tech_Review_Backend_Response.md](../backend-response/Journey_Tech_Review_Backend_Response.md). Only open items are below.

## 1. Samples: three extras

You will send masked samples after deployment (environment, exact query, `dataAsOf`). Please also include these, so we read the real keys and do not guess:

- the `definitions` object of the funnels, and `conversionDefinition` and `summaryScopes` of the people lists;
- one registration people row with a name and one with `name: null`;
- `otpReasons` and `otpReasonAvailability` of the registration funnel.

## 2. Reconciliation on production, 12 to 18 Oct 2026

Environment: **production** (`api.beasy.info`), timezone Asia/Phnom_Penh. Please list the IDs behind: 53 registrations (registration funnel) vs 40 (acquisition funnel); 9 first purchases vs 10 purchased vs 11 orders.

New, same window: the registration funnel on production shows **135, 135, 135** for OTP verified, Sign-up started and Completed. Please count the raw events `SIGNUP_VIEWED`, `OTP_VERIFIED` and `REGISTRATION_COMPLETED`, and list the registration flow IDs that have `SIGNUP_VIEWED` but no `REGISTRATION_COMPLETED`. Tell us whether the equal numbers come from the data or from how the stages are counted.

## 3. Registration stages from your own records

The app already sends `X-Reg-Flow-Id` and `X-App-Instance-Id` on: `customer/auth/account/{phone}/status/check`, `customer/auth/send/otp`, `customer/auth/account/login`, `customer/auth/account/create`. Do you read and store them today, with one record per call and its result (new or existing, OTP sent, wrong code, expired, locked, verified, account created)? If yes, the registration stages and the OTP failure reasons can be built without an app change. Please tell us what you store now and what is missing.

## 4. Duplicate orders: real-data check

Please find separate bulk order IDs with the same customer, service, slot and total created within minutes. For each pair tell us which request created it (time apart, `tranId`, whether the same `customer/order/v6/create` request came twice).

## Not asked now
Assisted conversion, finance amounts, consent data, registration flow rework, full phone in journey APIs, extra list filters.
