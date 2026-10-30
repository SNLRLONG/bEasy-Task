# Customer list: row action to open Profile

- **ID:** T-0026
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Worktree:** none, works directly on `feat/journey-cms` in `bEasy/bEasy-portal/` (same as the other journey tasks)
- **Branch:** `feat/journey-cms`
- **Tier:** T1 · small UI change in one column file, no new API, no new route (routes already exist)
- **Claimed by:** none
- **Due:** 2026-10-21
- **Started / updated:** 2026-10-21 / 2026-10-21
- **Docs:** `Docs/tracking/v2/README.md`; parent task `Tasks/3-in-progress/Portal-journey-cms.md`
- **Depends on:** none (routes `customer/:id/profile` and `customer/:id/journey` are in `app/routes.ts`)
- **Touches:** `app/components/data-table/customer-column.tsx` (the `Action` menu and the name cell); maybe the Card View list if it has row actions

## Goal
On the Customer page (List View) a CS user can go straight from a customer row to that customer's Profile page, without opening the detail drawer first.

## Current state
- The row `Action` menu in `customer-column.tsx` has only drawers: View Customer Detail, Overall Rating, Last Contact Date, Order History.
- The only link to `/customer/:id/journey` is inside the detail drawer (`customer-detail-content.tsx:165`). There is no link to `/customer/:id/profile` from this page.
- Both target routes already exist and use the same permission as this page (`MARKETING_CUSTOMER` + `VIEW`), so no extra permission check is needed.

## Scope
- Add one item to the row `Action` menu: **View Customer Profile** → `/customer/{id}/profile`. No Journey item: `/customer/:id/journey` is the old UI (owner, 2026-10-21). Use router `Link`/`navigate`, so Cmd/Ctrl-click and middle-click open a new tab.
- Keep the name cell opening the detail drawer as it does now (no behaviour change).
- Keep the menu colour/style of the existing items.
- Out of scope: new pages, tab deep-links inside Profile, Card View changes (note below if it has row actions).

## Open question (decide at build time, no need to ask)
- Whether a visible button in the row is better than the menu item only. Default: menu items only, because the sticky actions column is 40px wide.

## Gates (T1)
- [ ] G1 scope  - [ ] G2 static (`npm run lint`, `npm run typecheck`)  - [ ] G3 existing tests  - [ ] G4 run it (click both items on dev backend, e.g. customer 3370)
- [ ] G8 PR message  - [ ] G11 changelog
- Skipped G7: no contract or doc change.

## Done when
- [x] The menu item opens the right Profile page for the clicked row's customer id
- [x] Ctrl/Cmd-click opens a new tab; the drawer items still work
- [x] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
