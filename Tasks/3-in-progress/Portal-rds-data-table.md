# RDS 3: responsive data table, filter bar, list-page template

- **ID:** T-0051
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Started / updated:** 2026-10-26 / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-data-table` off it
- **Tier:** T2 · new shared component, 80 files will adopt it
- **Claimed by:** claude session, 2026-10-22
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS shell, RDS primitives
- **Touches:** `app/components/v2/patterns/`, `app/hooks/use-data-table-config.ts` (read-only unless a column-priority meta field is added; note in PR)

## Goal
v2 data table on the existing TanStack config (`useDataTableConfig`/`useTableState` unchanged): table at `lg+`, card mode or sticky-first-column scroll below, skeleton and empty states; filter bar that becomes a bottom sheet below `lg`; list-page template (header + filters + table + pagination).

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [x] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-22, uncommitted in worktree)
Built in `app/components/v2/patterns/`: `DataTable` (same TanStack `table` instance as the old tables; table from lg, card list below, or `mobile="scroll"`), `data-table-mobile.ts` (column roles `title/subtitle/meta/action/hidden` via optional `meta.mobile`, typed in `column-meta.d.ts`, with defaults so unannotated tables still work), `FilterBar` (inline from lg, bottom sheet below; children mount once), `SearchInput`, `DataTablePagination` (page size + range text + responsive pagination), `ListPage` (card with toolbar/table/footer; table scrolls inside from lg with sticky header, page scrolls below). Also `ui/skeleton`, `ui/dropdown-menu`, `ui/select-field` (data-driven select, same props as `CustomSelect`).
Checked: `tests/v2-data-table.test.mjs` 5/5; eslint, prettier clean; no new `tsc` errors; verified against the real Coupon page in Chrome (see the pilot task).
Not done: row selection checkboxes and drag-sortable rows (old `sortable-rows.tsx`) are not in v2 yet; add when a page needs them. Sorting UI is unchanged (columns render their own header).

## Update
2026-10-26: added `sortable` (drag handle on table rows and cards, dnd-kit, same onChange(id,index) contract as the old `DraggableContext`) and `busy` (aria-busy) to `DataTable`; `Avatar` primitive. Drag itself not exercised against the backend.

## Update 2
2026-10-26: `DataTable onRowClick` (rows and cards, ignores clicks on links/buttons/inputs/menus, Enter/Space on the focused row); `SelectFieldApi`.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
