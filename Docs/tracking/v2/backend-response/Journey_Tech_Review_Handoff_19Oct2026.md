> **Status (portal, 20 Oct 2026):** backend's own summary of branch `fix/journey-data`. **Not verified**: it is not on dev yet and `bEasy-api/` is not cloned. Portal follow-ups and open questions: `../backend-request/pending-request-backend.md` §6. Plan: `Tasks/_plans/Cross-journey-tech-review.md`.

# Customer Journey Tech Review Handoff

Branch: `fix/journey-data`  
Review source: `bEasy_Customer_Journey_Tracking_Tech_Review_18Oct2026.docx`

## What backend now guarantees

- A purchase is counted only from `PURCHASE_COMPLETED`, produced from a backend order whose `paymentStatus` is `PAID`.
- Booking `converted` means purchased in the selected journey. `everPurchased` means the customer had an earlier paid order. Historical purchase does not hide a new abandoned journey.
- Invalid timestamps return `null`; the frontend must display `Time unavailable`. They are not classified as abandoned.
- Unproven acquisition source is `Unknown` (`other:Unknown`), not Organic.
- Acquisition defaults to the `new` customer cohort. Use `customerKind=all` to include returning customers or `customerKind=existing` for returning customers only.
- Every stage includes `dropRate` (dropped/reached at that stage) and `dropPctOfFirst` (dropped/first measurable stage).
- Ad-click data remains unavailable and returns `null` with `unavailableReason: meta_not_connected`.
- Refund status may be counted, but refund amount is `null` with `unavailableReason: not_tracked`; order total is not presented as the refund amount.

## Frontend API changes

### Booking stage people

`GET /funnels/booking/stages/{stage}/people`

- `abandoned=all|yes|no` works for both `state=reached` and `state=dropped`.
- `converted=true|false` filters purchase in this journey.
- Each item includes `converted` and `everPurchased`.
- Response includes `convertedBreakdown: { converted, notConverted, everPurchased }`.

### Registration

`GET /funnels/registration`

- KPI key is `otp_not_verified`, not `otp_failed`.
- `otpReasons` only reports `expired` or `not_verified_unknown` from currently available evidence.
- `otpReasonAvailability` says delivery, gateway, wrong-code and rate-limit details are not instrumented. Do not display those categories as zero.
- Completed users show their saved name. The fallback is `Registered user · {masked phone}`.

`GET /funnels/registration/stages/{stage}/people`

- `purchased=all|yes|no` filters whether the user has any successful paid order.
- Response includes `purchaseBreakdown.purchased` and `purchaseBreakdown.noOrderYet` before the purchased tab is applied.
- Each item includes `everPurchased` and `firstPurchasedAt`.

### Acquisition

`GET /funnels/acquisition` and acquisition stage lists

- Default: `customerKind=new`.
- `customerKind=all|new|existing` selects the cohort.
- `sourceType=all|attributed|organic|unknown` selects source provenance.
- `unit` is `people`; `identityUnit` is `customer_or_device` because an unlinked installation remains a device identity.
- `purchaseCounts` separates `firstTimeBuyers`, `buyers`, `orders` and `purchaseEvents`.
- Each after-purchase row declares an order denominator.
- Each person item includes `customerKind`, `converted`, `firstPurchase`, `everPurchased` and `firstPurchasedAt`.

## Review status

| Review item | Status | Backend result |
| --- | --- | --- |
| DQ-01 Identity | Implemented | Registration completion resolves User ID/name with an explicit fallback. |
| DQ-02 Invalid time | Implemented | Invalid time is `null`, never `NaN`. |
| DQ-03 Cohort discrepancy | Implemented definition | Acquisition defaults to new users and exposes its seven-day cohort window. Totals still require environment reconciliation. |
| DQ-04 Purchase definitions | Implemented | First buyers, buyers, orders and events are separate. |
| DQ-05 Ad clicked | Existing expected behavior | Not connected is explicit; no false zero or conversion rate. |
| DQ-06 Source attribution | Implemented | Attributed, Organic and Unknown are separate. |
| DQ-07 Details versus slot | Backend events already separate | Mobile QA must prove the two events fire independently. |
| DQ-08 Drop denominator | Implemented | Both stage-relative and whole-funnel drop percentages are returned. |
| DQ-09 Historical purchase | Implemented | `converted` and `everPurchased` are separate. |
| DQ-10 After-purchase denominator | Implemented where supported | Order denominator is explicit; unsupported refund/cancellation amounts stay unavailable. |
| DQ-11 Identity unit | Implemented in response | Funnel returns customer/journey or customer-or-device unit definitions. Multi-device reconciliation still requires QA. |

## Deliberately not fabricated

- SMS delivery, gateway/API, wrong-code and rate-limit reasons need new OTP instrumentation.
- Paid ad clicks, spend, ad sets, CAC and ROAS need Meta/Google/attribution connectors.
- Actual booking refund and cancellation-fee amounts need a reliable finance source.
- CS-assisted conversion needs a traceable contact/case-to-order identifier and an approved attribution window.
- Permissible-contact and suppression eligibility need consent/opt-in data. Recoverable counts currently exclude anonymous devices and require an abandoned registered journey, but cannot claim consent eligibility.
- The cross-department Action Dashboard is deferred and was not added.

## Demo checks before sign-off

1. Confirm five completed registrations open the correct customer profiles and names.
2. Verify one invalid/missing timestamp displays `Time unavailable` and sorts safely.
3. Compare new versus returning acquisition users using `customerKind`.
4. Reconcile first buyers, buyers, order IDs and purchase events for one seven-day window.
5. Test leaving booking details before slot selection to verify the events are independent.
6. Demonstrate one historical buyer starting a new journey: `everPurchased=true` while `converted=false` until that journey is paid.
7. Confirm Unknown source is not shown as Organic.
8. Confirm unavailable OTP/ad/refund facts are not rendered as zero.
