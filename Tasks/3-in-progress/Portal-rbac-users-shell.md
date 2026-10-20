# RBAC users: route, tabs and shared section builder

- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T2 · new route; small refactor of the role page
- **Claimed by:** agent (claude), 2026-10-19
- **Started / updated:** 2026-10-19 / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-rbac-users-shell/`  · **Branch:** `feat/rbac-users-shell`
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** none
- **Touches:** `app/routes.ts`, `app/routes/rbac/rbac.tsx` (extract only), `app/routes/rbac/users.tsx` (new), `app/routes/rbac/_components/rbac-tabs.tsx` (new), `app/hooks/use-rbac-sections.ts` (new), `app/components/headers/header-title.tsx` (one line), `app/hooks/use-can-go-back.ts` (one line: show header on /rbac/users). Check overlap with Portal-marketing-rbac.

## Goal
Make room for the Users view without changing how the Roles page behaves.

## Scope
- Move the inline `sections` `useMemo` of `rbac.tsx` into `use-rbac-sections.ts`; `rbac.tsx` calls it. No other change to the Roles page.
- New route `rbac/users` with the same `handle` (`RBAC`, `VIEW`) and an empty placeholder body.
- `rbac-tabs.tsx`: Roles | Users switch as links, shown at the top of both pages. Sidebar stays one entry.
- Do not touch `PermissionsMatrix`, `_utils.ts` or `permission.ts`.

## Gates (T2)
- [x] G1 scope  - [x] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [x] G5 code review  - [x] G7 docs sync  - [x] G8 PR message  - [ ] G11 changelog
(G6 security review: skipped unless the diff touches auth or input handling; write `Skipped G6: reason`.)

## Done when
- [x] Roles page behaves exactly as before: select role, edit, save, ADMIN guard toast, add module (compare before/after in the browser)
- [x] Users tab visible only with `RBAC:VIEW`; direct URL without it is blocked by the protected layout
- [ ] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge

## Notes (agent)
- G2: eslint clean on touched files, prettier applied, `yarn typecheck` has no errors in rbac files (pre-existing unrelated errors in service-bundle, chat-store, vite.config.ts).
- G4 not run: needs login and backend (no credentials). Not verified in browser.
- Tabs bar is h-11; Roles page keeps h-[calc(100vh-88px)] outer, inner content is flex-1 min-h-0. Prettier re-indented rbac.tsx body (use `git diff -w`).

Skipped G6: owner will review security themselves first (decided 2026-10-19); the diff touches no auth or input handling.

- G5: read `git diff -w`; section extraction is line-for-line identical (same logic, `grouped` as memo dep instead of `permissionsData`, same output). Wrapper divs balanced; Roles keeps outer h-[calc(100vh-88px)], tabs h-11 shrink-0, content flex-1 min-h-0. Both routes have `handle` RBAC/VIEW (protected layout blocks direct URL); `RbacTabs` returns null without RBAC:VIEW. eslint and prettier re-run clean on all 5 touched files. No code fixes needed.
  - Finding (outside Touches, not fixed): `app/components/headers/header-title.tsx:80` has no `'/rbac/users'` entry, so the header title for Users will fall back. Add `'/rbac/users': 'Users'` (or in the users-data task). Sidebar highlight works (`startsWith(url + '/')` in nav-dropdown.tsx).
  - Minor: `use-rbac-sections.ts` (hooks) imports `@/routes/rbac/_utils`; acceptable, no cycle.
  - Overlap: Portal-marketing-rbac is unclaimed, no worktree (`git worktree list` shows none); it touches `app/routes/rbac/*` data and new module keys, no file clash yet. Users page body is owned by Portal-rbac-users-data.
- G7: no API/architecture doc change (no new call, param or header); route and tab plan already in `Docs/rbac/Portal_User_Permissions_Review.md`. Nothing to update.
- G8 PR draft:
  - Title: `feat(rbac): add Users route, Roles|Users tabs and shared sections hook`
  - Why: make room for the staff Users view without growing `rbac.tsx` or changing Roles behaviour.
  - What changed: move the `sections` useMemo into `useRbacSections` (`app/hooks/use-rbac-sections.ts`); new route `rbac/users` (placeholder body, same `handle` RBAC/VIEW); `RbacTabs` Roles|Users links on both pages, hidden without RBAC:VIEW; `rbac.tsx` wrapped in a flex container under the tabs.
  - How tested: eslint and prettier clean on touched files; `yarn typecheck` no errors in rbac files (unrelated pre-existing errors elsewhere). Not run in browser (G4, no credentials); no automated tests exist for this area.
  - Risk / rollback: T2; Roles layout wrapper changed (height now shared with tabs), verify scroll and sticky strip in browser. Revert the branch to roll back. Skipped G6 per above.
  - Task: Tasks/3-in-progress/Portal-rbac-users-shell.md
