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
| `7-deleted/` | Dropped, duplicate or obsolete; kept for history, hidden on the board | Decided not to do it |

## Planning
`Tasks/_plans/<Prefix>-<feature>.md` holds the plan for a feature (flow, scope, task breakdown, open questions). Written by the `task-planner` skill before tasks go to `2-todo/`.

## File name
`<Prefix>-<short-name>.md`, e.g. `Portal-acquisition-export.md`. Prefix = `Portal`, `App`, `Mini-App`, `Landing-Page` (or `Backend`/`Cross` for work across repos). Same prefix for inbox names. Copy `_TEMPLATE.md`.

## Worktree rules
- `bEasy-portal/` (and each repo's main folder) is the **final** checkout. Never delete it.
- A task that is big enough (see `AGENTS.md`) gets its own worktree and branch, as a sibling folder: `bEasy/<repo>-<short-name>/`, branch `feat/<short-name>`.
- When the task is merged: delete the worktree (`git worktree remove <path>`), delete the branch, move the task file to `5-done/`. Uncommitted work is never removed without asking.
- The root `bEasy/` repo (local only, no remote) tracks `Tasks/`, `Docs/`, `CLAUDE.md`, `TODO.md`. Add each new worktree folder to the root `.gitignore`, and commit at the end of a session or when a task changes stage.
- Before starting, list live worktrees: `git -C <repo> worktree list`. Two tasks must not touch the same files; note overlaps in the task file.

## Changelog (one file at the root: `bEasy/CHANGELOG.md`)
Tasks live at the root, so the changelog does too. **Do not add a `CHANGELOG.md` inside any sub-repo.** On each merge to `main`, add one entry at the top, prefixed with the project:

```
## 2026-10-20 · [Portal] feat(export): acquisition Excel report
- Owner: <name>
- Task: Tasks/5-done/Portal-acquisition-export.md
- Branch/PR: feat/acquisition-export · #123
- What changed: <1-3 bullets, user-visible>
- Needs: <backend / config / migration, or "none">
```

Prefixes: `[Portal]`, `[App]`, `[Mini-App]`, `[Landing-Page]`, `[Backend]`, `[Cross]`. Commit it in the root repo.

## Keeping docs in sync
Task files link to docs; they do not copy them. Backend asks and replies follow the rules in the root `CLAUDE.md`.

## Working in parallel (one agent, one task)

**Claim.** An agent works on exactly one task file. It writes `Claimed by: <session/agent name>, <date>` in that file before editing code, and clears it when it stops. If a task is already claimed, do not work on it; ask the user.

**Stay in your lane.**
- Edit code only inside your task's worktree and the paths in its `Touches`. Never edit another task's worktree, and never switch branches in a checkout that another task uses.
- Never run `git add -A`, `git commit -a`, `git stash`, `git reset --hard` or `git checkout .` in a shared place. In the root repo, `git add <specific paths>` only.
- Need a change outside your lane (another task's files, a shared component, shared docs)? Do not make it. Leave a note (below) and carry on with your own scope.

**Shared files** (`CLAUDE.md`, `Docs/**`, `backend-request/pending-request-backend.md`, `TODO.md`): re-read right before editing, change only your own lines with a small edit, never rewrite the whole file.

**Talking to each other: `Tasks/_inbox/`.** One small file per message, so nothing conflicts:

`Tasks/_inbox/<to-task>/<YYYY-MM-DD>-from-<task>-<topic>.md`

```
From: portal-journey-cms   To: portal-acquisition-export   Date: 2026-10-16
Type: heads-up | question | blocker | done
Message: <2-5 lines: what you need or changed, and which files>
Needs reply: yes/no
```

- At the start of work, read `Tasks/_inbox/<your-task>/`. When handled, move the note to `Tasks/_inbox/_read/` (do not delete).
- Use `blocker` when your task cannot continue without the other. The user decides who goes first.
- Cross-task decisions that outlive the session belong in the task file or the docs, not only in the inbox.

**Sub-agents.** When an agent spawns sub-agents, each gets: its task file path, its worktree path, and the "Stay in your lane" rules above. Sub-agents report in their final message, and write an inbox note when the finding concerns another task. One owner per worktree: sub-agents do not share a checkout for writing.
