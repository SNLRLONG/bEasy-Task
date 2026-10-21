# Frontend user permissions: backend contract answers

This document answers the questions in `backend-request.md` against the current
`bEasy-api` implementation on `feature/bunlong-user-permissions`.

## Blocking the editor

### 1. Role values are codes

All of these fields use the role `code`, such as `CS`, `FINANCE`, or `ADMIN`:

- `roles: string[]` returned by user permission details
- `roles` sent to `PUT /api/admin/rbac/users/:userId/roles`
- `roleCode` sent to `GET /api/admin/rbac/users`

Do not send the display `name`. The update endpoint uppercases and
deduplicates codes, then verifies that every code exists in the roles table.

### 2. Roles do not have active/inactive status

The current role model contains `id`, `name`, and `code`. It does not contain a
`status` column. `GET /api/admin/rbac/roles` returns all staff roles as:

```json
[
  { "id": 1, "name": "Customer Service", "code": "CS" }
]
```

Every role returned by this endpoint is assignable. The frontend should remove
`status` from its `RbacRoleItem` type and should not filter roles by status.

### 3. The logged-in user's full-access flag

Both authentication responses now include `userInfo.fullAccess`:

- `POST /api/admin/auth/account/login`
- `GET /api/admin/profile/info`

Example:

```json
{
  "userInfo": {
    "id": 1,
    "username": "admin",
    "type": "ADMIN",
    "role": null,
    "fullAccess": true
  },
  "permissions": []
}
```

The login response also contains `token`. The portal API interceptor unwraps
the normal response envelope, so the frontend reads `response.userInfo.fullAccess`.

Full-access role codes are `ADMIN`, `SUPER_ADMIN`, and `SUPER-ADMIN`.

### 4. VIEW is not coupled to other actions

The backend treats every `(module, action)` pair independently. These are both
valid:

- `DENY VIEW` while inherited `UPDATE` remains effective.
- `GRANT UPDATE` without `VIEW`.

The backend does not reject or automatically rewrite either case. The user
permission editor should not copy the role editor's automatic VIEW coupling.
It should show exactly the inherited, granted, denied, and effective pairs
returned by the API.

### 5. Valid staff account types

The `type` filter accepts these values:

```text
ADMIN
CUSTOMER_SERVICE
FINANCE
SALES
TECH
OPERATION
MARKETING
```

Account type and role code are separate concepts. Do not build the type filter
from `GET /rbac/roles`. Use the list above, or omit the type filter and use the
dynamic `roleCode` filter populated by `GET /rbac/roles`.

## Runtime behavior

### 6. When edited access takes effect

The API clears the target user's permission cache after role or override
updates. Server-side checks therefore use the new access on the target user's
next request. Tokens do not contain the permission list and are not invalidated.

The admin portal keeps its navigation permissions in the browser auth store.
The edited user's visible navigation updates after the portal calls
`GET /api/admin/profile/info`, reloads, or signs in again.

Recommended frontend message:

> Access is updated immediately. The user may need to reload the portal to see
> updated navigation.

### 7. Full-access users in responses

For a full-access user whose `users.role` is `null`:

- The user summary's `roles` is `[]` because it represents the stored role
  array.
- The permission detail response's top-level `roles` contains the fallback
  account type, for example `["ADMIN"]`.
- `fullAccess` is `true`.
- `fromRoles` contains stored mappings for the resolved role.
- `effective` contains those same stored mappings.

`effective` is not an exhaustive list for a full-access account because route
checks bypass the permission list. Show the matrix as informational and
read-only when `fullAccess` is true.

### 8. Activity Log coverage

Role and override mutations create the existing admin request activity entry
with the actor, URL, method, response status, and request ID. The current audit
sanitizer does not store the `roles` or `overrides` request fields, and database
before/after change triggers are currently configured for orders rather than
users or `user_permissions`.

Therefore the Activity Log identifies who called the update endpoint and
whether it succeeded, but it does not currently provide role/override
before-and-after values. Detailed permission history would require a separate
audited change implementation and is outside the portal editor contract.

### 9. Self-edit and the last full-access account

Both role and override endpoints reject self-edit with HTTP `403`:

```text
You cannot change your own roles
You cannot change your own permissions
```

Because a full-access administrator cannot change their own roles, the only
remaining full-access account cannot remove its own access through these
endpoints. A full-access administrator can change another full-access account;
the acting administrator still remains full access.

The API also rejects individual overrides for any full-access target with HTTP
`403`:

```text
Full-access accounts cannot have individual permission overrides
```

### 10. Pagination and sorting

`GET /api/admin/rbac/users` defaults to:

```text
page = 1
limit = 15
sort = createdAt DESC
```

`limit` is constrained to 1 through 100. There are currently no public sort
parameters. The frontend should not send `sortBy` or `sortOrder`.

## Frontend implementation summary

1. Use role codes everywhere.
2. Remove role `status` assumptions.
3. Add `fullAccess: boolean` to the portal's authenticated user type.
4. Read `userInfo.fullAccess` from login and profile responses.
5. Keep user override actions independent; do not force VIEW.
6. Use `fullAccess`, self-edit, and `RBAC:UPDATE` to determine read-only state.
7. Send the complete override array because the update endpoint is a full
   replacement.
8. After a successful edit, invalidate the selected user detail and user list
   queries.
