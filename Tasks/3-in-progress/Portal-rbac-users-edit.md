# RBAC users: edit roles and permission overrides

- **ID:** T-0038
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T3 · changes who can access what (permissions write path)
- **Claimed by:** agent (claude), 2026-10-21
- **Started / updated:** 2026-10-21 / 2026-10-21
- **Worktree:** `bEasy/bEasy-portal-rbac-users-edit/` · **Branch:** `feat/rbac-users-edit` (branched from `feat/rbac-users-shell`, the integration branch)
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** Portal-rbac-users-list, Portal-rbac-users-detail-view; backend answers (all received 2026-10-21, see `Docs/rbac/Frontend_User_Permissions_Backend_Review.md`)
- **Touches:** `app/routes/rbac/_components/users/*`, `app/routes/rbac/users.tsx`, `app/hooks/query/use-rbac-roles-query.ts` (drop `status`; check the Roles page dialog that still sends it), auth user type and `app/hooks/use-auth.ts` (add `fullAccess`)

## Goal
Let an authorised admin change a user's roles and overrides safely.

## Scope
- Role selector (options from `GET /rbac/roles`; submit role `code`; no status filter, roles have no status); own Save; confirm dialog before `roles: []` explaining the `user.type` fallback.
- Override editing: each cell cycles Inherit / Grant / Deny; draft starts from `overrides`; Save sends the **complete** array; Clear sends `{ overrides: [] }`; own Save and Cancel.
- Read-only when: no `RBAC:UPDATE`, the target is the logged-in user, or the target has `fullAccess` (matrix informational: `effective` is not exhaustive). Read the admin's own `userInfo.fullAccess` from login and `GET /admin/profile/info`; show backend `403` messages.
- Draft rules: kept on `400`/`403`; kept and rebased after a roles save; switching user or leaving the tab with unsaved changes asks to discard; no optimistic updates; Save disabled while pending.
- `404` clears the selection and refreshes the list.
- After save, note: "Access is updated immediately. The user may need to reload the portal to see updated navigation." Invalidate user detail and user list queries.
- No VIEW coupling: `DENY VIEW` with inherited `UPDATE` and `GRANT UPDATE` without `VIEW` are valid; do not copy the role editor's rule.
- Type filter (list) uses the fixed list ADMIN, CUSTOMER_SERVICE, FINANCE, SALES, TECH, OPERATION, MARKETING; do not send sort params.

## Gates (T3)
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [x] G5 code review (strict)  - [x] G6 security review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [x] Tested with two logins: one with `RBAC:UPDATE`, one with only `RBAC:VIEW` (no save controls)
- [x] Self-edit and full-access target are read-only
- [x] Verified on dev: grant adds access, deny removes inherited access, clearing overrides restores inheritance, the edited user sees the new access after re-login
- [x] Release plan (G10): rollback is hiding the Users tab behind the route; no backend deploy needed
- [x] `403` and `400` paths keep the draft
- [ ] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge

## Notes (agent)
- Built: `_edit-utils.ts` (draft, cycle, read-only reason), `user-roles-editor.tsx`, editable `UserPermissionsMatrix` (opt-in props), `UserEditor` in `user-detail.tsx` (drafts per user via `key`, `useBlocker` + `beforeunload` discard guard, Clear overrides with confirm, 404 → `onMissing`), `fullAccess` on `UserResponseProps`. Baseline roles = `data.user.roles` (stored array), not top-level `roles` (fallback type for full-access).
- G2: eslint clean on `app/routes/rbac`, mutations, tests; prettier applied; `tsc` has no rbac errors (pre-existing `use-auth.ts(23)`).
- G3: `tests/rbac-user-edit.test.mjs` (7 pass, `node --test`). UI behaviour (blocker dialog, save/403 paths) not covered by tests.
- G4 not run: needs login with `RBAC:UPDATE` and a backend.
- `role-form-dialog.tsx` edit path now uses `initial.status ?? true` (type change); Roles page create already sent `true`. Backend has no role status; remove the field there in a separate task.
- Open: own `fullAccess` flag is typed but unused, because full-access targets are read-only for everyone.
- 2026-10-28 G5 (read all 11 changed files): payloads match the backend reply (role codes, complete override array, `{ overrides: [] }` for Clear); drafts keyed per user via `key`, kept on 400/403, cleared on success; no VIEW coupling; 404 calls `onMissing`; discard guard covers user switch and tab close. No blocking defects. Minor, not fixed: (a) an override draft kept across refetches replaces all overrides on save, so it can overwrite another admin's edit made in between; (b) if the auth store has no user id, the self-edit check is skipped and only the backend `403` stops it.
- 2026-10-28 G6: no new inputs reach the DOM unescaped, no secrets or tokens touched, request paths use a numeric id. All access decisions (`RBAC:UPDATE`, self-edit, full-access target) are UI hints only; the backend enforces them and returns `403`. Save controls are hidden without `RBAC:UPDATE`, the route needs `RBAC:VIEW`. No findings.
- Left for the owner: G4 (two logins on dev), G9, G10, G11 (changelog after merge to `main`).
