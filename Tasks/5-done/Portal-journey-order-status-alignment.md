# Customer profile Orders tab: same order statuses as Order management

- **ID:** T-0029
- **Repo:** bEasy-portal (+ one backend ask)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-order-status/` · **Branch:** `fix/journey-order-status`, from `origin/feat/journey-cms`
- **Tier:** T2 · needs a backend change; reuses shared status helpers (see `PIPELINE.md`)
- **Claimed by:** agent (claude), 2026-10-20. Owner said: work directly on `feat/journey-cms`, no separate worktree, portal side only
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-20 / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` · `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` (`GET /customers/{id}/orders`)
- **Depends on:** backend: send the real order status (item A10 in `Journey_Tech_Review_Backend_Request.md`)
- **Touches:** `app/components/journey/customer/orders-tab.tsx`, `app/types/journey.ts` (`CustomerOrderRow`), `app/locales/journey/{en,km}/customer.json`, the after-purchase orders sheet (same labels, spec §9.5). Read-only use of `app/lib/utils.ts` helpers: no edit to shared code

## Goal
One order shows the same status on the customer profile and on the Order management page.

## Cause found
- Order management shows the raw order status through `getStatusDisplayText` and `getBadgeStatusVariant` (`app/lib/utils.ts`): Pending, Confirmed (`ACCEPTED`), In Progress, Completed, Cancelled.
- The Orders tab shows a backend-made summary: `upcoming | completed | cancelled | refunded` (`orders-tab.tsx`, spec). "Upcoming" merges Pending, Confirmed and In Progress. "Refunded" is a payment state, not an order state, so one order can only show one of them.

## Scope
1. Backend adds `orderStatus` (`PENDING | ACCEPTED | IN_PROGRESS | COMPLETED | CANCELLED`) and `paymentStatus` to each row of `GET /customers/{id}/orders`, and keeps `status` until the portal has switched.
2. Portal shows `orderStatus` with the existing helpers, so labels and colours match Order management, and shows refund as a separate payment badge. Unknown value prints raw, never hidden.
3. Order id links to the order page (`MARKETING`/order permission as that page requires). Khmer labels reuse the order module's.
4. Same labels in the after-purchase orders sheet.
5. Update the API spec and the `Orders` section of the customer page doc.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G6: no new personal data.

## Done when
- [ ] For 5 orders in different states, the profile and Order management show the same status text
- [ ] A refunded order shows its order status plus a Refunded payment badge
- [ ] API spec updated; backend request closed
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress 2026-10-20 (agent, on `feat/journey-cms`, uncommitted)
Backend answered A10: `orderStatus` (raw) and `paymentStatus` are added to `GET /customers/{id}/orders`, `status` kept. Done: Orders tab shows `orderStatus` with the Order management words and colours (Pending, Confirmed, In Progress, Completed, Cancelled; unknown prints raw in grey) plus a payment badge (Paid, Refunded, Unpaid, ...). Falls back to `status` while `orderStatus` is missing. Helpers in `app/lib/journey/order-status.ts`, with a test (`test:journey-contract`).
Not done: the after-purchase orders sheet still prints its raw `status` string; the order id link is unchanged (already opens the order by bulk order id); not run against a backend (G4). G5, G8, G11 open.
