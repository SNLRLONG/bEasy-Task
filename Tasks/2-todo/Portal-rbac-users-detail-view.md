# RBAC users: user detail with read-only tri-state matrix

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T2 · new UI over a new API; read-only, no save
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-rbac-users-detail/` (create when claimed) · **Branch:** `feat/rbac-users-detail`
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
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
(G6 security review: skipped unless the diff touches auth or input handling; write `Skipped G6: reason`.)

## Done when
- [ ] All four cell states render for a user that has each case (use dev data; list the user ids used in the PR)
- [ ] Module labels and section order match the Roles page
- [ ] A new module added by the catalogue appears without code change
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
