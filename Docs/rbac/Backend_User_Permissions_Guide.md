> **Source: sent by backend, 19 Oct 2026. Kept as received; do not edit.**
> Portal review, corrections and task split: [Portal_User_Permissions_Review.md](Portal_User_Permissions_Review.md).
> Questions we still owe backend: [backend-request.md](backend-request.md).
> Where this guide and the review differ, the review wins for portal structure; this guide wins for API behaviour until backend says otherwise.

# Admin portal integration: user roles and permissions

This guide is for implementing per-user roles and permission overrides in the
`bEasy-portal` admin frontend. It matches the API behavior documented in
`docs/permissions.md` and follows the portal's existing Axios and TanStack Query
patterns.

## What the frontend should provide

Add a **Users** view to the existing Roles & Permissions page. An administrator
with `RBAC:VIEW` can:

1. Search and filter staff users.
2. Select a user.
3. See assigned roles, inherited permissions, individual overrides, and the
   final effective permissions.

An administrator with `RBAC:UPDATE` can also:

1. Replace the selected user's roles.
2. Set a permission to inherit, grant, or deny.
3. Save the complete override list.

Keep the existing role permission view and components working as they are.
Reuse its permission catalogue, labels, section order, and permission checks.

## Existing portal behavior to keep

- `app/api/api.ts` uses `VITE_BASE_URL` and prefixes requests with
  `/api/admin`.
- The response interceptor returns `response.data.data` when the backend
  wrapper contains `data`.
- Query hooks can therefore type the inner response directly; they do not need
  the `{ code, status, message, data }` wrapper.
- The interceptor already adds the bearer token and displays backend errors
  with Sonner.
- `app/lib/permission.ts` is the source of `MODULES`, `ACTIONS`, module labels,
  action lists, and section order.
- `usePermission()` must continue to control whether edit actions are visible.

## API endpoints to add

Add these entries to `app/api/endpoint.ts` beside the current RBAC endpoints:

```ts
RBAC_USERS: 'rbac/users',
RBAC_USER_PERMISSIONS: 'rbac/users/{userId}/permissions',
RBAC_USER_ROLES: 'rbac/users/{userId}/roles'
```

The full URLs become:

| Method | URL | Use |
|---|---|---|
| `GET` | `/api/admin/rbac/users` | Paginated staff list |
| `GET` | `/api/admin/rbac/users/:userId/permissions` | Selected user's complete access details |
| `PUT` | `/api/admin/rbac/users/:userId/permissions` | Replace individual overrides |
| `PUT` | `/api/admin/rbac/users/:userId/roles` | Replace assigned roles |
| `GET` | `/api/admin/rbac/roles` | Available staff roles |
| `GET` | `/api/admin/rbac/permissions` | Permission catalogue and matrix metadata |

Add query keys to `app/constants/query-key-enum.ts`:

```ts
RBAC_USERS = 'rbac-users',
RBAC_USER_PERMISSIONS = 'rbac-user-permissions'
```

## Types

Place shared RBAC user types with the existing RBAC query types, or in the
portal's normal shared type location if the team chooses one.

```ts
export type PermissionPair = {
  module: string;
  action: string;
};

export type PermissionEffect = 'GRANT' | 'DENY';

export type PermissionOverride = PermissionPair & {
  effect: PermissionEffect;
};

export type RbacUser = {
  id: number;
  username: string;
  firstName: string | null;
  lastName: string | null;
  email: string | null;
  profileUrl: string | null;
  type: string;
  roles: string[];
  status: string;
  fullAccess: boolean;
  createdAt: string;
};

export type RbacUsersResponse = {
  data: RbacUser[];
  pagination: {
    totalItems: number;
    totalPages: number;
    currentPage: number;
    pageSize: number;
  };
};

export type RbacUserPermissions = {
  user: RbacUser;
  roles: string[];
  fullAccess: boolean;
  fromRoles: PermissionPair[];
  overrides: PermissionOverride[];
  effective: PermissionPair[];
};
```

The list endpoint has a nested `data` property inside its response payload.
After the Axios interceptor unwraps the standard backend wrapper, the hook
receives `{ data: RbacUser[], pagination: ... }`.

## Query hooks

### Staff user list

Follow the current `use-rbac-roles-query.ts` pattern:

```ts
import api from '@/api/api';
import { API_ENDPOINT } from '@/api/endpoint';
import { QUERY_KEY_ENUM } from '@/constants/query-key-enum';
import { useQuery } from '@tanstack/react-query';

type Params = {
  searchText?: string;
  type?: string;
  roleCode?: string;
  page: number;
  limit: number;
};

export default function useRbacUsersQuery(params: Params) {
  return useQuery<RbacUsersResponse>({
    queryKey: [QUERY_KEY_ENUM.RBAC_USERS, params],
    queryFn: () => api.get(API_ENDPOINT.RBAC_USERS, { params })
  });
}
```

The API supports these query parameters:

| Parameter | Behavior |
|---|---|
| `searchText` | Searches username, first name, last name, and email |
| `type` | Filters by a valid staff account type |
| `roleCode` | Matches `users.role` or the `users.type` fallback |
| `page` | Minimum 1, default 1 |
| `limit` | Range 1 to 100, default 15 |

Use the project's existing debounced search pattern when available. Keep the
current result visible while changing pages or filters if that is already the
page's query behavior.

### Selected user access

```ts
export default function useRbacUserPermissionsQuery(userId?: number) {
  return useQuery<RbacUserPermissions>({
    queryKey: [QUERY_KEY_ENUM.RBAC_USER_PERMISSIONS, userId],
    queryFn: () =>
      api.get(
        API_ENDPOINT.RBAC_USER_PERMISSIONS.replace('{userId}', String(userId))
      ),
    enabled: userId !== undefined
  });
}
```

## Mutation hooks

### Save individual overrides

```ts
type UpdateUserPermissionsPayload = {
  userId: number;
  overrides: PermissionOverride[];
};

export function useUpdateUserPermissionsMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ userId, overrides }: UpdateUserPermissionsPayload) =>
      api.put(
        API_ENDPOINT.RBAC_USER_PERMISSIONS.replace('{userId}', String(userId)),
        { overrides }
      ),
    onSuccess: (_data, variables) => {
      toast.success('User permissions updated');
      queryClient.invalidateQueries({
        queryKey: [QUERY_KEY_ENUM.RBAC_USER_PERMISSIONS, variables.userId]
      });
      queryClient.invalidateQueries({ queryKey: [QUERY_KEY_ENUM.RBAC_USERS] });
    }
  });
}
```

`PUT /permissions` replaces every override for that user. Build the full array
from the editor state on every save. Sending only changed cells would delete
previous overrides that are not included.

### Save roles

```ts
type UpdateUserRolesPayload = {
  userId: number;
  roles: string[];
};

export function useUpdateUserRolesMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ userId, roles }: UpdateUserRolesPayload) =>
      api.put(API_ENDPOINT.RBAC_USER_ROLES.replace('{userId}', String(userId)), {
        roles
      }),
    onSuccess: (_data, variables) => {
      toast.success('User roles updated');
      queryClient.invalidateQueries({
        queryKey: [QUERY_KEY_ENUM.RBAC_USER_PERMISSIONS, variables.userId]
      });
      queryClient.invalidateQueries({ queryKey: [QUERY_KEY_ENUM.RBAC_USERS] });
    }
  });
}
```

Send `roles: []` only when the intended behavior is to clear the explicit role
array and use `user.type` as the fallback role.

## Permission editor behavior

Each module/action cell has three states:

| UI state | Saved override | Result |
|---|---|---|
| **Inherit** | No row | Follow the selected user's roles |
| **Grant** | `GRANT` | Add the permission even when roles do not provide it |
| **Deny** | `DENY` | Remove the permission even when roles provide it |

Use the returned arrays instead of recalculating backend rules:

- `fromRoles` controls the inherited indicator.
- `overrides` controls the editable tri-state value.
- `effective` controls the final access preview.

For `fullAccess: true`, treat the matrix as informational and read-only. The
API bypasses module/action checks for those accounts, so the returned
`effective` role rows are not a complete list of everything they may access.

Suggested cell display:

```text
Inherited and effective       neutral checked state
Inherited but denied          red Denied state
Not inherited but granted     blue Granted state
Not inherited, no override    neutral empty state
```

Do not convert every effective permission into a `GRANT`. Inherited permissions
must stay inherited so later role changes still apply.

## Screen integration

Use the existing `/rbac` page and add a top-level switch between:

- **Roles**: the current role permission editor.
- **Users**: the new user-specific editor.

For the Users view:

1. Left section: search, filters, paginated staff list.
2. Main section: selected user identity, role selector, and permission matrix.
3. Matrix: reuse `PermissionsMatrix` styling and the catalogue returned by
   `useRbacPermissionsQuery()`.
4. Header badges: show account type, assigned roles, and `Full access` where
   applicable.
5. Summary: show inherited, granted, denied, and effective counts.

Reuse `moduleSection`, `moduleLabel`, `RBAC_SECTION_ORDER`, `MODULES`, and
`ACTIONS`. Do not create a second list of permission modules in the new page.

## Edit permissions and disabled states

```ts
const { hasPermission } = usePermission();
const canUpdate = hasPermission(MODULES.RBAC, ACTIONS.UPDATE);
const currentUserId = Number(user?.id);
const isSelf = selectedUser.id === currentUserId;
const readOnly = !canUpdate || isSelf || selectedDetails.fullAccess;
```

- Hide or disable save controls without `RBAC:UPDATE`.
- Disable editing for the logged-in account; the API also rejects it.
- Show full-access permission details as read-only.
- A non-full-access administrator may see a full-access account but cannot
  change its roles.
- Let the API remain the final authority for privilege escalation rules.

## Loading, switching, and failure states

- Show the user list skeleton only for its first load.
- Show a detail skeleton when selecting a user whose details are not cached.
- Keep the selected user ID in component state or the URL; do not clear the
  whole screen while the detail query changes.
- Disable Save while its mutation is pending.
- On `400`, preserve the draft so the administrator can correct it.
- On `403`, preserve the draft and show the backend message from the existing
  Axios interceptor.
- On `404`, clear the selection and refresh the user list.
- Do not optimistically change permission state before the API succeeds.

## Example responses as received by portal hooks

Because `api.ts` unwraps the standard backend response, a user-list hook sees:

```json
{
  "data": [
    {
      "id": 12,
      "username": "dara",
      "firstName": "Dara",
      "lastName": "Sok",
      "email": "dara@example.com",
      "profileUrl": null,
      "type": "CUSTOMER_SERVICE",
      "roles": ["CS"],
      "status": "ACTIVE",
      "fullAccess": false,
      "createdAt": "2026-10-19T01:00:00.000Z"
    }
  ],
  "pagination": {
    "totalItems": 1,
    "totalPages": 1,
    "currentPage": 1,
    "pageSize": 15
  }
}
```

A selected-user hook sees:

```json
{
  "user": {
    "id": 12,
    "username": "dara",
    "type": "CUSTOMER_SERVICE",
    "roles": ["CS"],
    "status": "ACTIVE",
    "fullAccess": false
  },
  "roles": ["CS"],
  "fullAccess": false,
  "fromRoles": [{ "module": "ORDER", "action": "VIEW" }],
  "overrides": [
    { "module": "CHAT", "action": "VIEW", "effect": "GRANT" }
  ],
  "effective": [
    { "module": "ORDER", "action": "VIEW" },
    { "module": "CHAT", "action": "VIEW" }
  ]
}
```

## Implementation file checklist

Expected portal changes:

```text
app/api/endpoint.ts
app/constants/query-key-enum.ts
app/hooks/query/use-rbac-users-query.ts
app/hooks/query/use-rbac-user-permissions-query.ts
app/hooks/mutations/use-update-user-permissions-mutation.ts
app/hooks/mutations/use-update-user-roles-mutation.ts
app/routes/rbac/rbac.tsx
app/routes/rbac/_components/... user list/editor components
```

Keep the exact component split consistent with the current RBAC route. The API
does not require changes to `app/api/api.ts`, the auth store, login, profile
query, or `usePermission()`. Login and profile already receive effective
permissions in the same array shape.

## Review checklist

- [ ] The Users view requires `RBAC:VIEW` through the existing route handle.
- [ ] Edit controls require `RBAC:UPDATE`.
- [ ] Search and pagination send backend-supported parameters.
- [ ] Role options come from `GET /rbac/roles`.
- [ ] Matrix modules/actions come from `GET /rbac/permissions`.
- [ ] Role, override, and effective states are visually different.
- [ ] Saving overrides sends the complete override array.
- [ ] Clearing overrides sends `{ "overrides": [] }`.
- [ ] Clearing roles explains the `user.type` fallback.
- [ ] Self-edit and full-access override controls are disabled.
- [ ] Detail and list queries are invalidated after successful mutations.
- [ ] API error messages remain visible through the existing interceptor.
- [ ] Existing role management still works without behavior changes.
- [ ] A refreshed login/profile reflects the edited user's effective access.
