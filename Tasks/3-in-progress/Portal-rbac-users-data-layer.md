# RBAC users: endpoints, keys, types and hooks

- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T2 · new API calls, no screen yet; hooks are unused until the UI tasks land (permission write path is verified in the edit task)
- **Claimed by:** agent (claude), 2026-10-19
- **Started / updated:** 2026-10-19 / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-rbac-users-data/`  · **Branch:** `feat/rbac-users-data`
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
- [x] G1 scope  - [x] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [x] G5 code review  - [x] G7 docs sync  - [x] G8 PR message  - [ ] G11 changelog
G2 (2026-10-19): eslint and prettier clean on the 6 touched files. `yarn typecheck` reports 100+ errors in untouched files (calendar, location, service-bundle, chat-store, vite.config.ts, ...), none in the touched files; same baseline as main, not fixed here.
G3: not run. No hook-level test setup exists in the repo (only `node --test` scripts in `tests/`); no new logic beyond request wiring.
G5 (2026-10-19): types, endpoints, keys, invalidation, payloads and `roles: string[]` checked against Backend_User_Permissions_Guide.md; match. bEasy-api not cloned, so no path:line cites. One fix: list hook now drops empty `searchText`/`type`/`roleCode` (trim, `|| undefined`) and keys the cache on the cleaned params. No shared files edited. Note: list `data` is nested (`{data, pagination}`) as the guide says; the screen task must read `res.data`.
G7 (2026-10-19): no mismatch with the guide; nothing changed in Docs/rbac/. Open backend question 1 (role `code` vs name) and 5 (type values) still in backend-request.md and still apply to these hooks.
G8 (draft PR message):
```
feat(rbac): add user roles/permissions endpoints, keys and hooks

Why: the Users view needs a data layer before any UI lands.
What changed:
- API_ENDPOINT: RBAC_USERS, RBAC_USER_PERMISSIONS, RBAC_USER_ROLES; query keys RBAC_USERS, RBAC_USER_PERMISSIONS
- useRbacUsersQuery (keepPreviousData, empty filters dropped), useRbacUserPermissionsQuery (enabled with user id; exports PermissionPair/PermissionOverride/RbacUser/RbacUserPermissions)
- useUpdateUserPermissionsMutation, useUpdateUserRolesMutation (toast, invalidate detail + list)
How tested: eslint + prettier clean on touched files; typecheck has no errors in touched files (repo baseline has 100+ elsewhere). Not run: unit tests (none apply), calls against dev backend (no credentials). Roles are assumed to be role codes (backend answer pending).
Risk / rollback: T2, hooks unused until UI tasks land; revert the commit.
Task: Tasks/3-in-progress/Portal-rbac-users-data-layer.md
```
G4: not run: no credentials (no dev backend login available to the agent; `.env.example` only).
(G6 security review: skipped unless the diff touches auth or input handling; write `Skipped G6: reason`.)

## Done when
- [x] Typecheck and lint pass
- [ ] Each hook called once against the dev backend (browser console or a throwaway test) and the returned shape matches the guide's examples; differences recorded in `Docs/rbac/`
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

Skipped G6: owner will review security themselves first (decided 2026-10-19); the diff touches no auth or input handling.
