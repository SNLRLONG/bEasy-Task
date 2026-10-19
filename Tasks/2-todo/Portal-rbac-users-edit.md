# RBAC users: edit roles and permission overrides

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T3 · changes who can access what (permissions write path)
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-rbac-users-edit/` (create when claimed) · **Branch:** `feat/rbac-users-edit`
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** Portal-rbac-users-list, Portal-rbac-users-detail-view; backend-request answers 1, 2, 3, 4 (blocking). 6 to 9 only change copy
- **Touches:** `app/routes/rbac/_components/users/*`, `app/routes/rbac/users.tsx`

## Goal
Let an authorised admin change a user's roles and overrides safely.

## Scope
- Role selector (options from `GET /rbac/roles`, active only if answer 2 says so); own Save; confirm dialog before `roles: []` explaining the `user.type` fallback.
- Override editing: each cell cycles Inherit / Grant / Deny; draft starts from `overrides`; Save sends the **complete** array; Clear sends `{ overrides: [] }`; own Save and Cancel.
- Read-only when: no `RBAC:UPDATE`, the target is the logged-in user, or the target has `fullAccess`.
- Draft rules: kept on `400`/`403`; kept and rebased after a roles save; switching user or leaving the tab with unsaved changes asks to discard; no optimistic updates; Save disabled while pending.
- `404` clears the selection and refreshes the list.
- After save, one-line note about when the change reaches the user (per backend answer 6).
- VIEW coupling per backend answer 4.

## Gates (T3)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Tested with two logins: one with `RBAC:UPDATE`, one with only `RBAC:VIEW` (no save controls)
- [ ] Self-edit and full-access target are read-only
- [ ] Verified on dev: grant adds access, deny removes inherited access, clearing overrides restores inheritance, the edited user sees the new access after re-login
- [ ] Release plan (G10): rollback is hiding the Users tab behind the route; no backend deploy needed
- [ ] `403` and `400` paths keep the draft
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
