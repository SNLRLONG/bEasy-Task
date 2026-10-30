# RDS 0: tokens, breakpoints and v2 scaffold

- **ID:** T-0056
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Started / updated:** 2026-10-22 / 2026-10-22
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-foundation` off it
- **Tier:** T2 · new shared tokens/hook, touches `app.css` imports
- **Claimed by:** claude session, 2026-10-22
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** none
- **Touches:** `app/app.css` (import line only), `app/styles/`, `app/lib/breakpoints.ts`, `app/components/v2/`, `app/routes.ts` (dev route), `tests/`

## Goal
Add `app/styles/suntel-tokens.css` (semantic colour, radius, spacing, type, elevation; additive), `app/lib/breakpoints.ts` + `useBreakpoint`/v2 `useIsMobile` (one value for JS and CSS, fixes 768/769), Khmer font fallback, `app/components/v2/{ui,layout,patterns}` with `index.ts`, and a dev-only preview route showing every v2 piece. Add `node --test` for breakpoint helpers.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [x] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-22, uncommitted in worktree)
Built: `app/styles/suntel-tokens.css` (+1 import line in `app.css`), `app/lib/breakpoints.ts`, `app/hooks/use-breakpoint.ts`, `app/components/v2/` scaffold, dev-only `/dev/design-system` route, `tests/breakpoints.test.mjs` (`yarn test:breakpoints`).
Checked: new test 5/5; eslint clean on new files; production build compiles the new utilities and excludes the dev route; preview verified in Chrome at 360/428/768/1024/1280/1536 (no horizontal overflow, breakpoint/gutter/radius/10px-to-12px floor switch as designed).
Not done: `tsc` still reports about 100 errors in untouched files (already on `main`; none in new files); Khmer web font not loaded (open question 4); colour values are provisional aliases until Figma primitive ramps are exported; G5 review and G7 docs sync pending.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
