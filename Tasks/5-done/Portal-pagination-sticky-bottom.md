# Pagination: bottom margin and stick to bottom on all modules

- **ID:** T-0061
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Worktree:** removed after merge (#543)
- **Tier:** T2 · edits shared v2 patterns used by about 30 pages, refactor over 5 files
- **Claimed by:** claude session, 2026-11-02
- **Due:** 2026-11-02
- **Started / updated:** 2026-11-02 / 2026-11-02
- **Docs:** `app/components/v2/README.md`, `docs/responsive-design-system.md` (in the portal repo)
- **Depends on:** none (branch from `main`). Overlaps the open `Portal-rds-*` tasks in 3-in-progress; rebase on their merges and keep edits to the footer only.
- **Touches:** `app/components/v2/patterns/{list-page,data-table-pagination}.tsx`, `app/components/v2/ui/pagination.tsx`, `app/components/v2/ui/tabs.tsx`, `app/components/v2/patterns/data-table.tsx`; pages still on old pagination: `routes/order/order.tsx`, `routes/service-bundle/service-bundle.tsx`, `routes/category-addon/category-addon.tsx`, `routes/product/product.tsx`, `routes/product-option/product-option.tsx`, `routes/users.tsx` (old import left alongside v2)

## Goal
Audit every pagination in the portal. Each one has a bottom margin and sticks to the bottom of the viewport/content area on every device size, using the v2 pagination.

## Audit so far (code read, not yet run)
- v2 `DataTablePagination` + `ListPage footer` is used by about 32 files (customer services, finance, partner, cleaner, coupon, banner, push notification, journey, and others). `ListPage` puts the footer under the card with `pb-sibling`; below `lg` the page scrolls, so the footer scrolls away with it. It is not sticky anywhere.
- Old pagination (`common/table-pagination`, `common/custom-pagination`) is still used by Order (`custom-pagination`) and Service bundle, Category add-on, Product, Product option (`table-pagination`); `users.tsx` has a commented old import.
- Chat list and any non-table list (cards, infinite lists) were not checked: the audit must find them.

## Plan
1. Run every route with pagination at 360 / 428 / 768 / 1024 / 1280 / 1536 and list what is wrong (no margin, scrolls away, hidden behind a bottom bar or the mobile nav, double scrollbars) in the Progress section.
2. Fix once in the v2 patterns: make the footer sticky (`sticky bottom-0`, `bg-bg-surface`, top border, safe-area bottom padding, bottom margin from tokens), inside the correct scroll container for both `<lg` (page scroll) and `lg+` (card scroll). Do not add per-page overrides unless a page has its own scroll shell.
3. Move the five old-pagination pages to `ListPage` + `DataTablePagination`/v2 `Pagination`; remove no old component files (old components are not edited).
4. Pages with a custom footer or no `ListPage` (found in step 1) use the same v2 footer.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new for any new helper)  - [ ] G4 run it (all six widths)
- [ ] G5 code review  - [ ] G7 docs sync (v2 README note)
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-11-02, uncommitted in worktree `bEasy-portal-pagination-sticky/`)
Shell scrolls inside `AppShell`'s `overflow-auto` div, so a `sticky bottom-0` footer works for all pages. Added `ListPageFooter` in `v2/patterns/list-page.tsx` (sticky, surface background, no top border, bottom margin `max(--spacing-sibling, safe-area)`); `ListPage` now wraps its `footer` in it, which fixes the ~26 pages on `ListPage`. `partner-payout.tsx` and `cleaner/cleaner.tsx` had their own footer outside `ListPage`; they now use `ListPageFooter` (cleaner wrapper moved from `p-4` to `px-gutter pt-sibling`). `tsc`, eslint and `test:v2-pagination` clean for these files. **Not run in a browser** at the six widths yet: G4 open.
Left as is on purpose: footers inside cards, tabs or sheets (`push-notification-detail`, `earnings-content`, `journey-event-table`, `stage-list-content`) because they are not page-level.
Deferred, overlap with in-progress tasks: old `TablePagination` pages Service bundle, Category add-on, Product, Product option belong to `Portal-rds-catalog-setup`; Order (`CustomPagination`) belongs to `Portal-rds-orders` (backlog). Their pagination should use `ListPage` when migrated; pointer to add in those tasks.

Also fixed in this branch (2026-11-02): card clips table corners (`overflow-hidden`) and the last row no longer draws its own bottom line; equal space above and below the pagination; Customer page lost its Card View / List View tabs on large or zoomed screens because `TabsList` (`overflow-x-auto`) shrank to 0 height inside the `lg:h-full` flex column, fixed with `shrink-0` in `v2/ui/tabs.tsx` (also protects the other pages using `TabsList`). G4 open.

## Done when
- [x] Audit table (route, before, after) in Progress
- [x] Every paginated page uses the v2 pagination, with a bottom margin and sticking to the bottom at 360, 428, 768, 1024, 1280, 1536
- [x] Footer does not cover the last row (content has matching bottom space) and respects the iOS safe area
- [x] No horizontal page scroll; rows-per-page and page buttons still work
- [x] Only v2 files and the listed pages changed; no old component edited
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
