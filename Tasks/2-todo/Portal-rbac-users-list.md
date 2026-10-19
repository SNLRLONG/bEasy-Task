# RBAC users: searchable, paginated staff list

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T2 · new UI over a new API; read-only
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** `bEasy/bEasy-portal-rbac-users-list/` (create when claimed) · **Branch:** `feat/rbac-users-list`
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** Portal-rbac-users-data-layer, Portal-rbac-users-shell; backend answer 5 for the type filter (ship role filter first if still open)
- **Touches:** `app/routes/rbac/users.tsx`, `app/routes/rbac/_components/users/user-list.tsx` (new)

## Goal
Left panel of the Users view: find a staff user and select one.

## Scope
- Search box (debounced with `useDebounce`), type filter, role filter (options from `GET /rbac/roles`), pagination (limit 15).
- Row: avatar or initials, name or username, type, role badges, `Full access` badge, status.
- Skeleton only on first load; keep the previous page while paging or filtering.
- Selected user id stored in the URL search param `?user=<id>` so refresh and back work.
- Empty and error states. On a `404` from the detail task, clear the selection and refetch the list (hook point only).

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
(G6 security review: skipped unless the diff touches auth or input handling; write `Skipped G6: reason`.)

## Done when
- [ ] Search, filters and pagination send only the parameters the guide lists
- [ ] Selection survives a refresh
- [ ] No horizontal scroll at laptop width (1280px)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
