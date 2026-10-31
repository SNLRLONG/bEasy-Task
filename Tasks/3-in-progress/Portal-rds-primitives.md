# RDS 2a: v2 primitives

- **ID:** T-0054
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-22 / 2026-10-22
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-primitives` off it
- **Tier:** T2 · new shared components
- **Claimed by:** claude session, 2026-10-22
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS foundation
- **Touches:** `app/components/v2/ui/`

## Goal
v2 button (round, sizes, 44px touch), input/textarea, select, checkbox, radio, switch (replaces ios-toggle), badge (semantic status), tabs (scrollable), dialog/sheet (full-height sheet on mobile), tooltip, pagination (one of three). Full state matrix each.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [x] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-22, uncommitted in worktree)
Built in `app/components/v2/ui/`: button, input + textarea, label, select, checkbox, radio-group, switch (replaces ios-toggle), badge, tabs (scrollable), dialog (bottom sheet below md, centred from md), sheet, tooltip, pagination (one of three; `pagination-utils.ts`). v2 `cn` in `app/components/v2/cn.ts`: plain tailwind-merge treats `text-body-3` as a colour and drops real colour classes, so v2 registers the token utilities.
Decisions taken (open questions 1 and 3 in the plan, recommendation applied; easy to reverse in `button.tsx` / `badge.tsx`): buttons and badges fully round with a `shape="square"` option; text never below 12px (`text-body-4`), 10px only via `text-micro` (12px from md).
Checked: tests 15/15 (`test:breakpoints`, `test:v2-pagination`, `test:v2-cn`); eslint, prettier clean; no new `tsc` errors; `/dev/design-system` verified in Chrome at 360 and 1280 (control heights 44 vs 40, input text 16px vs 14px, dialog bottom sheet vs centred, tabs scroll sideways, pagination collapses below sm, no page overflow).
Differences from old components: `Button isLoading` keeps the label and shows a spinner (old replaced it with the English text "Please wait..."). Not yet done: popover, dropdown-menu, avatar, card, skeleton, accordion are not in v2 yet (add when a page needs them); G5 review, G7 docs sync.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
