# Order Management revamp: UI and v2 components

- **ID:** T-0095
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-11-04
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Schedule:** 2026-11-02 → 2026-11-04 (3 working days, order 1 of 12)
- **Worktree:** `bEasy/bEasy-portal-order-management-revamp/` · **Branch:** `feat/order-management-revamp`
- **Tier:** T2 · UI and flow adjustments on existing order screens, no payment or refund logic changes (re-tier up if the diff touches them) (see `PIPELINE.md`)
- **Claimed by:** claude (Sonnet 5.5), 2026-11-03
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
- Checked on dev: pending, confirmed and completed app orders, completed and pending DS orders, 430px width. Test order `DS #0000955569` was created for this and still needs cancelling or deleting by the user.
- Left out (no data or API): status groups and row flags, Staff filter, Timeline, Operations summary, Telegram states, "Added services charged to balance" copy.
- Open: date filter label (`startDate`/`endDate` semantics unconfirmed, labelled "Date"); DS confirmed, booking-day, problem-reported and no-name states not checked (no dev data, a deposit needs a real ABA payment); Payment and Notes/attachment cards still use old `ui/*` imports and hex colours; G2 typecheck has unrelated errors on main, touched files are clean; G5 review, G7 docs sync and G11 changelog still to do.

## Notes
Scope cut 2 Nov 2026 to 3 days: the user confirmed this is UI and flow adjustment on screens that already exist, not a rebuild.
Estimate from about 25 current frames in the Direct Sale v2 section (list 4, details about 17, staff drawers 3, Telegram dispatch states 1). No separate Order Management section was found on page `1:2`; if a different frame set exists on the Design page (`1:4`), re-estimate. Overlaps `Portal-rds-orders` (T-0049, backlog; its assessment says the order UI needs no responsive migration, only token cleanup) and `Portal-journey-order-status-alignment`. Cancel and reschedule dialogs belong to `Portal-kpi-cancellations`.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [x] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [x] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog (G9 optional)

## Done when
- [ ] Screens match the Figma frames and states
- [ ] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
