# Plan: User-level roles and permissions (Roles & Permissions › Users)

Updated: 2026-10-19 · Owner: Suntel · Planner: agent (no code written)
Docs: `Docs/rbac/` (guide from backend, review, open questions)

## Goal
Let an admin with `RBAC:VIEW` inspect any staff user's roles, inherited permissions, overrides and effective access, and an admin with `RBAC:UPDATE` change the user's roles and GRANT/DENY overrides. Portal only; backend endpoints are said to be live (unverified).

## Findings that shape the plan
Full list in `Docs/rbac/Portal_User_Permissions_Review.md`. The ones that change the work:
1. A separate route `rbac/users` plus a shared tab bar, not more state inside the 334-line `rbac.tsx`.
2. `PermissionsMatrix` is binary and role-specific: leave it alone, build `UserPermissionsMatrix`, share the section builder through a small extraction.
3. Roles and overrides save separately; override draft survives a roles refetch.
4. Unknowns for backend (role code vs name, actor `fullAccess`, VIEW coupling, session refresh) sit in `Docs/rbac/backend-request.md`. Only the editor waits on them.

## Out of scope
Creating or editing staff accounts, role create/delete (already in Roles), changing `usePermission`, the auth store, login or profile, forcing a live refresh of another user's session, new i18n keys.

## Tasks

Sizes: S ≤ 1 day · M 2–4 · L 5+. Tier from `PIPELINE.md`.

| # | Task file (in `2-todo/`) | Repo | Depends on | Tier | Size |
|---|---|---|---|---|---|
| 1 | `Cross-rbac-users-backend-questions` | Docs | none | T0 | S |
| 2 | `Portal-rbac-users-data-layer`: endpoints, keys, types, 2 query + 2 mutation hooks | Portal | none | T2 | S |
| 3 | `Portal-rbac-users-shell`: extract section builder, `rbac/users` route, Roles/Users tabs | Portal | none | T2 | S |
| 4 | `Portal-rbac-users-list`: search, type/role filters, paginated list, selection in URL | Portal | 2, 3 | T2 | M |
| 5 | `Portal-rbac-users-detail-view`: header, badges, summary, read-only tri-state matrix | Portal | 2, 3 | T2 | M |
| 6 | `Portal-rbac-users-edit`: role selector + save, override editing + save, guards, error states | Portal | 4, 5, answers 1 to 4 in backend-request | T3 | L |

Tasks 2 and 3 can run in parallel (no shared files). Tasks 4 and 5 can run in parallel after both (list and detail components are separate; both touch `users.tsx`, so the first to merge owns it and the second rebases).

## Overlaps with other work
- `Portal-marketing-rbac` (2-todo): adds modules and `MODULE_META` labels in `app/lib/permission.ts`. This plan never edits that file. Both touch `app/routes/rbac/*`; check `git worktree list` and the file lists before claiming.
- `Portal-journey-cms` (3-in-progress) is on branch `feat/journey-cms` in `bEasy-portal/`. Start RBAC worktrees from `main`, not from that checkout.

## Open questions
All backend questions: `Docs/rbac/backend-request.md`. Product decision still needed from the business: none found.
