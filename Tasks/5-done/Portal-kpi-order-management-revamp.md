# Order Management revamp: UI and v2 components

- **ID:** T-0095
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-11-04
- **Started / updated:** 2026-11-03 / 2026-11-06
- **Schedule:** 2026-11-02 → 2026-11-04 (3 working days, order 1 of 12)
- **Worktree:** `bEasy/bEasy-portal-order-management-revamp/` · **Branch:** `feat/order-management-revamp`
- **Tier:** T3 · re-tiered up 4 Nov 2026 before testing: the diff adds `usePermission` guards (ORDER:UPDATE / ASSIGN) across order screens, a permissions signal in `PIPELINE.md`. Planned as T2 (see `PIPELINE.md`)
- **Claimed by:** claude (Opus 5.5), 2026-11-03
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) page `1:2` section `11:343` Direct Sale v2 — Handoff; KPI plan: November 2026 (added by the user after the 12 master tasks); `Tasks/_plans/Portal-responsive-design-system.md`
- **Depends on:** v2 UI primitives from the `Portal-rds-*` tasks; order status and staff endpoints already live (verify).
- **Touches:** `app/routes/order/*`, `app/components/order/*`, `common/order-*` (overlaps T-0049 and the cancellations task, check `git worktree list` first)

## Goal
Rebuild the order list and order details on the v2 components: list default, filter open, filters applied, no results, and the details states (pending, confirmed, booking day, completed, problem reported, no-name customer, staff loading/conflict, manage and replace staff drawers).

## Decisions (user, 3 Nov 2026)
- Match the Figma look as closely as possible using v2 components. Functionality the portal does not have is **not** added (e.g. no Needs-action/Short-of-staff/Telegram filters or row flags unless the data already exists).
- **Order list is not grouped by status** as in the Figma frames. Keep the current flat list and its pagination.
- Everything else about filtering goes into the one Filters panel (Figma `246:5220`), limited to filters the API supports today.

## Progress (3 Nov 2026)
- Done: list panel (header, search, filter popover, active chips, flat paginated rows with the old round thumbnail, empty states), DS details header, profile, address, service line styling. Checked on the dev backend against Figma `246:4859`, `246:5220`, `246:5964`, `17:2287`.
- Done: consistency pass. List rows, header and customer card share the v2 `Badge` and the `getBadgeStatusVariant` / `getBadgePaymentStatusVariant` helpers (Pending amber, Confirmed blue, Completed green); search uses v2 `SearchInput`; active chips use `Badge`.
- Done: status filter fix. The orders API only matches `status` in all caps (`PENDING`, `ACCEPTED`, `COMPLETED`); title case returned nothing. Payment status is case-insensitive, sent as `PENDING` / `PARTIALLY_PAID` / `PAID`. Verified on dev. `IN_PROGRESS` and `CANCELLED` returned nothing on dev (no data or not supported), so they are not offered.
- Checked on dev: pending, confirmed and completed app orders, completed and pending DS orders, 430px width. Test order `DS #0000955569` was created for this; it is cancelled now.
- Left out (no data or API): status groups and row flags, Staff filter, Timeline, Operations summary, Telegram states, "Added services charged to balance" copy.
- Done (3 Nov, pm): details body below the header matched to Figma `17:1564` for every order type. Customer card: booking-date and address rows with pill icon tiles, slot length hint, floor/room line, View map link. Services card: DS hint text and "Add service" (opens the DS editor, same rule as the header's Edit, now shared as `canOpenDirectSaleEditor`); each line shows meta, price/Qty and a "Staff · n of m" box with Not assigned / Short / Fully staffed badge and the deposit note. Right column: Payment (Figma sizes, riel estimate under Total, exchange-rate field and ABA PayWay moved into Payment details), new Operations card (service lines, fully staffed, short of staff), Sales agent card, and Notes & attachments (remark edit + receipts as file rows) replacing Remark/Attachment/Assigned Cleaners cards. New strings under `orderDetail.*` in en/km (km wording needs a native check).
- Left out on details (no data): "N previous orders", service-line code (SL-…), Scope of work, Photos from customer, Timeline, Telegram/open-issue counts, customer-vs-internal note split, attachment file size.
- Checked on dev (3 Nov, pm) at 1440px: pending app order `#0000822297`, completed DS `#0000954995`, cancelled DS `#0000955569` (the test order is now cancelled). Not checked: DS awaiting deposit with Add service visible (no such order on dev), narrow widths. Touched files pass tsc and eslint.
- Done (3 Nov, late pm): staff assignment to Figma `1283:62711` (manage) and `1290:63630` (replace). Service line Staff box lists assigned people as rows (name · staff ID, in-house/partner · team, Leader badge, partner acceptance status) with "Manage". Manage staff drawer (v2 Sheet): service/time/staff-needed facts, assigned list with Replace and remove, amber callouts for missing staff, rejected partners and assigned cleaners on day off or booked elsewhere, In-house / Partners pick list with search, skill badge, "Can't assign (n)" group (day off, busy or blocked at this time from the cleaner availability API, no skill for the category). Replace stages the pick and saves on "Replace" (assign new, then unassign old; reason required for partners, as before). Picks and removals outside Replace still save immediately through `useAssignmentEditor`. Checked on dev: `#0000822297`, `DS #0000166619` (drawer opened and Replace mode only; no assignments changed).
- Left out of staff (no API): Telegram send/receipts, "Reserve and send", Retry auto-assign, Coordinator change, reason for a hand pick, last-change log, chat per person, travel buffer, equipment/transport checks.
- Open: date filter label (`startDate`/`endDate` semantics unconfirmed, labelled "Date"); DS confirmed, booking-day, problem-reported and no-name states not checked (no dev data, a deposit needs a real ABA payment); G2 typecheck has unrelated errors on main, touched files are clean; G5 review, G7 docs sync and G11 changelog still to do.

## Review findings (3 Nov 2026, late pm, from code; Figma side not yet checked)
- **Payment filter is Direct Sale only** by an old guard in `use-order-query.ts` (`paymentStatus` is sent only when `type === 'DIRECT_SALE'`; came from the main-branch move commit, not a decision in this task). App orders also carry `paymentStatus` in detail. Not yet known if the list API filters app orders by it. Test on dev, then lift the guard or keep it with a backend note.
- **Current rules on main (kept in v2):** Edit order / Add service opens only if the DS has no managed payment and status is `PENDING`; with managed payment, status not `COMPLETED`/`CANCELLED`/`REJECTED` and (`PENDING` or `ORDER:UPDATE`). Cancel: status `PENDING`/`ACCEPTED`, `payment.canCancel`, and (`PENDING` with nothing paid, or `ORDER:UPDATE`). Staff box is read-only for completed/cancelled and locked while auto-assigning. Rules live in `app/lib/direct-sale-payment.ts`.
- **Correction (4 Nov):** the earlier "no reschedule in the portal" was wrong (a grep failed). Main has all of these:
  - **Reschedule:** `DateTimePicker` in the Booking date card (`order-detail-card.tsx`), `PUT order/reschedule/{id}` with `scheduleStartDate` + `expectedRevision`, availability `order/reschedule/{id}/options`. Disabled for `COMPLETED`, `CANCELLED`, `REJECTED`, `IN_PROGRESS` or any cleaner in progress. No reason, no max-2 counter, no 30-day rule in the portal (backend may enforce).
  - **Service Add-On (per service line):** `ServiceAddOnDialog` (`service-details-v2.tsx`), `POST order/add-extra-addon`. Shown only for app orders (not Direct Sale), `bookingMode` not `INSTANT`, status `PENDING` or `ACCEPTED`, and the category has add-ons. When payment is `PAID` an attachment is required.
  - **Direct Sale "Add service":** opens the DS editor (rule above).
- **Permission gaps (per `docs/new-page-guide.md`):** `MODULES.ORDER` supports VIEW/ADD/UPDATE/DELETE/ASSIGN but only VIEW and UPDATE are checked. `ASSIGN` is never checked, so Manage/Replace/remove staff have no gate. Not gated: remark edit, receipts upload, exchange rate (uses `EXCHANGE_RATE:ADD`), mark paid, complete/accept, New Direct Sale (route handle uses ORDER, check action is ADD). Plan: define one action map (below) and add `usePermission` guards in the drawer, notes card, header; no new module needed (route, sidebar, header title and breadcrumbs already exist).

### Order action → permission map (agreed 4 Nov 2026, guards added; hidden, not disabled)
| Action | Permission |
|---|---|
| View list / details | ORDER:VIEW |
| New direct sale | ORDER:ADD |
| Edit order, Direct Sale Add service (header, details link, edit route) | ORDER:UPDATE (now also for `PENDING`) |
| Manage / replace / remove staff (link hidden, drawer locked) | ORDER:ASSIGN |
| Accept, complete, mark paid (header and mobile footer), app order Service Add-On, edit remark, receipts, create or replace balance link | ORDER:UPDATE |
| Cancel (all order types) and refund | ORDER:UPDATE (now also for `PENDING` and non-managed orders) |
| Exchange rate | EXCHANGE_RATE:ADD (kept) |
| Customer link | MARKETING_CUSTOMER:VIEW (kept) |

## Business-logic decisions (user, 4 Nov 2026; Figma read through the Chrome extension, MCP rate-limited on the View seat)
Comparison artifact: https://claude.ai/artifact/LcWUXC2gG3xNKdANZsbjqL
- **Add service:** conditions and logic stay as on main (`canOpenDirectSaleEditor`); UI follows Figma (Add service dialog, "Added … · on balance" line badge, Instant Cleaning greyed in the category list).
- **Edit order:** Pending only; it edits the whole bulk order, never a single service line. Confirmed orders can only add service or reschedule.
- **Reschedule:** follow the Figma UI and rules (reason required, max 2, within 30 days of the first date, Current/New dialog). Where the backend blocks something, skip it and note it. Reuse the existing endpoint and availability query; the guard stays as on main.
- **Cancel:** keep the current rule (`canCancelManagedDirectSale`, typed refund); follow the Figma look as far as the data allows. No reason-driven fee logic (stays T2).
- **Staff (manage, replace, coordinator):** old logic, new UI. No new permission gate, no Telegram / Make coordinator.
- **Pending, deposit link, assignment states:** no change.
- **Payment filter:** same for every order type, chips never disabled (done 4 Nov, `use-order-query.ts`, `use-order-list-state.ts`, `order-list-filters.tsx`; list API result for app orders still to verify on dev).
- **Permissions (4 Nov 2026, later):** guards added per the map above in `order-detail-header.tsx`, the mobile footer and `OrderDetailContent` in `order.tsx`, `edit-direct-sale.tsx`, `order-staff-box.tsx`, `service-details-v2.tsx`, `order-notes-card.tsx`, `direct-sale-payment-card.tsx` and `direct-sale-payment-dialog.tsx`. Status and payment rules in `direct-sale-payment.ts` are unchanged (tests untouched); the permission is checked at the call sites. Users with only `ORDER:VIEW` now see no write actions. New Direct Sale link and route still only need `ORDER:VIEW` (ORDER:ADD not added yet). Whether the backend rejects these calls without the permission is not verified. Lint and tsc clean; not yet checked in the browser with a view-only role.

## Next
- [x] Review Figma (add service, reschedule, cancel, staff, refund, expired, assignment notes).
- [x] Comparison artifact and decisions.
- [x] Payment filter available for all types.
- [x] Payment filter on dev (4 Nov): the list API ignores `paymentStatus` for app orders (App order total 793 with Paid and with Pending). User decision: keep it as is. Backend ask not filed; add to `pending-request-backend.md` only if wanted.
- [x] Add service on main logic with Figma UI (4 Nov). Direct Sale: the "Add service" link already matches Figma and opens the editor, no change. App orders: `service-add-on-dialog.tsx` restyled and kept as Service Add-On, never called Add service so it is not confused with Direct Sale (link "Add service add-on", title "Service Add-On", balance hint, price each, "Add service add-on · +$X"); conditions and `POST order/add-extra-addon` unchanged. Lint and tsc clean on the file; not yet looked at in the browser (needs an app order in `PENDING`/`ACCEPTED` whose category has add-ons).
- [x] Reschedule (4 Nov): the existing dialog already matches Figma (Current / New, button disabled until a change, full slots greyed, same disabled statuses). Reason, "Changes used X of 2" and the 30-day limit need backend; asked in `pending-request-backend.md` item 15, skipped in the UI until answered. No code change.
- [x] Cancel dialog (4 Nov): Figma look kept on current rules. Added the Payment received / Refund / Balance summary for Direct Sale with managed payment, and "Keep order" replaces "Close"/"No". Refund amount, reason and permission checks unchanged. New `directSaleCancellation.*` strings in en/km (km needs a native check).
- [x] Staff states (4 Nov): Loading (3 grey bars, no buttons, Operations shows "—") added; "Leader" label is now "Coordinator" (en and km, shared string); Completed and Conflict were already covered (read-only, amber callouts for day off or booked elsewhere). No new gates.
- [x] Browser check on dev, 4 Nov, localhost second dev server, 1440px: filter chips enabled for All; app order #0000411149 Service Add-On dialog ("$5.00 each", not saved); DS #0000166619 cancel summary and unpaid wording (not confirmed), reschedule dialog opens as Figma (Current, slots, disabled until changed); Coordinator badge on #0000822297. Staff loading bars are too brief to catch by eye; checked in code only.
- [x] G5 review (4 Nov, medium). Fixed: stale remark draft when switching orders (`key` on the notes card), payment filter was cleared by the type chip, old `?status=Pending` links (now case-insensitive), Operations card showed "short of staff" when staff failed to load. Not changed, for the owner to decide: (1) ABA payment check panels now sit behind Payment details > toggle (earlier Figma move; staff may miss the result); (2) Manage staff drawer reads only page 1 of the cleaner list, so cleaners beyond it can't be picked or flagged; (3) date filter can build from > to and silently drops; (4) new list/filter strings are hardcoded English, only en and km updated; (5) `ServiceDetailsV2` runs `useAssignmentEditor` and the add-on query it no longer needs in v2; (6) riel estimate parses the formatted total string.
- [Skipped by user] Figma frames not opened: create date and time, deposit-paid-after-hold, cleaner-reported-problem, no-name customer, change coordinator.

## Notes
Scope cut 2 Nov 2026 to 3 days: the user confirmed this is UI and flow adjustment on screens that already exist, not a rebuild.
Estimate from about 25 current frames in the Direct Sale v2 section (list 4, details about 17, staff drawers 3, Telegram dispatch states 1). No separate Order Management section was found on page `1:2`; if a different frame set exists on the Design page (`1:4`), re-estimate. Overlaps `Portal-rds-orders` (T-0049, backlog; its assessment says the order UI needs no responsive migration, only token cleanup) and `Portal-journey-order-status-alignment`. Cancel and reschedule dialogs belong to `Portal-kpi-cancellations`.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [x] G1 scope  - [x] G2 static (prettier, eslint and tsc clean on the touched files; repo-wide tsc errors on main are unrelated)  - [ ] G3 tests  - [x] G4 run it (dev, 4 Nov)
- [x] G5 code review (4 Nov, see Next)  - [ ] G6 security  - [x] G7 docs sync (backend request item 15; task file)
- [ ] G6 security (T3, required)  - [ ] G9 human review (T3)  - [ ] G10 release plan (T3)
- [ ] G8 PR message  - [ ] G11 changelog
Moved to `4-testing/` 4 Nov 2026: user commits, pushes and merges to dev to test. Remaining gates: G3 (new tests + edge cases for the permission guards), G6, G7 done, G5 strict re-run on the permission diff.

## Done when
- [ ] Screens match the Figma frames and states
- [ ] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Closed (6 Nov 2026)
Merged to `main` in PR #551 (feat/order-management-revamp). Changelog entry is in the root `CHANGELOG.md`. Backend-blocked items stay in `Docs/tracking/v2/backend-request/Order_Revamp_Backend_Request.md`.
