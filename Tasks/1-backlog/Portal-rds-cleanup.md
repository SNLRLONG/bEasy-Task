# RDS 6: remove old components and leftovers

- **ID:** T-0044
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-cleanup` off it
- **Tier:** T2 · deletes shared components, lint rule
- **Claimed by:** claude session, 2026-10-26
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** all RDS module tasks
- **Touches:** `app/components/{ui,common,data-table}/*`, `eslint.config.js`

## Goal
Delete old components with zero imports (one commit each), duplicates listed in doc section 2.4, and add ESLint/grep guard against raw hex, palette classes and sub-12px text in `app/`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-26, uncommitted in worktree)
Done early because it was cheap and safe:
- **Text under 12px:** replaced all 97 uses of `text-[9px]`, `text-[10px]` and `text-[11px]` with `text-xs` (12px) in 25 files; only `routes/rbac/*` is left (other branches are changing it). Chat went from 138 elements under 12px to 0, Calendar from 30 to 0, no overflow, layouts intact (checked in Chrome at 390 and 1100px).
- **Lint guard:** `eslint.config.js` now fails the build if anything under `app/components/v2` uses an arbitrary font size, a raw hex colour, a raw palette colour (`bg-red-500`), `100vh`/`h-screen`, or an arbitrary radius. Verified: v2 passes; a test file with each violation triggers every rule.
Not done yet: replacing old colours/hex (253 hex, 550 palette classes) in old components, removing old components with zero imports, deleting the unrouted legacy pages (category, product, service-bundle, voucher, promotion, location, referral-program, `new-push-notification copy.tsx`), the `rbac` files. Those should go in small commits, each with a grep showing zero imports first.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
