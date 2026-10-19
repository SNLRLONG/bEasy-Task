# RBAC users: send questions to backend and record answers

- **Repo:** root (`Docs/`)
- **Owner:** Suntel
- **Tier:** T0 · docs only
- **Claimed by:** none
- **Started / updated:** – / 2026-10-19
- **Worktree:** none (docs only, edited in the root repo)
- **Docs:** `Docs/rbac/Portal_User_Permissions_Review.md`, `Docs/rbac/Backend_User_Permissions_Guide.md` · plan: `Tasks/_plans/Portal-rbac-users.md`
- **Depends on:** none
- **Touches:** `Docs/rbac/backend-request.md`, `Docs/rbac/Portal_User_Permissions_Review.md` ("Backend answers")

## Goal
Get answers to the 10 questions so the edit task is not blocked.

## Scope
- Send `Docs/rbac/backend-request.md` to backend (the user sends it; do not message anyone yourself).
- On each reply: delete the item from the request file, write the answer under "Backend answers" in the review doc, and adjust the affected task file.
- If `bEasy-api` gets cloned: read-only check of route names, error codes and the VIEW rule against the guide; record mismatches in the review doc (cite `path:line`).

## Gates (T0)
- [ ] G1 scope  - [ ] G8 commit message

## Done when
- [ ] Questions 1 to 5 answered and recorded
- [ ] Affected task files updated
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
