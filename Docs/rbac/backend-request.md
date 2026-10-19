# Backend questions: user roles and permissions

**To:** Backend · **Updated:** 19 Oct 2026 · **Context:** [Backend_User_Permissions_Guide.md](Backend_User_Permissions_Guide.md), review in [Portal_User_Permissions_Review.md](Portal_User_Permissions_Review.md)

The `bEasy-api` repo is not cloned in this workspace, so none of this was checked against code. Please answer in this file's order; we will move the answers into the review doc.

## Blocking the editor (needed before `Portal-rbac-users-edit`)

1. **Roles in `roles: string[]` and `PUT .../roles`: codes or names?** The portal's `GET /rbac/roles` returns `{ id, name, code, status }`. We assume `code` (e.g. `CS`). Also confirm `roleCode` in the list filter is the same value.
2. **Does `GET /rbac/roles` return inactive roles (`status: false`)?** If yes, may an inactive role be assigned? We plan to hide them from the selector.
3. **Is the logged-in admin's own `fullAccess` flag in login and `GET /auth/me`?** The guide says the profile only carries effective permissions. Without the flag the portal cannot know that a non-full-access admin may not edit a full-access user, so it shows the edit controls and relies on a `403`. Please add `fullAccess` to `userInfo`, or confirm `403` with a stable message is the contract.
4. **VIEW coupling.** The role editor forces `VIEW` on when any other action is on. For user overrides: is `DENY VIEW` with an inherited or granted `UPDATE` valid, and is `GRANT UPDATE` without `VIEW` valid? If the backend rejects or ignores these, say which (`400` with message?). Otherwise the portal will mirror the role editor's rule in the UI.
5. **Valid values for the `type` filter** (staff account types). We need the list to build the filter; or confirm it can come from `GET /rbac/roles`.

## Behaviour we need to set expectations on

6. **When does an edited user's access change?** The portal stores permissions in the browser at login and refreshes on page load. Do you invalidate the target user's token or session when roles or overrides change? If not, we show "Takes effect the next time this user signs in or reloads."
7. **Full-access user in the list:** is `roles` empty for them, and what does `fromRoles`/`effective` contain? (The guide says it is not the complete list.)
8. **Activity Log:** are role and override changes written to the activity log with the actor and the before/after? If not, do you want it?
9. **Self-edit and last full-access admin:** besides rejecting self-edit, do you block removing the last full-access account? Which status code and message?
10. **Pagination default sort** for `GET /rbac/users`, and whether sort params exist.
