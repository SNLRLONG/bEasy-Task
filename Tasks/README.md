# Tasks

One file per task, moved between folders as it progresses. The folder is the status.

| Folder | Meaning | Move here when |
|---|---|---|
| `1-backlog/` | Idea or later; not scheduled | Captured, no decision yet |
| `2-todo/` | Decided, ready to start | Scope and owner agreed |
| `3-in-progress/` | Being built in a worktree | Worktree + branch created |
| `4-testing/` | Built; QA / dev backend / device test | PR open or merged to `dev` |
| `5-done/` | Merged to `main`, changelog written, worktree deleted | Tested and merged |
| `6-deploy/` | Released (store, hosting, backend) | Live in production |

## File name
`<repo>-<short-name>.md`, e.g. `portal-acquisition-export.md`. Copy `_TEMPLATE.md`.

## Worktree rules
- `bEasy-portal/` (and each repo's main folder) is the **final** checkout. Never delete it.
- One task = one worktree = one branch, as a sibling folder: `bEasy/<repo>-<short-name>/`, branch `feat/<short-name>`.
- When the task is merged: delete the worktree (`git worktree remove <path>`), delete the branch, move the task file to `5-done/`. Uncommitted work is never removed without asking.
- The root `bEasy/` repo (local only, no remote) tracks `Tasks/`, `Docs/`, `CLAUDE.md`, `TODO.md`. Add each new worktree folder to the root `.gitignore`, and commit at the end of a session or when a task changes stage.
- Before starting, list live worktrees: `git -C <repo> worktree list`. Two tasks must not touch the same files; note overlaps in the task file.

## Changelog (per repo, on merge to `main`)
Each repo root has `CHANGELOG.md` (newest first). Add one entry in the merge commit:

```
## 2026-10-20 · feat(export): acquisition Excel report
- Owner: <name>
- Task: Tasks/5-done/portal-acquisition-export.md
- Branch/PR: feat/acquisition-export · #123
- What changed: <1-3 bullets, user-visible>
- Needs: <backend / config / migration, or "none">
```

## Keeping docs in sync
Task files link to docs; they do not copy them. Backend asks and replies follow the rules in the root `CLAUDE.md`.
