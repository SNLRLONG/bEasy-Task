# RBAC users: route, tabs and shared section builder

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T2 · new route; small refactor of the role page
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-rbac-users-shell/` (create when claimed) · **Branch:** `feat/rbac-users-shell`
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** none
- **Touches:** `app/routes.ts`, `app/routes/rbac/rbac.tsx` (extract only), `app/routes/rbac/users.tsx` (new), `app/routes/rbac/_components/rbac-tabs.tsx` (new), `app/hooks/use-rbac-sections.ts` (new). Check overlap with Portal-marketing-rbac.

## Goal
Make room for the Users view without changing how the Roles page behaves.

## Scope
- Move the inline `sections` `useMemo` of `rbac.tsx` into `use-rbac-sections.ts`; `rbac.tsx` calls it. No other change to the Roles page.
- New route `rbac/users` with the same `handle` (`RBAC`, `VIEW`) and an empty placeholder body.
- `rbac-tabs.tsx`: Roles | Users switch as links, shown at the top of both pages. Sidebar stays one entry.
- Do not touch `PermissionsMatrix`, `_utils.ts` or `permission.ts`.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
(G6 security review: skipped unless the diff touches auth or input handling; write `Skipped G6: reason`.)

## Done when
- [ ] Roles page behaves exactly as before: select role, edit, save, ADMIN guard toast, add module (compare before/after in the browser)
- [ ] Users tab visible only with `RBAC:VIEW`; direct URL without it is blocked by the protected layout
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
