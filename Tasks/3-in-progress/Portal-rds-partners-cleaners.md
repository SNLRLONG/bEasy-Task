# RDS 5d: Partners and cleaners

- **ID:** T-0050
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-26 / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-partners-cleaners` off it
- **Tier:** T2
- **Claimed by:** claude session, 2026-10-26
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS pilot-coupon
- **Touches:** `routes/{partner,cleaner}/*`, `partner-tracking`, `partner-payout`

## Goal
Migrate: partner list/detail/onboarding, tracking, payout, cleaner list/create/edit. Follow the pilot recipe. Check `git worktree list` for overlapping branches first; own worktree `bEasy-portal-rds-partners-cleaners/`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-26, uncommitted in worktree)
Migrated: **Cleaner** list (table/pagination swapped to v2 only; the page's own summary cards, header and date filter were already responsive and are covered by tests, so they stay; `cleaner-expertise` tests updated for the new import, 31/31 pass; table card now has a minimum height so it is usable on short screens), **Partner** list (header rewritten with `FilterBar`, summary card fixed: the Services chart + legend forced a 937px page width on phones, now stacks), **Onboarding**, **Partner Payout** (page layout, stat tiles, table; payout status/approve/complete logic untouched, presentation only).
At 360px sideways scroll before/after: cleaner 1,761 to 0, partner 937 to 0, onboarding 1,652 to 0, payout 1,208 to 0; no page overflow.
Not touched: cleaner create/edit form and `partner/:id` detail (not audited, forms already stack), `/tracking` map (works; its text under 12px is Google Maps attribution, avatar initials and one empty-state line, left for the cleanup task with the other `text-[10px]` uses).
Not tested: payout approve/complete/reject, export, partner detail pages, cleaner create/edit.
Payout is money-adjacent: tier note: re-tier to T3 if the payout actions are touched; this change is presentation only.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
