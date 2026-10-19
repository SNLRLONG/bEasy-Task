# RBAC: user-level roles and permissions

Index for the "Users" view of the admin Roles & Permissions page (per-user roles and GRANT/DENY overrides). Role-level RBAC already exists in the portal.

| File | What it is | Read when |
|---|---|---|
| [Backend_User_Permissions_Guide.md](Backend_User_Permissions_Guide.md) | Integration guide as sent by backend (19 Oct 2026). Unedited apart from a banner | You need endpoint shapes, response examples, rules |
| [Portal_User_Permissions_Review.md](Portal_User_Permissions_Review.md) | Our analysis: what the guide gets right, what does not fit the portal, final structure, task split | Before picking up any `Portal-rbac-users-*` task |
| [backend-request.md](backend-request.md) | Open questions to backend (open items only) | Before you ask backend anything about RBAC |

Plan and tasks: `Tasks/_plans/Portal-rbac-users.md`, `Tasks/2-todo/Portal-rbac-users-*.md`.

When backend answers: delete the item from `backend-request.md`, and record the reply in the review doc under "Backend answers". One fact lives in one place.
