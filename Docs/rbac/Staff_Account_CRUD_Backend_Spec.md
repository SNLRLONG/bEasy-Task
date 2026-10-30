> **Answered 2026-10-28.** Backend's contract is [Staff_Account_CRUD_Frontend_Guide.md](Staff_Account_CRUD_Frontend_Guide.md) and wins. Differences: inactive is stored as `SUSPENDED`; no `phone`, `mustChangePassword` or `lastLoginAt`; no invite email, so a one-time `temporaryPassword` is returned on create, activate and reset; `DELETE` is a soft delete; field errors arrive in `data.errors`. Kept as the record of what we asked. Later change: permissions for #3 to #7 are now a new `USERS` module (see backend-request.md).

# Spec for backend: staff account CRUD

**To:** Backend · **From:** Portal · **Updated:** 2026-10-28 · **Status:** proposal, please confirm or correct
**Context:** the portal Users view (Roles & Permissions) can list staff and change roles and overrides (see [Backend_User_Permissions_Guide.md](Backend_User_Permissions_Guide.md) and [Frontend_User_Permissions_Backend_Review.md](Frontend_User_Permissions_Backend_Review.md)). It cannot create, edit, deactivate or remove a staff account. This file asks for those endpoints and fixes the response shape. The open questions behind it are in [backend-request.md](backend-request.md).

The `bEasy-api` repo is not cloned here, so nothing below was checked against code. **If an endpoint already exists, reply with its route and shape and we build against that.** Shapes are for backend to adjust; portal adapts.

## Conventions (same as the six existing RBAC endpoints)

- Base: `/api/admin/rbac/users`. Bearer token. Envelope `{ code, status, message, data }`; the portal reads `data`.
- Staff only. Customers are not touched by any of these calls.
- Roles are sent and returned as role `code` (`CS`, `FINANCE`, `ADMIN`), never `name`.
- Dates are ISO 8601 UTC.
- Every error carries a readable `message`; the portal shows it in a toast.

## Shared object: `StaffUser`

Same fields as the items in `GET rbac/users`, so one type serves list, create, edit and status change.

```json
{
  "id": 42,
  "username": "sophea.k",
  "firstName": "Sophea",
  "lastName": "Kim",
  "email": "sophea@beasy.info",
  "phone": "+85512345678",
  "profileUrl": null,
  "type": "FINANCE",
  "roles": ["FINANCE"],
  "status": "ACTIVE",
  "fullAccess": false,
  "mustChangePassword": true,
  "lastLoginAt": null,
  "createdAt": "2026-10-28T03:15:00.000Z",
  "updatedAt": "2026-10-28T03:15:00.000Z"
}
```

New compared with today: `phone`, `mustChangePassword`, `lastLoginAt`, `updatedAt` (nice to have: `lastLoginAt`, `updatedAt`). Please confirm the allowed `status` values; we assume `ACTIVE` and `INACTIVE`. `type` is one of ADMIN, CUSTOMER_SERVICE, FINANCE, SALES, TECH, OPERATION, MARKETING.

## Endpoints

| # | Method | Path | Purpose | Permission (proposed) | Exists? |
|---|---|---|---|---|---|
| 1 | `GET` | `/rbac/users` | Paginated list | `RBAC:VIEW` | yes |
| 2 | `GET` | `/rbac/users/:userId/permissions` | Detail with roles and overrides | `RBAC:VIEW` | yes |
| 3 | `POST` | `/rbac/users` | Create staff account | `USERS:ADD` | **needed** |
| 4 | `PUT` | `/rbac/users/:userId` | Edit profile fields | `USERS:UPDATE` | **needed** |
| 5 | `PATCH` | `/rbac/users/:userId/status` | Activate / deactivate | `USERS:UPDATE` | **needed** |
| 6 | `POST` | `/rbac/users/:userId/reset-password` | Send reset link or set temporary password | `USERS:UPDATE` | **needed** |
| 7 | `DELETE` | `/rbac/users/:userId` | Remove account (soft delete only, see below) | `USERS:DELETE` | optional |
| 8 | `PUT` | `/rbac/users/:userId/roles`, `/permissions` | Replace roles / overrides | `RBAC:UPDATE` | yes |

Decided 2026-10-28: a separate `USERS` module for 3 to 7 (`USERS:ADD`, `USERS:UPDATE`, `USERS:DELETE`).

### 3. Create staff: `POST /rbac/users`

Request:

```json
{
  "username": "sophea.k",
  "firstName": "Sophea",
  "lastName": "Kim",
  "email": "sophea@beasy.info",
  "phone": "+85512345678",
  "type": "FINANCE",
  "roles": ["FINANCE"],
  "sendInvite": true
}
```

| Field | Required | Rule |
|---|---|---|
| `username` | yes | Unique, case-insensitive, 3 to 50 chars, no spaces |
| `firstName`, `lastName` | yes | Non-empty |
| `email` | yes | Valid, unique, case-insensitive |
| `phone` | no | E.164 |
| `type` | yes | One of the seven values above |
| `roles` | no | Role codes that exist; `[]` means fall back to `type` (as today) |
| `sendInvite` | no, default `true` | `true`: backend emails a set-password link. `false`: backend generates a temporary password (see question A) |

We would rather the portal **never receives or shows a password**. Preferred flow: invite link by email, user sets own password on first sign-in. If a temporary password is unavoidable, return it once in the response under `temporaryPassword` and never log it.

Response `201`:

```json
{
  "code": 201,
  "status": "success",
  "message": "Staff account created",
  "data": { "user": { "...StaffUser": "" }, "inviteSent": true }
}
```

### 4. Edit profile: `PUT /rbac/users/:userId`

Editable: `firstName`, `lastName`, `email`, `phone`, `profileUrl`, `type`. Not editable here: `username`, `roles` (use #8), `status` (use #5), password. Send the full set of editable fields; omitted means unchanged is also fine, tell us which. Response `200`: `{ "data": { "user": StaffUser } }`.

### 5. Status: `PATCH /rbac/users/:userId/status`

```json
{ "status": "INACTIVE", "reason": "Left the company" }
```

`reason` optional, stored in the audit log. Inactive accounts cannot sign in, and existing tokens stop working on the next request (please confirm; today the guide says tokens are not invalidated). Response `200`: `{ "data": { "user": StaffUser } }`.

### 6. Reset password: `POST /rbac/users/:userId/reset-password`

No body. Backend emails a reset link and sets `mustChangePassword: true`. Response `200`: `{ "data": { "inviteSent": true } }`.

### 7. Delete: `DELETE /rbac/users/:userId` (optional)

We recommend **no hard delete**: orders, assignments and the activity log reference staff. If you offer `DELETE`, make it a soft delete equal to `INACTIVE` plus hidden from the default list. Response `200`: `{ "data": { "id": 42, "deleted": true } }`. If you do not offer it, the portal only shows Deactivate.

### 1. List (existing), small additions

Please add `status` (`ACTIVE` / `INACTIVE`) as a filter, and the new fields to each item. Everything else stays: `searchText`, `type`, `roleCode`, `page`, `limit`; response `{ data: StaffUser[], pagination: { totalItems, totalPages, currentPage, pageSize } }`.

## Errors

| Code | When | Example `message` |
|---|---|---|
| `400` | Validation. Please add field detail: `errors: [{ "field": "email", "message": "Invalid email" }]` | `Validation failed` |
| `401` | Missing or expired token | |
| `403` | No permission, or a rule below | `You cannot deactivate your own account` |
| `404` | Unknown `userId` | `Staff user not found` |
| `409` | Duplicate `username` or `email`; include which field | `Email already in use` |

### Rules we expect (403 with a stable message)

- You cannot deactivate, delete, or reset the password of **your own** account through these endpoints.
- You cannot deactivate or delete the **last active full-access** account.
- A non-full-access admin cannot create a full-access account, give `type: ADMIN`, or edit or deactivate a full-access account.
- Full-access role codes stay `ADMIN`, `SUPER_ADMIN`, `SUPER-ADMIN`.

## Audit

Please write create, edit, status change and password reset to the activity log with actor, target user id, and changed fields (**never** the password or token). Today role and override changes log only actor, URL and status ([review, answer 8](Frontend_User_Permissions_Backend_Review.md)); before/after values for this new set would be very helpful.

## Questions we need answered

- **A.** Invite link, temporary password, or both? Does the user have to change a temporary password on first login?
- **B.** Are `status` values `ACTIVE` / `INACTIVE`? Are there others (e.g. `PENDING` for an unaccepted invite)?
- **C.** Does an inactive account lose access on the next request, or only at token expiry?
- **D.** Is the `type` list closed, and can `type` differ from the assigned roles? (We assume yes to both.)
- **E.** Which module and action gate #3 to #7 (`RBAC:ADD/UPDATE/DELETE` or a separate `USERS` module)?
- **F.** Can a staff account be linked to a partner or cleaner profile, or is that always a separate table?
- **G.** For #4, are omitted fields unchanged (PATCH-like) or cleared?

## What the portal does with this

Create dialog from the Users list ("Add staff", then pick roles); edit form in the user detail panel; Deactivate / Activate with a confirm; Reset password with a confirm. All three hide when the admin lacks the permission, the target is the admin themselves, or the target is full access. The portal blocks nothing on its own: the rules above are enforced by backend, and the portal shows your `message`.
