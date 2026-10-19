# RBAC users: endpoints, keys, types and hooks

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T2 · new API calls, no screen yet; hooks are unused until the UI tasks land (permission write path is verified in the edit task)
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-rbac-users-data/` (create when claimed) · **Branch:** `feat/rbac-users-data`
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** none (backend says endpoints are live; verify on dev)
- **Touches:** `app/api/endpoint.ts`, `app/constants/query-key-enum.ts`, 4 new files in `app/hooks/query|mutations`

## Goal
Add everything the Users view needs to talk to the backend, with no UI.

## Scope
- `RBAC_USERS`, `RBAC_USER_PERMISSIONS`, `RBAC_USER_ROLES` endpoints and the two query keys.
- `use-rbac-users-query.ts` (params: searchText, type, roleCode, page, limit; `placeholderData: keepPreviousData`).
- `use-rbac-user-permissions-query.ts` (enabled only with a user id) exporting `PermissionPair`, `PermissionOverride`, `RbacUser`, `RbacUserPermissions`.
- `use-update-user-permissions-mutation.ts`, `use-update-user-roles-mutation.ts`: toast on success, invalidate the detail and list keys. Errors are left to the Axios interceptor.
- Roles param type follows backend answer 1 (assume role `code`).
- Do not edit `api.ts`, the auth store or `permission.ts`.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
(G6 security review: skipped unless the diff touches auth or input handling; write `Skipped G6: reason`.)

## Done when
- [ ] Typecheck and lint pass
- [ ] Each hook called once against the dev backend (browser console or a throwaway test) and the returned shape matches the guide's examples; differences recorded in `Docs/rbac/`
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
