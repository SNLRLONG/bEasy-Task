# Duplicate orders: checkout creates one, payment success creates another

- **ID:** T-0010
- **Repo:** cross (backend first, then portal Orders tab; app only if the app is the cause)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none until a portal or app change is needed
- **Tier:** T3 · order and payment data, order counts used by reports (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-10-20
- **Docs:** `Journey_Tech_Review_Backend_Request.md` item C2 · `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` (`GET /customers/{id}/orders`) · `Tasks/2-todo/Portal-journey-order-status-alignment.md`
- **Depends on:** backend investigation
- **Touches:** docs for now. Possibly `app/components/journey/customer/orders-tab.tsx` and the after-purchase sheet

## Goal
Find out whether the same purchase appears as two orders, and decide where it is fixed. Customer Profile > Orders must show one row per real purchase, and Order management must agree.

## Reported scenario
In the app, tapping checkout creates an order. After the payment succeeds, a second order is created with the paid status. Staff see two rows for one purchase.

## What we know
- The app calls `customer/order/v6/create` from the checkout screen (`confirm_and_pay_checkout_screen.dart:1696`, `payment_controller.dart:2018`). I found only that one call site in the app, so the second order is probably created on the backend or by the payment callback. Not verified: the backend repo is not cloned.
- Why it matters beyond the tab: it can inflate order counts. The review shows 11 orders for 10 buyers (DQ-04, DQ-10). The handoff counts a purchase only for a `PAID` order, so funnel purchases are probably safe, but "orders" and the after-purchase denominators may not be.

## Steps
1. **Backend (item C2):** run a query on dev/prod for customers with two orders of the same service, slot and total created within minutes, one unpaid and one `PAID`. Report: how many, which created each (create API vs payment callback), what `status` and `paymentStatus` each has, and whether Order management lists both.
2. Agree the rule with backend and product, one of: (a) the payment callback updates the existing order instead of creating a new one (preferred, fixes the data); (b) the unpaid order is marked as superseded and hidden from the Orders tab, Order management and counts; (c) the portal hides unpaid orders that have a paid twin (last resort, hides data).
3. Apply the fix where it belongs. If the portal filters, add the rule to the API spec and write a test with a duplicate pair.
4. Re-run the order counts for 12 to 18 Oct (`Cross-journey-review-signoff` part 2).

## Gates (T3)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (if code)  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security (order and payment data)  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Backend report with real duplicate examples (order ids) and the cause
- [ ] Decision recorded in the API spec
- [ ] Profile Orders tab and Order management show one row per purchase for the sample customers
- [ ] Order and buyer counts for 12 to 18 Oct reconcile

## Update 2026-10-20 (backend response)
Backend code review: `customer/order/v6/create` creates the order; the PayWay callback and scheduler find it by `tranId` and update its payment status, so the callback does not create a second one. Still to check on real data: separate bulk order IDs with the same customer, service, slot and total within minutes. Likely cause to test: the app sending the create request twice (timeout retry, double tap). Preferred fix if confirmed: an idempotency key on order creation (a retry returns the existing order). The portal does not hide duplicates. New app step added to `App-journey-event-qa`.
