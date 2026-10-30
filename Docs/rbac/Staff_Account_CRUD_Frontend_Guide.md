# Staff Account CRUD: Frontend Guide (Admin Portal)

> **Decision 2026-10-28: account actions use a new `USERS` module, not `RBAC`.** Where the text below says `RBAC ADD / UPDATE / DELETE` for create, edit, status, reset and delete, read `USERS ADD / UPDATE / DELETE`. The actions live on a separate **Users** data-table page (`/users`, sidebar Setup > Users, needs `USERS:VIEW`), not in the Roles & Permissions Users tab. Where this guide says "the Users tab", "user header" or `user-header.tsx`, read that page's table row menu. Editing roles and overrides stays on Roles & Permissions with `RBAC:UPDATE`; each table row links there (`/rbac/users?user={id}`). Backend as sent still checks `RBAC`; the change is an open request in [backend-request.md](backend-request.md). The portal already gates on `USERS`.

**For:** Portal team · **Date:** 2026-10-28 · **API branch:** `feature/bunlong-user-permissions`
**Where:** Roles & Permissions → **Users** tab (`app/routes/rbac/_components/users/`)

The Users tab can already list staff and change roles and overrides. This guide adds: **Add staff**, **Edit profile**, **Deactivate / Activate**, **Reset password**, **Delete**. The API is built and tested; follow the shapes below exactly.

---

## 1. Endpoints

Base URL is the existing admin API. Same Bearer token and envelope as the other RBAC calls (`{ code, status, message, data }`). The `api` client already unwraps `data` and shows a toast with `message` on errors.

| Action | Method | Endpoint key (add to `app/api/endpoint.ts`) | Path | Needs |
|---|---|---|---|---|
| List (existing) | GET | `RBAC_USERS` | `rbac/users` | RBAC VIEW (unchanged) |
| Create | POST | `RBAC_USERS` | `rbac/users` | USERS ADD |
| Edit profile | PUT | `RBAC_USER` | `rbac/users/{userId}` | USERS UPDATE |
| Activate / deactivate | PATCH | `RBAC_USER_STATUS` | `rbac/users/{userId}/status` | USERS UPDATE |
| Reset password | POST | `RBAC_USER_RESET_PASSWORD` | `rbac/users/{userId}/reset-password` | USERS UPDATE |
| Delete (soft) | DELETE | `RBAC_USER` | `rbac/users/{userId}` | USERS DELETE |

```ts
// app/api/endpoint.ts (add)
RBAC_USER: 'rbac/users/{userId}',
RBAC_USER_STATUS: 'rbac/users/{userId}/status',
RBAC_USER_RESET_PASSWORD: 'rbac/users/{userId}/reset-password',
```

---

## 2. Types

Update `RbacUser` in `use-rbac-user-permissions-query.ts` (one field added):

```ts
export type StaffStatus = 'ACTIVE' | 'SUSPENDED' | 'PENDING';

export type RbacUser = {
  id: number;
  username: string;
  firstName: string | null;
  lastName: string | null;
  email: string | null;
  profileUrl: string | null;
  type: StaffType;
  roles: string[];
  status: StaffStatus;
  fullAccess: boolean;
  createdAt: string;
  updatedAt: string; // NEW
};

export const STAFF_TYPES = [
  'ADMIN', 'CUSTOMER_SERVICE', 'FINANCE', 'SALES', 'TECH', 'OPERATION', 'MARKETING'
] as const;
export type StaffType = (typeof STAFF_TYPES)[number];

/** 400 / 409 responses carry field errors here: error.response.data.data.errors */
export type FieldError = { field: string; message: string };
```

**Not available** (no DB columns): `phone`, `mustChangePassword`, `lastLoginAt`. Do not add them to forms.

### Status labels

The API stores `SUSPENDED` for a deactivated account. Show it as **Inactive**.

| API value | Label | Badge |
|---|---|---|
| `ACTIVE` | Active | green |
| `SUSPENDED` | Inactive | gray |
| `PENDING` | Pending | amber (old accounts only; new staff start `ACTIVE`) |

`DELETED` users never come back from the API.

---

## 3. List: status filter

`GET rbac/users` now accepts `status`: `ACTIVE`, `SUSPENDED` (or `INACTIVE`), `PENDING`. Omit it to show everything except deleted.

```ts
// use-rbac-users-query.ts
export type RbacUsersParams = {
  searchText?: string;
  type?: string;
  roleCode?: string;
  status?: 'ACTIVE' | 'SUSPENDED' | 'PENDING'; // NEW
  page: number;
  limit: number;
};
// in `cleaned`: status: params.status || undefined
```

UI: add a third select next to "All roles": **All statuses / Active / Inactive**.

---

## 4. Create staff

**Entry point:** "Add staff" button at the top of the Users list. Show only when `hasPermission(MODULES.USERS, ACTIONS.ADD)`.

### Request

```ts
type CreateStaffPayload = {
  username: string;   // 3–50 chars: letters, numbers, . - _ @  (cannot be changed later)
  firstName: string;
  lastName: string;
  email: string;
  type: StaffType;
  roles?: string[];   // role codes; [] or omitted = follow `type`
};
```

### Response `201`

```ts
type CreateStaffResponse = {
  user: RbacUser;
  inviteSent: false;          // always false: the API has no email service
  temporaryPassword: string;  // shown ONCE
};
```

### Form

| Field | Control | Rule (also checked by API) |
|---|---|---|
| Username | text | required, 3–50, regex `^[A-Za-z0-9._@-]+$`, helper text "Cannot be changed later" |
| First name / Last name | text | required, max 100 |
| Email | email | required, valid |
| Type | select of `STAFF_TYPES` | required. Hide `ADMIN` unless the signed-in user has `fullAccess` |
| Roles | multi-select from `useRbacRolesQuery` (codes) | optional. Hide ADMIN / SUPER_ADMIN codes unless signed-in user has `fullAccess` |

### After success: show the temporary password

This is the most important UI piece. The password is returned only once.

1. Close the create form and open a **"Staff account created"** dialog.
2. Show username and temporary password in a read-only field with a **Copy** button.
3. Text: *"Give this password to {firstName}. It will not be shown again. They can change it in their profile after signing in."*
4. Only one button: **Done**. Do not close on outside click / Esc until Done.
5. **Never** store the password in React Query cache, localStorage, URL, or logs. Keep it in component state and clear it on close.

Then invalidate `[QUERY_KEY_ENUM.RBAC_USERS]` and select the new user (`?user={id}`).

```ts
export function useCreateStaffMutation() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (payload: CreateStaffPayload): Promise<CreateStaffResponse> =>
      api.post(API_ENDPOINT.RBAC_USERS, payload),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: [QUERY_KEY_ENUM.RBAC_USERS] })
  });
}
```

---

## 5. Edit profile

**Entry point:** "Edit" button in the user header (`user-header.tsx`), opens a dialog or inline form.

### Request: `PUT rbac/users/{userId}`

Send only what changed (omitted fields stay unchanged):

```ts
type UpdateStaffPayload = Partial<{
  firstName: string;
  lastName: string;
  email: string;
  type: StaffType;
}>;
```

Do **not** send `username`, `roles`, `status`, `password` or `profileUrl`: the API returns `400` for the first four, and profile photos stay in the existing profile upload flow.

Response `200`: `{ user: RbacUser }`. Invalidate `RBAC_USERS` and `[RBAC_USER_PERMISSIONS, userId]`.

Show `username` as read-only text in the form.

---

## 6. Deactivate / Activate

**Entry point:** a "Deactivate" (or "Activate" when status is `SUSPENDED`) button in the user header, with a confirm dialog.

### Request: `PATCH rbac/users/{userId}/status`

```ts
{ status: 'INACTIVE' | 'ACTIVE'; reason?: string }
```

### Deactivate

- Confirm text: *"{name} will be signed out immediately and cannot sign in until reactivated."*
- Optional "Reason" textarea (sent as `reason`, not stored yet).
- Response: `{ user }` with `status: 'SUSPENDED'`.

### Activate

- Confirm text: *"{name} will be able to sign in again with a new temporary password."*
- Response: `{ user, temporaryPassword }`. **Show the same one-time password dialog as Create** (section 4).

If the status is already the requested one the API returns `200` with message `Status unchanged`.

---

## 7. Reset password

**Entry point:** "Reset password" in the user header menu. Only for `ACTIVE` users (for `SUSPENDED` show Activate instead; the API returns `400` otherwise).

- Confirm text: *"{name} will be signed out and must use the new temporary password."*
- `POST rbac/users/{userId}/reset-password`, no body.
- Response: `{ inviteSent: false, temporaryPassword }`. Show the one-time password dialog.

---

## 8. Delete

**Entry point:** "Delete" (destructive, red) at the bottom of the header menu. Show only with `hasPermission(MODULES.USERS, ACTIONS.DELETE)`.

- It is a **soft delete**: the account is hidden and cannot sign in; history (orders, logs) is kept. There is no restore in the portal.
- Confirm dialog: title *"Delete {name}?"*, text *"The account will be removed from the list and signed out. This cannot be undone from the portal."* Ask the admin to type the username to confirm.
- `DELETE rbac/users/{userId}` → `{ id, deleted: true }`.
- On success: clear `?user=`, invalidate `RBAC_USERS`, toast "Staff account deleted".

---

## 9. When to show / hide the actions

The API enforces every rule; the portal hides buttons so admins don't hit errors. Use the signed-in user from `useAuthStore` (`user.id`, `user.fullAccess`) and the selected user.

| Action | Show when |
|---|---|
| Add staff | `USERS:ADD` |
| Edit | `USERS:UPDATE` and (target is not full access **or** I am full access) |
| Change type to/from ADMIN | I am full access, and target is not me |
| Deactivate / Activate | `USERS:UPDATE`, target is not me, (target not full access or I am full access) |
| Reset password | same as Deactivate, and target `status === 'ACTIVE'` |
| Delete | `USERS:DELETE`, target is not me, (target not full access or I am full access) |

Extend `_edit-utils.ts` with one helper so all buttons use the same logic:

```ts
export function canManageStaff(args: {
  can: boolean;               // the USERS permission for this action
  targetId: number;
  targetFullAccess: boolean;
  selfId?: number | string | null;
  selfFullAccess: boolean;
}): boolean {
  if (!args.can) return false;
  if (args.selfId != null && Number(args.selfId) === args.targetId) return false;
  if (args.targetFullAccess && !args.selfFullAccess) return false;
  return true;
}
```

The one rule the portal cannot check itself is **"last active full-access account"** (needs a count). Just show the API `message` from the `403`.

---

## 10. Errors

| Status | Meaning | What to do |
|---|---|---|
| `400` | Validation | `error.response.data.data.errors: FieldError[]` → put each `message` under its form field (`field` = form field name). Keep the form open. |
| `403` | No permission or a rule (own account, admin account, last admin) | Toast the `message`. Close the confirm dialog. |
| `404` | User no longer exists (deleted) | Toast, clear `?user=`, refetch list (same as the existing `onMissing`). |
| `409` | Username or email taken | Field error from `data.errors[0]` (`field` is `username` or `email`). |

Messages you will see (stable, safe to show as-is):

- `Only full-access admins can create an admin account`
- `Only full-access admins can change an admin account`
- `Only full-access admins can give type ADMIN`
- `You cannot change your own account type`
- `You cannot change the status of your own account`
- `You cannot delete your own account`
- `You cannot reset your own password here. Use your profile page.`
- `You cannot deactivate the last active full-access account`
- `You cannot delete the last active full-access account`
- `You cannot remove full access from the last active full-access account`
- `Activate the account first; activating returns a temporary password`
- `Username already in use` / `Email already in use`
- `Staff user not found`

---

## 11. Query invalidation summary

| After | Invalidate |
|---|---|
| Create | `[RBAC_USERS]` |
| Edit / status / reset | `[RBAC_USERS]`, `[RBAC_USER_PERMISSIONS, userId]` |
| Delete | `[RBAC_USERS]`, remove `[RBAC_USER_PERMISSIONS, userId]` |

---

## 12. Test checklist

- [ ] Create with roles and without roles; one-time password dialog shows, copy works, closing clears it.
- [ ] Create with a used username / email → field error, form stays open.
- [ ] Non-admin signed in: ADMIN type and ADMIN role are hidden; admin users show no Edit / Deactivate / Reset / Delete.
- [ ] Edit sends only changed fields; username is read-only.
- [ ] Deactivate: the user's open portal session is signed out on their next action; they cannot log in.
- [ ] Activate: new temporary password shown; user can log in with it.
- [ ] Reset password: hidden for inactive users; old password stops working.
- [ ] Delete: user disappears from list, `?user=` cleared, history still visible elsewhere.
- [ ] Own account: no Deactivate / Reset / Delete / type change.
- [ ] Status filter: All / Active / Inactive.
- [ ] `SUSPENDED` always displayed as "Inactive".
