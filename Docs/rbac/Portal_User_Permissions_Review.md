# Portal review: user roles and permissions

Updated: 2026-10-19 · Reviewer: agent · Inputs: [Backend_User_Permissions_Guide.md](Backend_User_Permissions_Guide.md) and the current `bEasy-portal` code (`app/routes/rbac/*`, `app/hooks/query/use-rbac-*`, `app/lib/permission.ts`, `app/store/auth-store.ts`, `app/hooks/use-auth.ts`).

## What backend asks for

Inside **Roles & Permissions** (`/rbac`) add a **Users** view: list staff, pick one, see roles / inherited / override / effective permissions; with `RBAC:UPDATE` replace the user's roles and save a full GRANT/DENY override list. Six endpoints, two of them (`rbac/roles`, `rbac/permissions`) already used by the portal.

## What matches the portal (keep as written)

- API client, `/api/admin` prefix, unwrap of `response.data.data`, token and error toast: all as described (`app/api/api.ts:30`).
- Hook locations and style (`hooks/query/use-*-query.ts`, `hooks/mutations/use-*-mutation.ts`), `QUERY_KEY_ENUM` and `API_ENDPOINT` additions, `{placeholder}` `.replace()` style used by `RBAC_ROLE_PERMISSIONS`.
- Route guard: `export const handle = { module: MODULES.RBAC, action: ACTIONS.VIEW }`.
- Module list, labels and sections come from `MODULES`, `MODULE_META`, `RBAC_SECTION_ORDER`, plus the catalogue from `GET /rbac/permissions`. No second module list.
- Tri-state, "send the complete array", "do not recalculate rules", "no optimistic update", error handling per status.

## What does not fit the portal (corrections)

| # | The guide says | Problem | Decision |
|---|---|---|---|
| 1 | "Use the existing `/rbac` page and add a top-level switch" and list `app/routes/rbac/rbac.tsx` as a file to change | `rbac.tsx` is one 334-line component that owns role selection, edit mode, draft state and four dialogs. Adding user state to it makes it ~600 lines and risks the existing role editor, which the guide says must not change | `rbac.tsx` stays the **Roles** page. Add a sibling route `rbac/users` (`routes/rbac/users.tsx`) and a small shared tab bar (Roles / Users) rendered by both. Sidebar stays one entry, "Roles & Permissions". Same `handle` guard on both |
| 2 | "Reuse `PermissionsMatrix`" | `PermissionsMatrix` is binary (`checkedSet`), has Add/Edit-module controls and is built for roles. A tri-state cell with inherited/granted/denied states cannot be added without bending it | Do **not** edit `PermissionsMatrix`. New `UserPermissionsMatrix` that reuses `PERMISSION_COLUMNS`, `GRID_TEMPLATE`, `NaCell` look and the section list. Extract the section-building `useMemo` (inline in `rbac.tsx`) into a shared helper (`useRbacSections`) so both pages use one copy. This is a small refactor of `rbac.tsx`, done in its own task with a role-page regression check |
| 3 | `roles: string[]` and a role selector fed by `GET /rbac/roles` | Portal roles carry `name` and `code`; the guide never says which one a user's `roles` holds | Backend question 1. Selector must submit whatever backend confirms; assume `code` until answered |
| 4 | `readOnly = !canUpdate \|\| isSelf \|\| selectedDetails.fullAccess`, and "do not change the auth store" | The logged-in admin's own `fullAccess` is not in the profile, so the UI cannot know a non-full-access admin must not edit a full-access user | Backend question 3. Until answered: show controls, rely on `403` toast, keep the draft |
| 5 | Role selector and overrides appear to save together ("Save the complete override list") | Changing roles changes `fromRoles`, which the portal must not recalculate. A single save would send roles and overrides with a preview that is wrong until refetch | **Two independent saves**: Roles section (own Save), Overrides (own Save). After a roles save, refetch; the unsaved override draft is kept and rebased onto the new `fromRoles`. Switching user with a dirty draft asks to discard |
| 6 | Clearing roles "explains the fallback" | Easy to do by accident | Confirm dialog before sending `roles: []`, text explains the `user.type` fallback |
| 7 | VIEW coupling not mentioned | The role editor forces VIEW on when any action is on. For per-user overrides the rule is unknown | Backend question 4. Until answered the matrix does not auto-couple, and shows the backend `400` message if rejected |
| 8 | Staff `type` filter | Valid values not given | Backend question 5 |
| 9 | "A refreshed login/profile reflects the edited user's effective access" | The portal keeps permissions in the Zustand `auth-store` (localStorage) and refreshes them only through the `currentUser` query on load or login. Another admin's edit will not reach that user's open session | Backend question 6. Show a one-line note after saving. No new refresh mechanism in this scope |
| 10 | Example hooks | Snippets omit `useMutation`, `useQueryClient`, `toast` imports; the list hook has no `placeholderData` | Follow `use-update-role-permissions-mutation.ts` for style; use `placeholderData: keepPreviousData` and `useDebounce` for search |
| 11 | "Placed with existing RBAC query types" | Types live inside each hook file today (`RbacRoleItem` in `use-rbac-roles-query.ts`) | Put new types in the hook files that own them; shared `PermissionPair`/`PermissionOverride` in `hooks/query/use-rbac-user-permissions-query.ts` and import from there |
| 12 | Backend cites `docs/permissions.md` | Not in this workspace; `bEasy-api` is not cloned | Contract unverified against code. When cloned (read-only), check route names, error codes and the VIEW rule, and note mismatches in a backend-response file |

## Other findings

- The legacy `users`, `roles`, `new-user` routes are commented out in `app/routes.ts:101-104` and run on dummy data. There is **no staff-user create/edit screen** and this scope does not add one. Users here are only listed from `GET /rbac/users`.
- The new modules from `Portal-marketing-rbac` will appear in the user matrix automatically (catalogue-driven) but need `MODULE_META` entries for labels. That task owns that edit; this scope must not touch `app/lib/permission.ts`.
- RBAC screens use plain English strings, not i18n keys. Follow that.
- Tier: the feature reads and changes who can access what, so it is T3 for the write path. Read-only tasks are T2.

## Structure

```text
app/routes/rbac/
  rbac.tsx                       Roles page (kept; section builder extracted)
  users.tsx                      Users page (new route rbac/users, same handle guard)
  _components/
    rbac-tabs.tsx                Roles | Users switch (new)
    permissions-matrix.tsx       untouched
    users/                       user list, user header, role selector,
                                 user-permissions-matrix, summary (new)
  _utils.ts                      untouched (reused)
app/hooks/query/use-rbac-users-query.ts, use-rbac-user-permissions-query.ts
app/hooks/mutations/use-update-user-permissions-mutation.ts, use-update-user-roles-mutation.ts
app/hooks/use-rbac-sections.ts   shared section builder
```

## Task split

Plan: `Tasks/_plans/Portal-rbac-users.md`. Order: data layer → shell → list and read-only detail → editor.

## Backend answers

None yet. See [backend-request.md](backend-request.md).
