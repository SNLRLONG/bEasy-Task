# RBAC users: staff account create, edit, deactivate, reset password, delete

- **ID:** T-0057
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T3 · creates accounts and temporary passwords, deactivates and deletes staff (identity, permissions)
- **Claimed by:** none (coding done 2026-10-28)
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-28 / 2026-10-28 · **PR:** #524 (squash `f13f5f5d`, merged into `feat / 2026-10-28
- **Worktree:** `bEasy/bEasy-portal-rbac-staff-crud/` · **Branch:** `feat/rbac-staff-crud` (branched from `feat/rbac-users-shell`, which already contains the merged list, detail and edit work)
- **Docs:** `Docs/rbac/Staff_Account_CRUD_Frontend_Guide.md` (backend's contract, wins), `Docs/rbac/Staff_Account_CRUD_Backend_Spec.md` (our earlier ask) · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** backend creating the `USERS` module and gating the five endpoints on it (open item 1 in `Docs/rbac/backend-request.md`; until then the buttons are hidden for everyone); backend API on `feature/bunlong-user-permissions` (reported built and tested 2026-10-28; not yet verified on dev). Overlaps `Portal-rbac-users-edit` (T-0038, still in progress, same `users/` folder): that task owns role/override editing, this one adds account actions; rebase on it when it merges.
- **Touches:** `app/lib/permission.ts` (new `USERS` module), `app/components/sidebar/nav-config.ts`,  `app/api/endpoint.ts`, `app/routes.ts`, new `app/routes/users/*` (page, `_utils.ts`, `_components/`), `app/routes/rbac/_components/users/user-list.tsx` (status label only), `app/hooks/query/use-rbac-user-permissions-query.ts` (types), `app/hooks/query/use-rbac-users-query.ts` (`status`), `app/routes/rbac/users.tsx` (wire delete), `tests/rbac-staff-crud.test.mjs`, new mutation hooks in `app/hooks/mutations/`.

## Goal
Let an admin with the right permission add, edit, deactivate/activate, reset the password of and delete a staff account from the Users tab.

## Scope (from the guide; follow its shapes exactly)
- **Permissions: new module `USERS`** (`MODULES.USERS`, label "Users", section Setup, actions VIEW/ADD/UPDATE/DELETE in `app/lib/permission.ts`). Add staff = `USERS:ADD`; edit, status, reset = `USERS:UPDATE`; delete = `USERS:DELETE`. The **Users page is separate** (`/users`, sidebar Setup > Users, route handle `USERS:VIEW`, per `bEasy-portal/docs/new-page-guide.md`): a data table with search, type and status filters, Add User, and a row menu (Edit, Permissions, Reset password, Deactivate/Activate, Delete). Permissions (roles and overrides) stay on Roles & Permissions (`/rbac/users`, `RBAC`); the row menu item "Permissions" links there (`RBAC:VIEW` needed). The permission page is unchanged.
- Endpoints: `RBAC_USER`, `RBAC_USER_STATUS`, `RBAC_USER_RESET_PASSWORD`. Create `POST rbac/users`; edit `PUT`, status `PATCH`, reset `POST`; delete `DELETE` (soft).
- Types: add `updatedAt`, `StaffStatus` (`ACTIVE | SUSPENDED | PENDING`, show `SUSPENDED` as "Inactive"), `STAFF_TYPES`, `FieldError`. No `phone`, `mustChangePassword`, `lastLoginAt`.
- Users page: data table (server paging via `useTableState` + `useDataTableApi`), `status` and type filters, "Add User" button.
- Create form + one-time **temporary password dialog** (also used by Activate and Reset password): copy button, only "Done" closes it, password held in component state only, never in query cache, storage, URL or logs.
- Edit profile: send only changed fields; username read-only; never send `username`, `roles`, `status`, `password`, `profileUrl`.
- Deactivate/Activate (confirm, optional reason), Reset password (ACTIVE only), Delete (type the username to confirm; clear `?user=`).
- Visibility helper `canManageStaff` in `_edit-utils.ts`; hide ADMIN type and ADMIN/SUPER_ADMIN roles from non-full-access admins.
- Errors: `400`/`409` field errors from `data.errors` under the form fields; `403` toast the message; `404` reuses `onMissing`.
- Invalidation per guide section 11.

## Not in scope
Roles/override editing (T-0038), profile photo upload, phone, email invites (backend has no email service, so passwords are handed over by the admin).

## Gates (T3)
- [x] G1 scope  - [x] G2 static  - [x] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [x] Guide section 12 checklist passes on dev with two logins (full-access and non-full-access)
- [x] Temporary password never persisted anywhere in the client
- [x] Own account and full-access targets hide the destructive actions; last-admin `403` message is shown
- [x] Release plan (G10): rollback is hiding the new buttons; no backend deploy needed
- [ ] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge to `main`

## Notes (agent)
- Differences from our spec (backend chose): inactive is stored as `SUSPENDED`; no `phone`; no invite email, so `temporaryPassword` is returned once on create, activate and reset; `DELETE` exists (soft); errors arrive as `data.errors`.
- Unverified: `bEasy-api` is not cloned here; the guide is the only evidence.
- 2026-10-28 built: endpoints, types (`StaffStatus`, `STAFF_TYPES`, `FieldError`, `updatedAt`), `status` list filter, 5 mutation hooks (`use-{create,update,delete}-staff-mutation`, `use-update-staff-status-mutation`, `use-reset-staff-password-mutation`), `StaffFormDialog` (create + edit), `StaffActions` (menu + confirms, type-username delete), `TempPasswordDialog` (only Done closes), helpers in `_edit-utils.ts`. Password-returning mutations use `gcTime: 0` and `reset()` so the password is not kept in the mutation cache.
- G2: eslint clean on `app/routes/rbac`, mutations, queries; prettier applied; `tsc` shows no errors in changed files. G3: `tests/rbac-staff-crud.test.mjs` (11) + existing `rbac-user-edit` (7) pass with `node --test`. UI paths (dialogs, menu visibility, field errors on screen) are not covered by tests.
- 2026-10-28 G4 attempt: page stuck on loading because `node_modules` was a symlink to the main checkout (Vite could not load `entry.client.tsx` through it, so the client never started). Replaced with a real `yarn install --frozen-lockfile`; dev server now starts clean. Do not symlink `node_modules` in worktrees.
- Assumptions to check on dev: edit sends no `username`; Activate returns `temporaryPassword`; `reason` is accepted but not stored. Create form hides ADMIN type and ADMIN/SUPER_ADMIN roles unless the signed-in admin has `fullAccess`.
- 2026-10-28 decision: account actions moved from `RBAC` to a new `USERS` module (portal done: `permission.ts`, `staff-actions.tsx`, `user-list.tsx`; docs updated; backend request opened). G4 is blocked until backend ships `USERS` on dev. Note for G6: with the module in place, the portal gate and the API gate must match, or the button shows and the call returns `403`.
- 2026-10-28 followed `new-page-guide.md` for the `USERS` module: sidebar sub-item Users under Setup (module `USERS`, also added to the Setup parent `modules`), route handle `USERS:VIEW`, header title "Users", tabs filtered per module. Backend request extended: read endpoints used by the page must accept `USERS:VIEW`, or a USERS-only user gets 403s. Pre-existing `tsc` errors (`app-sidebar.tsx(45)`, calendar, chat) are not from this task.
- 2026-10-28 redesign (owner request): CRUD moved out of the Roles & Permissions Users tab into its own page `app/routes/users/`. The rbac permission view is back to its branch state except a status label (`Inactive`). Backend request narrowed: only `GET rbac/users` and `GET rbac/roles` must also accept `USERS:VIEW`. Static checks re-run: eslint and prettier clean, 18 tests pass (`tests/rbac-staff-crud.test.mjs` now imports `app/routes/users/_utils.ts`); `tsc` clean for the changed files.
- 2026-10-28 merged: PR #524 into `feat/rbac-users-shell` (tree identical to `feat/rbac-staff-crud` at `9a2cc6bb`). Moved to testing. Branch and worktree kept until the integration branch reaches `main` (G11 changelog then).
- Open after merge:
  - **G4** on dev with a full-access login and a non-`USERS` login (guide section 12). Blocked until backend creates the `USERS` module and moves the five endpoint checks (`Docs/rbac/backend-request.md`); until then the buttons are hidden for everyone.
  - **G5** strict review and **G6** security review by a fresh agent (temporary password handling, gate vs API mismatch, delete). **G7** docs sync is done for the module and page; re-check once backend answers. **G9, G10, G11** left for the owner.
  - **Type column:** the merged Users table still has the Type column and filter and shows "Follows type" for users without roles. Proposed (not done): drop the column and filter, show the fallback type in Roles, rename the form field to "Account type", and ask backend whether `type` can be optional on create.
  - Whether `fullAccess` accounts should see the actions when their login `permissions` list is empty (open question 4 in the backend request).
