# RDS 5c: Finance module

- **ID:** T-0047
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Started / updated:** 2026-10-26 / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-finance` off it
- **Tier:** T2
- **Claimed by:** claude session, 2026-10-26
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS pilot-coupon
- **Touches:** `routes/finance/*`, `data-table/{finance,topup,bcombo,direct-sale}-*`

## Goal
Migrate: dense finance tables (sticky-column scroll), top-up, b-combos, direct sales, all-user, coupons. Follow the pilot recipe. Check `git worktree list` for overlapping branches first; own worktree `bEasy-portal-rds-finance/`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-26, uncommitted in worktree)
Migrated all six Finance lists to v2 (`ListPage`, `FilterBar`, `DataTable`, `DataTablePagination`): finance-orders, direct-sales, all-user, coupons (announcement coupons), top-up, b-combos. Headers rewritten with a search + date range (+ status filter for all-user) + permission-gated Export; every export handler and permission check kept as it was. Card layout per table: title, subtitle and three key fields (columns without a role are left out of the card; the table still shows every column from 1024px).
Result at 360px: sideways scroll was 4,527 / 3,248 / 2,549 / 2,008 / 1,861 / 1,580 px, now 0 on all six, no page overflow. Desktop table renders (direct-sales 18 columns, 30 rows).
Not tested: Export (it writes a file download), the `top-up/new` route (it redirects to `/top-up`, as on main), sorting. Pre-existing type errors in `finance-columns.tsx` (2) are unchanged.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
