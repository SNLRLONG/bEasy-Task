# Backend questions: staff account management

**To:** Backend · **Updated:** 2026-10-21 · **Context:** the portal Users view (Roles & Permissions) lists staff and edits roles and overrides ([Frontend_User_Permissions_Backend_Review.md](Frontend_User_Permissions_Backend_Review.md)). The first ten questions are answered.

The six endpoints we have only read staff and change their roles and overrides. None creates, edits or removes a staff account. The `bEasy-api` repo is not cloned here, so nothing below was checked against code. Please answer in this file's order; if an endpoint already exists, give the route and we will build against it.

## Does it exist today?

1. **How is a staff account created today** (script, seed, database, another admin endpoint)? If an endpoint exists, give method, path, body and permission.
2. **Is there an endpoint to edit** a staff user's profile (name, email, phone, profile image) and to **change status** (active / inactive)? Same details.
3. **Is there an endpoint to reset a password** or to set an initial one? How does a new staff member first sign in?

## If not, what we need

4. **Create staff:** `POST` with which fields? We assume `username`, `firstName`, `lastName`, `email`, `type`, `roles` (codes), and a password or an invite. Which are required, and which are unique (`username`, `email`)?
5. **Password handling:** does the admin set a temporary password, or does the backend send an invite or reset link? Is a first-login password change forced? We would rather never show or log a password in the portal.
6. **Deactivate instead of delete:** do you support `status` change (blocks login, keeps history), and is hard delete allowed at all? What happens to the user's open token and to records that reference them (orders, assignments, activity log)?
7. **Valid values:** is `type` (ADMIN, CUSTOMER_SERVICE, FINANCE, SALES, TECH, OPERATION, MARKETING) the full list for new accounts, and may it differ from the assigned roles? Which `type` values are full access (`ADMIN` only, or also `SUPER_ADMIN`)?

## Who may do it

8. **Permissions:** which module and action gate each call (`RBAC:ADD`, `RBAC:UPDATE`, `RBAC:DELETE`, or a separate staff module)? Can a non-full-access admin create a full-access account or grant a full-access role? We assume not and expect `403` with a stable message.
9. **Safety rules:** besides rejecting self-edit, do you block deactivating yourself or the last full-access account? Which status code and message?
10. **Audit:** are create, status change and password reset written to the activity log with actor and target? (Role and override changes currently log only actor, URL and status.)

## Response shape

11. Please confirm error codes: `409` for duplicate `username` / `email`, `400` with a field-level message for validation, `404` for unknown user, `403` for rule violations. We show the backend `message` in a toast, so a readable message matters.
12. Does create return the new user in the same shape as `GET rbac/users` items (`id`, `username`, `firstName`, `lastName`, `email`, `profileUrl`, `type`, `roles`, `status`, `fullAccess`, `createdAt`)?
