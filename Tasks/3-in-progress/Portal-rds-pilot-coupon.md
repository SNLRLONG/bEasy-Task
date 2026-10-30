# RDS 4: pilot, Coupon list + form

- **ID:** T-0053
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Started / updated:** 2026-10-22 / 2026-10-22
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-pilot-coupon` off it
- **Tier:** T3 · coupon fee/price form (re-tiered up from T1 on 2026-10-22: PIPELINE signal "coupon or price math"). Visual swap only, logic untouched, but G5 review, G6 security, G9 human review and G10 release/rollback plan now apply.
- **Claimed by:** claude session, 2026-10-22
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS shell, primitives, form-fields, data-table
- **Touches:** `app/routes/coupon/*`, `app/components/data-table/coupon-*`, docs

## Goal
Migrate `routes/coupon/*` and its columns/header/form to v2 end to end. Verify behaviour parity and the viewport matrix, then write the migration recipe (imports to swap, patterns, pitfalls) into the doc, section 4.2, for the module tasks.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-22, uncommitted in worktree)
Coupon LIST migrated: `routes/coupon/coupon.tsx`, `components/data-table/coupon-header.tsx`, `coupon-columns.tsx` now use v2 (`ListPage`, `FilterBar`, `SearchInput`, `SelectField`, `DataTable`, `DataTablePagination`, v2 `Badge`/`Button`/`Dialog`/`DropdownMenu`). Data hooks untouched. `tests/coupon-admin.test.mjs` mocks updated for the new imports; all 11 original assertions kept and pass (31/31 across all new and coupon tests).
Verified read-only in Chrome against the dev backend: 390px shows 30 cards with a Filters button, filter sheet (2 selects at 44px, Reset, Done) and per-card action menu; 768px cards; 1024px and 1280px table with sticky header, inner scroll, pagination "1-30 / 58"; no page overflow at any width. Recipe written to `docs/responsive-design-system.md` section 4.2.1.
Coupon FORM migrated too (`routes/coupon/new-coupon.tsx`, presentation only: imports, `SelectField`, `StatCard`, `FormPageHeader`, `Card*`, layout classes; the diff has no change to submit, validation, fee, permission or read-only logic). Tests mocks updated; 11/11 coupon assertions pass (wallet read-only, fees, assign, payload).
Verified read-only in Chrome (view of coupon 3406 and /coupon/new, no save): 390px stacked fields, sticky header with Save, 2-up stat tiles, no overflow; 1280px two-column form with the Apply To card; date picker popover fits 360px (342px wide, 44px days). Layout moved from md to lg for the 3-column grid because the sidebar leaves ~500px for fields at 1024.
NOT exercised: Save, create, edit, assign (read-only testing only); wallet-coupon read-only banner and segment pickers were not visually checked. Because this is now T3, a human should click through create/edit/assign on the dev backend before merge. G5, G6, G9, G10 pending.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
