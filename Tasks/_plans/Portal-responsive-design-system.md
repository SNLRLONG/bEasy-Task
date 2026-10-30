# Portal: responsive layout + Suntel design system (v2 components)

Full audit, numbers and approach: `bEasy-portal-responsive-design-system/docs/responsive-design-system.md` (single home; this plan links, it does not copy).

## Goal
Make every routed portal page work from 360px to 1536px+, and bring typography, colour, spacing, radius, buttons, inputs and other patterns onto the Suntel design system, by building new components in `app/components/v2/` and migrating pages one at a time.

## Flow (who does what)
1. Foundation: tokens, breakpoints, `v2/` scaffold, preview route: **new**
2. Shell (`dashboard-layout`, header, sidebar, page container): **change** (swap to v2)
3. Primitives and form fields in v2, one of each concept: **new** (old ones stay)
4. Responsive data table + filter bar + list-page template: **new**
5. Pilot page (Coupon) migrated end to end, recipe written down: **change**
6. Module rollout, one worktree per module group: **change**
7. Cleanup of unused old components once imports are zero: **change**

## Screens and states
All routed pages in `app/routes.ts` (about 60 routes, 9 groups). Per page: loading, empty, error, permission-hidden actions, and 6 widths (360, 428, 768, 1024, 1280, 1536). Module list in the doc, section 3.

## Data and API
None. No backend, endpoint or permission changes. Tracking events: none.

## Gaps and decisions
- **Gap:** no portal Figma for responsive layouts is referenced anywhere. The Figma "Design System" file (`jZcf2OMkMyi5QE2TfefxiF`) has tokens and components only, and defines 3 breakpoints, no grid or gutters. Decision: follow the written Suntel tokens; layout patterns come from the doc, section 4.4.
- **Decision:** keep Tailwind's default breakpoints plus one new `xs` (420) instead of Suntel's 3-breakpoint model, because the request lists 6 widths. One source: `app/lib/breakpoints.ts`.
- **Decision:** `useIsMobile` (`< 769`) and Tailwind `md` (`>= 768`) disagree; v2 reads one value.
- **Decision (worktree model):** `bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`) is the umbrella integration branch, as requested. Phase tasks branch from it and PR into it; slices go to `main` only when a module is complete. This is the one exception to "one task = one worktree"; Phase 5 modules do get their own worktrees.
- **Decision:** Phase 5 tasks sit in `1-backlog/` until the pilot proves the recipe.
- **Decision:** v2 is light theme only; dark parity is out of scope.
- **Conflict risk:** `feat/journey-cms`, `feat/rbac-users-*`, `feat/acquisition-export` touch route folders listed in Phase 5. Check `git worktree list` and `Touches` before starting a module.

## Open questions (recommendation first)
1. **Button and badge radius: Suntel says fully round; portal is `rounded-md`.** A visible global change. Recommend: follow Suntel in v2, keep a `square` variant for toolbars; get design sign-off before Phase 2.
2. **Mobile tables: card list or horizontal scroll?** Recommend card list for list pages, horizontal scroll with a sticky first column only for dense finance tables.
3. **12px text floor vs 80 uses of 9, 10, 11px.** Recommend promote to 12px and trim content where it does not fit, per Suntel rule.
4. **Khmer font fallback.** Recommend Noto Sans Khmer in the foundation task; confirm.
5. **Automated visual checks (Playwright)?** Recommend a small screenshot/overflow suite for shell + pilot only; otherwise manual viewport matrix.

## Task breakdown
| Task | Repo | Depends on | Tier | Stage |
|---|---|---|---|---|
| Portal-rds-foundation | portal | none | T2 | 2-todo |
| Portal-rds-shell | portal | foundation | T2 | 2-todo |
| Portal-rds-primitives | portal | foundation | T2 | 2-todo |
| Portal-rds-form-fields | portal | primitives | T2 | 2-todo |
| Portal-rds-data-table | portal | shell, primitives | T2 | 2-todo |
| Portal-rds-pilot-coupon | portal | shell, primitives, form-fields, data-table | T1 | 2-todo |
| Portal-rds-customer-service | portal | pilot | T2 | 1-backlog |
| Portal-rds-orders | portal | pilot | T2 | 1-backlog |
| Portal-rds-finance | portal | pilot | T2 | 1-backlog |
| Portal-rds-partners-cleaners | portal | pilot | T2 | 1-backlog |
| Portal-rds-marketing | portal | pilot | T2 | 1-backlog |
| Portal-rds-catalog-setup | portal | pilot | T1 | 1-backlog |
| Portal-rds-dashboard-calendar-chat | portal | pilot | T2 | 1-backlog |
| Portal-rds-cleanup | portal | all modules | T2 | 1-backlog |

Order: foundation, then {shell, primitives} in parallel (different folders: `v2/layout` vs `v2/ui`), then form-fields and data-table in parallel, then pilot. After the pilot all module tasks run in parallel in separate worktrees; cleanup last.

## Out of scope
Backend, new features or copy, route or permission changes, dark mode, other repos, deleting unrouted legacy route files, tracking.
