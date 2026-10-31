# RDS 1: responsive app shell, header, sidebar, page container

- **ID:** T-0055
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-27 / 2026-10-27
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-shell` off it
- **Tier:** T2 · shared layout used by all pages
- **Claimed by:** claude session, 2026-10-22
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS foundation
- **Touches:** `app/components/v2/layout/`, `app/routes/dashboard-layout.tsx`, `app/components/sidebar/*` (tuning only, via v2 wrappers where possible), `app/components/headers/*` (read-only; v2 copy)

## Goal
v2 shell using `100dvh` flex (no `calc(100vh-88px)`), responsive page header (title + wrapping actions, back button), page container with `padding/screen` tokens, sidebar as sheet below `lg`. Swap only `routes/dashboard-layout.tsx` to v2; `ContentWrapper` stays for unmigrated pages.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [x] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-22, uncommitted in worktree)
Built in `app/components/v2/layout/`: `AppShell` (h-dvh flex; persistent 240px sidebar from `lg`, left sheet below), `SidebarNav` (same `nav-config.ts`, same permission and chat-badge logic, no dependency on `ui/sidebar`), `AppHeader` (56px below md, 88px from md so un-migrated pages that use `calc(100vh-88px)` still fit; avatar-only below md), `PageContainer`, `ShellProvider`. `routes/dashboard-layout.tsx` now renders `<AppShell />`; no old component was edited. Dev preview at `/dev/shell`.
Checked in Chrome at 360/428/768/1024/1280/1536: no horizontal overflow, sidebar visible only from 1024, hamburger below 1024, drawer opens full height at 288px, header 56/56/88/88/88/88. Logged-in smoke test (read-only, dev backend, 2026-10-22): `/`, `/customer`, `/coupon`, `/finance-orders` inside the shell at 360/768/1024/1280 have no horizontal overflow, correct header height and hamburger, drawer shows the real permission-filtered nav with the chat badge and closes on navigation. `/order` uses its own layout and is not affected. On phones the old tables are clipped inside their own scroll area (expected until the data-table task). Not tested: calendar, chat, pages with their own header.
Known gaps to decide: (1) pages with their own `CustomHeader` (header hidden by `useCanGoBack`) still have no hamburger on mobile, same as before; (2) the profile/logout/password dialogs inside `HeaderRight` are still the old components; a v2 user menu is a follow-up; (3) sidebar width follows Suntel (240px) instead of the old 300px, and the sidebar no longer collapses (it never could on desktop). G5 review, G7 docs sync and a logged-in smoke test of 3 old pages inside the shell are still to do.

## Correction (2026-10-27): the sidebar must not change
The first version of this task replaced the sidebar with a new v2 one (240px, new row styling). That was wrong: the sidebar UI style is not part of this work. `routes/dashboard-layout.tsx` now renders the ORIGINAL `AppSidebar` inside the original `SidebarProvider` (300px, same colours and drawer), and only the header and the scrolling content area are v2. All sidebar files (`components/sidebar/*`, `ui/sidebar.tsx`, `common/app-sidebar.tsx`, `lib/sidebar-tokens.ts`, `hooks/use-mobile.ts`) are byte-identical to `origin/main`. The header menu button uses `min-[769px]:hidden` so it matches the sidebar's own `useIsMobile` (< 769px) switch; the original header had no menu button at exactly 768px. The v2 `AppShell`/`SidebarNav` stay in `components/v2/layout` unused (nothing is deleted).

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
