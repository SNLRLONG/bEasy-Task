# RDS 5a: Customer service module

- **ID:** T-0045
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-26 / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-customer-service` off it
- **Tier:** T2
- **Claimed by:** claude session, 2026-10-26
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS pilot-coupon
- **Touches:** `routes/customer-services/*`, `data-table/customer-*`, `registered-customer-*`, `ticket-columns`

## Goal
Migrate: customer, registered, direct-sale customer, tickets, journey, profile pages and filter sheets. Follow the pilot recipe. Check `git worktree list` for overlapping branches first; own worktree `bEasy-portal-rds-customer-service/`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-26, uncommitted in worktree)
Worktree first synced with main: `origin/main` merged as `3d41187b` (one conflict, `package.json` test scripts; `yarn install` run for the new `yarn.lock`). Full suite after merge: 467 of 469 pass; the 2 failures (`customer-registration-source`, one `customer-segments` assertion) fail identically on plain `origin/main`.
Migrated to v2: **Direct Sale Customers** (page, header with date range + referral + category filters in the Filters sheet, card roles), **Customers** (page frame with Card/List tabs, header actions, active-filter chips, card roles; the old 690-line filter sheet, saved-filters menu and Card View dashboard are embedded unchanged because they already work on a phone), **Registered Customers** (page, header, graph/list tabs, row click opens the detail drawer), **Tickets** (page, stat tiles, search + type filter, card roles; ticket detail sheet still old), **Customer Overview** (stat tiles, tabs, date range).
New v2 pieces: `SelectFieldApi` (same props as `CustomSelectApi`), `DataTable onRowClick` with the same control guard as the old rows plus keyboard activation (`row-click.ts`, 5 unit tests).
Verified in Chrome, logged in, read-only: all five lists at 360 and 1280 (no overflow, cards below lg, table from lg, Filters button below lg); Customers phone view with the old filter sheet open and the Card View tab; Registered Customers row opens the drawer at 360 (328px) and 1280 (440px); Tickets stat tiles 2-up on phone, 4-up on desktop.
Deferred to the Marketing/journey task: **Customer Profile, Customer Journey and Registered Customer Profile** pages (built from `components/journey/customer/*`, also used by the journey CMS; they still use the old `CustomHeader` and `h-screen`). Not tested: any write (remarks, contact date, ticket status, export), the ticket detail sheet, the profile pages.
Known: `tickets.tsx` has one existing type error (`createdAtFormatted`), also on main; not touched.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
