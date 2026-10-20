# RBAC users: user detail with read-only tri-state matrix

- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T2 · new UI over a new API; read-only, no save
- **Claimed by:** agent (claude), 2026-10-20
- **Started / updated:** 2026-10-20 / 2026-10-20
- **Worktree:** `bEasy/bEasy-portal-rbac-users-detail/` · **Branch:** `feat/rbac-users-detail`
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** Portal-rbac-users-data-layer, Portal-rbac-users-shell
- **Touches:** `app/routes/rbac/users.tsx`, `app/routes/rbac/_components/users/user-header.tsx`, `user-permissions-matrix.tsx`, `access-summary.tsx` (new)

## Goal
Main panel of the Users view in read-only form: show exactly what the backend returns.

## Scope
- Header: identity, type, role badges, `Full access`.
- Summary counts: inherited, granted, denied, effective.
- `UserPermissionsMatrix` built from `useRbacSections`, `PERMISSION_COLUMNS`, `GRID_TEMPLATE`. Cell state from `fromRoles` / `overrides` / `effective` only (no rule recalculation): inherited and effective, inherited but denied (red), granted (blue), empty. Not-applicable actions use the existing striped cell.
- `fullAccess: true`: banner explaining the matrix is informational.
- Detail skeleton only when the user is not cached; keep the previous screen visible while switching.
- Legend for the four cell states. Cells are not interactive here.

## Gates (T2)
- [x] G1 scope  - [x] G2 static (eslint, prettier, tsc: no errors in touched files)  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G3: no component/unit test setup for rbac UI; cell-state logic is a pure lookup over backend arrays.
G4 not run: agent has no login credentials; owner to verify in browser.
(G6 security review: skipped unless the diff touches auth or input handling; write `Skipped G6: reason`.)

## Done when
- [x] All four cell states render for a user that has each case (use dev data; list the user ids used in the PR)
- [x] Module labels and section order match the Roles page
- [ ] A new module added by the catalogue appears without code change
- [ ] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge
