# AGENTS.md

Rules for any coding agent working in the `bEasy/` workspace (not a repo of products; they are sub-repos). Project layout and commands: `CLAUDE.md`. Each sub-repo may have its own `AGENTS.md`/`CLAUDE.md`; read it when working there. If rules conflict, the sub-repo wins for code style and commands, this file wins for tasks, worktrees, docs and delivery.

## Before you start
1. Read `Tasks/` for the task you were given (its folder is its status) and `Tasks/_inbox/<your-task>/`.
2. Claim the task, work only in its worktree, follow "Working in parallel" in `Tasks/README.md`.
3. Know the task's **tier** (`Tasks/PIPELINE.md`). It decides which checks you run before moving the task forward.

## Journey tracking docs (keep in sync)

`Docs/tracking/v2/README.md` is the index. One fact lives in one place; update it there and link, never copy.

| Fact | Single home |
|---|---|
| Open asks to backend | `backend-request/pending-request-backend.md` (open items only; later items in `backlog.md`) |
| What backend replied / what is live | `backend-response/` (newest dated file wins) |
| Endpoint shapes and live status | `api/Journey_Backend_API_Spec.md` |
| Mobile events and data model | `architecture/` |

When something changes, do all of these in the same session:
1. **Backend replies** → delete the answered item from `pending-request-backend.md`, update the status in the API spec, and add the reply to `backend-response/`.
2. **Portal or app behaviour changes** (new param, new call, new header) → update the API spec or `architecture/`, and add or close the matching request.
3. **Finished or replaced doc** → move to `archive/` with a one-line banner pointing to its replacement.
4. Set `Updated: <date>` at the top of any request file you edit. No history logs inside docs; git and the response files hold history.

Do not save doc contents, status or dates in Claude memory; memory only holds pointers to the index above.

## Tasks and worktrees

`Tasks/` holds one file per task; its folder is the status (`1-backlog` → `2-todo` → `3-in-progress` → `4-testing` → `5-done` → `6-deploy`). Rules and templates are in `Tasks/README.md`.

- One task = one worktree (`bEasy/<repo>-<short-name>/`) = one branch. The repo's main folder (e.g. `bEasy-portal/`) is the final checkout and is never deleted.
- Task files are named `<Prefix>-<short-name>.md` (`Portal-`, `App-`, `Mini-App-`, `Landing-Page-`). Start a task: copy `Tasks/_TEMPLATE.md`, set owner, create the worktree, move the file to `3-in-progress/`. Check `git worktree list` for overlap first.
- Merged to `main`: add an entry to the root `CHANGELOG.md` prefixed `[Portal]`/`[App]`/`[Mini-App]`/`[Landing-Page]` (owner, task, branch/PR, what changed; never a changelog inside a sub-repo), remove the worktree and branch, move the task to `5-done/`. Ask before removing a worktree with uncommitted changes.

**Parallel agents:** one agent = one task file. Claim it (`Claimed by:`) before editing, stay inside that task's worktree and `Touches`, never touch another task's worktree, and never use `git add -A`/`stash`/`reset --hard` in shared places. To reach another task, write a note in `Tasks/_inbox/<to-task>/` and read your own inbox at the start. Full rules: `Tasks/README.md` ("Working in parallel").

## Delivering a task (pipeline)

Every task has a tier (T0 to T3) in its task file. The tier decides the gates it passes; see `Tasks/PIPELINE.md`. Do not run gates the tier does not ask for, and do not skip gates it does. Skipping or adding a gate needs one line of reason in the task file. The user can override any tier.
