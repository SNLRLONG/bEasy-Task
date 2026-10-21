# Customer Journey — Backend Response for Frontend Review

**Branch reviewed:** `fix/journey-data`  
**Date:** 20 Oct 2026  
**For:** Admin Portal / Frontend team

## Status summary

The backend contract supports the requested customer-journey screens. This document explains what each value means, how frontend should use it, and which items still require verification after deployment.

## API contract

### 1. Purchased and converted

- `purchased` means a backend-confirmed successful payment, represented by `PURCHASE_COMPLETED`.
- App checkout or payment-attempt events do not count as purchased.
- With `countBy=journey`, `converted` means that journey reached purchased.
- With `countBy=customer`, `converted` means at least one selected journey for that customer reached purchased.
- With the same funnel filters, `convertedBreakdown.converted` should equal the funnel purchased count before list-only filters such as `q`, `abandoned`, or `converted` are applied.

Frontend should read:

- Funnel: `definitions.converted` and `definitions.convertedMatchesPurchased`
- People list: `conversionDefinition`

### 2. After-purchase denominator

Each after-purchase row uses orders as its denominator:

```json
{
  "key": "auto_promo",
  "orders": 5,
  "ofPurchases": 0.4545,
  "denominator": {
    "unit": "orders",
    "count": 11
  }
}
```

Frontend display: **5 of 11 orders (45%)**.

### 3. Drop percentages

- `dropRate = dropped / reached at that stage`
- `dropPctOfFirst = dropped / reached at the first funnel stage`
- Values are ratios from `0` to `1`; multiply by 100 for display.
- Both values are `null` on the final stage because no following stage exists.
- The fields are available for booking, registration, and acquisition funnels.

### 4. Acquisition counting unit

- `unit` is `people`.
- `identityUnit` is `customer_or_device`.
- A linked identity counts as a customer; an unlinked identity counts as a device.
- The query parameter `unit=people` is accepted. Other values return `422 INVALID_FILTER`.

`purchaseCounts` shape:

```json
{
  "firstTimeBuyers": 9,
  "buyers": 10,
  "orders": 11,
  "purchaseEvents": 11
}
```

These values have different meanings and do not need to be equal.

### 5. Source filters

- Unknown source key: `other:Unknown`
- Organic source key: `other:Organic`
- Never display Unknown as Organic.
- Unknown is included in `GET /meta/filter-options` under `campaigns`.
- Source counts are returned in `filterCounts.sources`.
- `sourceType` and `sources` use AND logic.

Example:

```text
sourceType=unknown&sources=other:Unknown
```

This returns only people whose source is Unknown.

### 6. Missing activity times

- `lastActivityAt` and `idleMinutes` can be `null` when the time is unavailable or invalid.
- Valid times are sorted newest first.
- Rows with unavailable times are sorted last.
- Frontend should display **Time unavailable** and should not move these rows above valid rows.

Other fields may also be `null` when the source data is unavailable, including name, masked phone, device, service, value, and source details.

### 7. Registration name and search

- Every registration people row contains the `name` key.
- It contains the saved customer name when available; otherwise it is `null`.
- Backend does not return fallback text such as `Registered user · phone`.
- `q` searches both customer name and phone.
- It also searches OTP-only phones for people who did not finish registration.

These phone formats are treated as the same number:

```text
012 345 678
012345678
+855 12 345 678
85512345678
```

### 8. Customer order status

`GET /customers/{id}/orders` returns:

```json
{
  "status": "upcoming",
  "orderStatus": "ACCEPTED",
  "paymentStatus": "PAID"
}
```

- `status` is the existing journey display status and is kept for backward compatibility.
- `orderStatus` is the raw order-management status.
- `paymentStatus` is the raw payment status.

Frontend should migrate to `orderStatus` and `paymentStatus` while continuing to accept `status` during the transition.

### 9. People-list summary scopes

People-list responses include `summaryScopes` so frontend does not need to guess which filters affect each number.

- Booking `abandonStrip`: after base filters and search, before `abandoned` and `converted` filters.
- Booking `convertedBreakdown`: after `abandoned`, before the `converted` filter.
- Registration `purchaseBreakdown`: before the `purchased` filter.
- `reach`, `breakdown`, and `count`: after all list filters and before pagination.
- Acquisition `reach`, `breakdown`, and `count`: after all list filters and before pagination.

## Confirmed backend behavior

### Booking details and slot mapping

- `details_completed`: `BOOKING_DETAILS_COMPLETED`
- `slot_selected`: `SLOT_SELECTED`, `BOOKING_SUMMARY_VIEWED`, or `PROMO_APPLIED`
- Date and time selection remain under `booking_started`.

If details and slot show equal totals, the backend is not merging them. It means the selected data contains the same number reaching both stages. Mobile QA should confirm the two main events fire independently.

### OTP data

The importer supports:

- `OTP_SUBMITTED` with `metadata.attemptNo`
- `OTP_FAILED` with `metadata.errorCode`

The current funnel loads `OTP_SUBMITTED`, but does not yet classify `OTP_FAILED`. OTP reordering and detailed failure reasons are outside this review and require the planned OTP funnel redesign.

### Recoverable bookings

A booking is currently recoverable when:

1. It belongs to a registered customer; anonymous devices are excluded.
2. Its furthest stage is details completed, slot selected, or checkout started.
3. It has exceeded that stage's configured abandonment time.
4. It has not reached purchased.

With `countBy=customer`, multiple journeys for one customer are grouped. With `countBy=journey`, attempts remain separate.

`openValue` is the sum of stored values for eligible entities, rounded to two decimals.

Important limitation: the current count does not confirm consent, suppression status, or the existence of a permissible contact channel. Frontend should call it **Recoverable**, not **Contactable**, until those business rules are added.

## Frontend checklist

- Use backend `definitions`, `conversionDefinition`, and `summaryScopes`.
- Convert ratio values to percentages for display.
- Show `Time unavailable` for nullable activity times.
- Use raw `orderStatus` and `paymentStatus` when showing order-management state.
- Keep supporting legacy `status` during migration.
- Display `other:Unknown` as Unknown, never Organic.
- Do not assume first-time buyers, buyers, orders, and purchase events should match.
- Do not present unreconciled screenshot totals as authoritative backend totals.

## Items pending deployment verification

### Real masked JSON samples

After deployment, backend must capture one masked real response for:

- Booking funnel
- Registration funnel
- Acquisition funnel
- One Reached people list for each funnel
- One Dropped people list for each funnel

Each sample must include the environment, exact query parameters, and `dataAsOf`.

### Oct 12–18 reconciliation

The environment that produced the screenshots was not identified. Backend must run the comparison in that exact environment and timezone using unique event IDs, customer IDs, order IDs, and bulk order IDs.

The different figures can be valid because:

- Registration and acquisition use different cohorts.
- First-time buyers, buyers, orders, and purchase events use different denominators.

### Reported duplicate orders

Code review shows:

- `customer/order/v6/create` creates the order.
- PayWay callback and scheduler reconciliation locate existing orders by `tranId` and update their payment status.
- The callback does not intentionally create a second order.

Real data still needs to be checked for separate bulk order IDs with the same customer, service, slot, and total created within minutes.

If repeated create requests are confirmed, the preferred fix is an idempotency key on order creation so a retry returns the existing order. The payment callback should continue updating that existing order.

The portal should not hide duplicate records by itself.
