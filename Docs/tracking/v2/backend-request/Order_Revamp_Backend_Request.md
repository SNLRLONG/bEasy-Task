# Order list, order details and Direct Sale: what is blocked by backend

**To:** Backend · **From:** Portal · **Updated:** 5 Nov 2026
**Portal task:** T-0095 (order list and details), T-0096 (Direct Sale) · **Branch:** `feat/order-management-revamp`
**Figma:** `wA47r8vSgQmqYZDMnL99RI`, page `1:2`, section `11:343` (Direct Sale v2 Handoff)

The portal UI is built to Figma using the data and endpoints that exist today. The items below are in the Figma frames but are **not built** because the API has no field or endpoint for them. Please reply per item with **yes / no / later**, the field or route name, and a date. The backend repo is not cloned here, so the portal side is cited by file and line; we could not check the backend code.

| # | Item | Where it shows in Figma | Type |
|---|---|---|---|
| 1 | Scope of work per service line | Order details line, Direct Sale create | Field |
| 2 | Address name (Save as) | Add address dialog and address dropdown | Field |
| 3 | Attachment on managed Direct Sale create | Create, Notes and attachments | Endpoint field |
| 4 | Cross-sell line discount includes the coupon | Order details, service line price | Data bug |
| 5 | Reschedule reason, limit, count | Reschedule dialog | Rules (item 15 in pending) |
| 6 | Deposit link expired and regenerate | Direct Sale details, pending | Endpoint |
| 7 | When the balance opens and Complete rule | Payment card, Complete action | Decision |
| 8 | Customer photos and notes split | Order details | Field |
| 9 | Order list filters | Filters panel | Contract |
| 10 | Staff and dispatch actions | Manage staff drawer | Endpoints |
| 11 | Permission check on write calls | All order actions | Verify |

---

## 1. Scope of work per service line

Figma shows a **Scope of work** block on each service line (what the cleaner must do) and on the Direct Sale create form. The order detail has no such field: a line has `categoryNameEn`, `productEn`, `productOptionEn`, `hourCount`, `cleanerCount`, `qty`, `amount`, `discount`, `addOns` and nothing else. (`items[]` from `GET order/{bulkOrderId}/detail`, typed in `app/types/api.d.ts:362`.)

Please:
1. Accept an optional `scopeOfWork` text (and area, e.g. `areaOfWork`) per service line on Direct Sale create and edit.
2. Return it on each item in `GET order/{id}/detail` and on the staff job page.
3. Tell us the max length and whether it is per line or per order. Proposal: per line, up to 500 characters.

Until then the portal shows no scope block.

## 2. Address name (Save as)

Figma has an optional **Save as** (Home, Condo, Office) in the Add address dialog, and shows that name in the address dropdown next to the address text.

Today: `POST direct-sale/add-users-address` and `GET direct-sale/users-address` (`app/api/endpoint.ts:145-146`). The portal sends no name and the dropdown shows only the address text.

Please:
1. Accept `name` (optional, up to 50 characters) on `add-users-address`. Say the real field name if it differs.
2. Return it on `users-address` so the dropdown can show "Home · Street 313…".
3. Confirm it is the same record the customer app uses, so a name saved in the portal shows in the app and the other way round.

Until then: no Save as field in the dialog, no name in the dropdown.

## 3. Attachment on managed Direct Sale create

Tested on dev: adding a file field `attachment` to the create call with `paymentMode=PAYMENT_LINK` answers `{"code":"ERROR","message":"Unexpected field"}`. Legacy orders send the receipt as `imgUrl` and work.

Please accept one optional file (JPG, PNG or PDF, up to 10 MB) on the managed create, tell us the field name, and store it as a **note attachment** on the order, not as a payment receipt. Return it in the order detail so the Notes card can list it with its file size.

Until then the upload box is not shown for managed orders.

## 4. Cross-sell line discount includes the coupon

Order `0000233329`, item id 6307 (Aircon Cleaning, `isCrossSell: true`): `amount` 10, `discount` **6.5**. The order has one coupon, "1 × Welcome – Enjoy $5 OFF", and the order-level `discount` is also 6.5. So the $5 coupon was added to the cross-sell line's `discount` (1.5 + 5). The portal shows `amount − discount` = $3.50; the cross-sell price should be $8.50 (the expected price from the team lead).

The order total is right (45 − 6.5 = 38.5, plus fees and VAT = 45.65); only the per-line figure is wrong.

Please:
1. Return only the line's own discount in `items[].discount` (1.5 here). Keep the coupon in the order-level `discount` and `couponCode`.
2. Optionally add a net price per line (for example `netAmount` and `netAmountDisplay`). Today there is no such field; the portal computes it (`app/components/common/service-details-v2.tsx:87`).
3. Check the customer app shows the same line; it may have the same issue.

## 5. Reschedule reason, limit, count

Already filed as item 15 in [pending-request-backend.md](pending-request-backend.md): required `reason`, max 2 changes with `rescheduleCount`, 30-day rule, and whether the current endpoint already enforces them. Not repeated here.

## 6. Deposit link expired and regenerate

Figma has `pending · link expired` and a **Generate new link** button. The payment attempts have no Expired status and there is no endpoint to create a new deposit link; only the balance link can be created (`action: 'balance'`, `app/components/common/direct-sale/direct-sale-payment-card.tsx:57`).

Please add an `EXPIRED` state on attempts (with the expiry time) and an endpoint or `action: 'deposit'` to create a new deposit link, with the same `expectedRevision` rule.

## 7. When the balance opens, and the Complete rule

1. The payment summary has `balanceReady` and `balanceAvailableAt`. Is `balanceAvailableAt` the time the balance link opens or the due time? The UI shows "Due {date}" today; Figma wants "Link on {date}". When does `balanceReady` turn true (start of the slot, end, hours before)?
2. Does `PUT order/update` to COMPLETED reject an order with an unpaid balance? The portal only checks status and that the booking start has passed (`app/hooks/use-order-completion.ts`).
3. Is there an `ASSIGNMENT_PENDING` state when no staff is free after the deposit? Figma mentions it; it is not in the data.

## 8. Customer photos and notes split

Figma shows **Photos from customer** and separates the customer note from the internal note ("Internal note, Ops only"). Today there is one `note` (customer) and one `remark`, and no customer photos in the detail. Please return `customerPhotos[]` and confirm `note` = from the customer, `remark` = internal. On create, accept an internal note separate from `note`.

Also asked for in the same frames, lower priority: `previousOrderCount` for the customer, a service-line code (`SL-…`), an order timeline, and open-issue counts.

## 9. Order list filters

1. `paymentStatus` is ignored for app orders on `order/list` (App order total 793 with Paid and with Pending on dev). Please apply it for both types.
2. `status` `IN_PROGRESS` and `CANCELLED` return nothing on dev. Support them, or tell us the right values.
3. `startDate` / `endDate`: is this the booking date or the created date? The portal labels it "Date".
4. Figma also has Staff, Needs-action and Short-of-staff filters and status groups. Optional; say if you can add a `staffId` filter and a `shortOfStaff` flag on the list row.

## 10. Staff and dispatch actions

Not built, no endpoint: Telegram send and receipts, Reserve and send, Retry auto-assign, change Coordinator (Make coordinator), a reason for a hand-picked cleaner, last-change log, chat per person. For the Manage staff drawer we also read only page 1 of the cleaner list; please confirm the page size or add a search parameter.

## 11. Permission check on write calls

The portal now hides write actions without `ORDER:UPDATE` / `ORDER:ASSIGN` / `ORDER:ADD`. Please confirm the backend rejects these calls for users without those permissions (`order/update`, `order/reschedule`, assign and unassign cleaner, `direct-sale` create and edit, receipts, remark). We did not test with a view-only account.
