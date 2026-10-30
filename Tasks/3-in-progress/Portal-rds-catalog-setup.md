# RDS 5f: Catalogue and setup

- **ID:** T-0043
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Started / updated:** 2026-10-26 / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-catalog-setup` off it
- **Tier:** T1
- **Claimed by:** claude session, 2026-10-26
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS pilot-coupon
- **Touches:** `routes/{category,category-addon,product,product-option,item,blocked-schedule,rbac,activityLog}/*`

## Goal
Migrate: category, add-on, product, option, item, blocked schedule, RBAC, activity log. Follow the pilot recipe. Check `git worktree list` for overlapping branches first; own worktree `bEasy-portal-rds-catalog-setup/`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-26, uncommitted in worktree)
Scope corrected after reading `nav-config.ts`: Category, Category Add-on, Product and Product Option are commented out of the sidebar (legacy, still routed but unreachable from the menu; add-on uses dummy data). The live Setup pages are Items, Blocked Schedule, Popup Banner, Activity Log and Roles & Permissions.
Migrated to v2: **Items** (list, header with date range, columns, form), **Blocked Schedule** (list, header, columns, form), **Activity Log** (list page, header, column mobile roles; detail sheet and result badge still old). New v2 pieces built for this: `DateRangePicker` (one range calendar, bottom sheet on phones, tests for presets), `Avatar`, drag-to-reorder support in `DataTable` (`sortable`, handle on rows and cards, keyboard sensor), `busy` prop.
Deferred on purpose: **Roles & Permissions** (overlaps the `rbac-users-edit` and `rbac-users-shell` worktrees; do after they merge), **Popup Banner** (moves to the Marketing task: 713-line form, tracking tab, needs the uploader), legacy Category/Product pages (do last or delete with the cleanup task).
Checks: eslint and prettier clean; no new `tsc` errors; `node --test tests/*.test.mjs` = 327 pass / 16 fail, and the same 16 fail on the base commit `7b3f820c` (303 pass / 16 fail there), so no regressions; the 16 are order/instant and customer-segments tests that main has since changed.
Browser verification (2026-10-26, logged in, read-only, nothing saved): Items, Blocked Schedule and Activity Log lists at 360/768/1024/1280 have no page overflow, 30/15/30 cards below lg, table from lg, Filters button below lg; date-range picker opens as a bottom sheet at 390px (6 presets, 35 days, fits); Item and Blocked Schedule forms at 390px stack correctly with a sticky Save header; multi-language dialog (5 inputs) fits with Save visible. Small fix: date-range dialog and filter sheet now focus the container instead of the first control. NOT exercised: Save/create/edit on any form, drag-to-reorder, the Activity Log detail sheet on a phone. Drag-to-reorder is untested end to end (reordering writes to the backend; it is only used by the hidden Category/Add-on pages and Banner for now).

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
