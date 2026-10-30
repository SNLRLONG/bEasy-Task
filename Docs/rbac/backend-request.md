# Backend questions: RBAC

**To:** Backend · **Updated:** 2026-10-28

## 1. New permission module `USERS` for staff account actions

The portal now gates create, edit, deactivate/activate, reset password and delete of staff accounts on a **new module `USERS`**, not on `RBAC`. Contract: [Staff_Account_CRUD_Frontend_Guide.md](Staff_Account_CRUD_Frontend_Guide.md) (banner at the top). Please:

1. **Create the module** `USERS` with actions `VIEW`, `ADD`, `UPDATE`, `DELETE` in the permission catalogue, so it appears in `GET /rbac/permissions` and in the role matrix (label "Users"; it covers admin-portal staff accounts only, not customers). `USERS:VIEW` opens the new **Users** page (Setup > Users, `/users`), a data table of staff accounts with its own sidebar item and route, following `bEasy-portal/docs/new-page-guide.md`. It is separate from **Roles & Permissions** (`/rbac`, `/rbac/users`), which keeps editing roles and per-user overrides on `RBAC`. The Users table links each row to that permission view.
2. **Gate the endpoints** on `USERS` instead of `RBAC`:

| Endpoint | Was | Now |
|---|---|---|
| `POST rbac/users` | `RBAC:ADD` | `USERS:ADD` |
| `PUT rbac/users/:userId` | `RBAC:UPDATE` | `USERS:UPDATE` |
| `PATCH rbac/users/:userId/status` | `RBAC:UPDATE` | `USERS:UPDATE` |
| `POST rbac/users/:userId/reset-password` | `RBAC:UPDATE` | `USERS:UPDATE` |
| `DELETE rbac/users/:userId` | `RBAC:DELETE` | `USERS:DELETE` |

   Also accept `USERS:VIEW` (in addition to `RBAC:VIEW`) on the two reads the Users page makes, otherwise a user with only `USERS:VIEW` sees the page but gets `403`: `GET rbac/users` (the table) and `GET rbac/roles` (role picker in the Add form).
   Unchanged on `RBAC`: `GET rbac/users/:userId/permissions`, `GET rbac/permissions`, `PUT rbac/users/:userId/roles`, `PUT rbac/users/:userId/permissions` and all role endpoints (the permission editor stays on `RBAC`).
3. **Existing grants:** should roles that have `RBAC:ADD / UPDATE / DELETE` get the matching `USERS` actions by migration, or start empty? Our suggestion: copy them once, so nobody loses access on release.
4. **Full-access accounts** keep bypassing the check as today. Their login `permissions` is `[]` in the example: does it ever list modules? The portal hides buttons from the stored list, so we need to know whether to treat `fullAccess` as allowed in the UI.

Until this is live on dev, the new buttons stay hidden for everyone without `USERS` in their permissions (G4 blocker for `Portal-rbac-staff-crud`).

User roles and overrides answers: [Frontend_User_Permissions_Backend_Review.md](Frontend_User_Permissions_Backend_Review.md).
