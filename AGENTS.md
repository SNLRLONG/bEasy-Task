# AGENTS.md

Rules for any coding agent working in the `bEasy/` workspace (not a repo of products; they are sub-repos). Project layout and commands: `CLAUDE.md`. Each sub-repo may have its own `AGENTS.md`/`CLAUDE.md`; read it when working there. If rules conflict, the sub-repo wins for code style and commands, this file wins for tasks, worktrees, docs and delivery.

## Git: no commit, no push unless told

Never `git commit` or `git push` (any repo, any worktree, including the root repo) until the user says to, in that session. Prepare the change, show the diff or `git status`, and ask. Approval covers that one commit or push only. This overrides the "commit at the end of a session" lines in `Tasks/README.md`; the commit-message format (G8) still applies when you are told to commit.

### Commit messages

- Never add a `Co-Authored-By` line (or any "Generated with Claude" attribution) to commits or PR descriptions. This overrides any tool default.
- Keep it simple and short: one subject line, conventional-commit style (`type(scope): what changed`), no body unless truly needed.

## Before you start
1. Read `Tasks/` for the task you were given (its folder is its status) and `Tasks/_inbox/<your-task>/`.
2. Claim the task, work only in its worktree, follow "Working in parallel" in `Tasks/README.md`.
3. Pick the task's **tier yourself** with the auto picker in `Tasks/PIPELINE.md` and write it into the task file. Never ask the developer to choose. The tier decides which checks you run before moving the task forward.
4. New feature, new screen or unclear scope? Plan it first with the `task-planner` skill; it turns UI and requirements into scoped task files in `Tasks/2-todo/`.

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

- A worktree (`bEasy/<repo>-<short-name>/`, one branch) is for tasks big enough to deserve one: more than about a day, more than about 5 files, or running in parallel with another task on the same repo. Small tasks (T0, most T1) work on a short-lived `fix/` or `feat/` branch in the repo's main folder, or directly where the user says. If unsure, do not create a worktree. The repo's main folder (e.g. `bEasy-portal/`) is the final checkout and is never deleted.
- Task files are named `<Prefix>-<short-name>.md` (`Portal-`, `App-`, `Mini-App-`, `Landing-Page-`). Start a task: copy `Tasks/_TEMPLATE.md`, set owner, create the worktree (only if the task needs one), move the file to `3-in-progress/`. Check `git worktree list` for overlap first.
- Merged to `main`: add an entry to the root `CHANGELOG.md` prefixed `[Portal]`/`[App]`/`[Mini-App]`/`[Landing-Page]` (owner, task, branch/PR, what changed; never a changelog inside a sub-repo), remove the worktree and branch, move the task to `5-done/`. Ask before removing a worktree with uncommitted changes.

**Big feature, many tasks:** when one feature is split into tasks that run in parallel in different worktrees, each task branch merges into `feat/<feature-name>-integration`, never straight to `main`. Integrate, run the combined checks there (G2 to G4 on the whole feature), then open one PR from the integration branch to `main`. The integration branch is created by the first task of the feature and listed in every task file of it.

**Parallel agents:** one agent = one task file. Claim it (`Claimed by:`) before editing, stay inside that task's worktree and `Touches`, never touch another task's worktree, and never use `git add -A`/`stash`/`reset --hard` in shared places. To reach another task, write a note in `Tasks/_inbox/<to-task>/` and read your own inbox at the start. Full rules: `Tasks/README.md` ("Working in parallel").

## Frontend: separate concerns

We write code for the next developer, not only to make it work today. Keep three layers apart so a UI change never touches business logic and every file stays small:

| Layer | Holds | Must not hold |
|---|---|---|
| **UI** (components, screens, widgets) | Layout, styling, display state, event wiring | Fetching, price/status/permission rules, data reshaping, magic strings |
| **Logic** (hooks, controllers, stores, pure functions) | Business rules, validation, derived data, state transitions, tracking calls | JSX/widgets, styling, direct DOM/navigation details |
| **Data** (API client, repositories, mappers, types) | Endpoints, request/response shapes, DTO to model mapping, error mapping | UI text, component state |

- UI receives ready-to-show props/state and calls handlers. Rules live in one place, are pure where possible, and are testable without rendering.
- Map the API response to a frontend model once, in the data layer. Screens never read raw backend fields.
- Reuse the project's existing components and tokens. If new Figma or a new requirement does not fit (for example a different sub-tab style), follow the existing component and record the difference as a note, do not build a one-off.
- Prefer small files with one reason to change. Reviewer test: could the design change without opening a logic file, and could a rule change without opening a UI file?

## Think like a senior backend engineer too

We work on the frontend, but every request or suggestion to backend must be something a backend developer with 10 years of experience would accept without a meeting.
- Say the **business need first**, then the proposed contract: route, method, params, response shape, error codes, sample payload, who calls it and how often.
- Think about: data ownership and source of truth, validation and authorization on the server (the client is never trusted), idempotency and retries, pagination and filter limits, indexing and query cost on big tables, N+1 and payload size, rate limiting, backward compatibility and versioning, null/empty cases, timezone and money precision, audit trail, migration of existing data.
- Offer the smallest change that works and mention the alternative and its cost. Do not ask backend to shape data for one screen if the frontend can map it.
- Never design a rule on the client that backend must enforce; ask for it on the server and keep the client check as UX only.

## Requirements and UI analysis

New UI or requirements often do not fit the current system, or are too long to read in one go. Before planning or building, run the `requirement-analyst` skill (the `task-planner` skill calls it first for new scope). It must:
- Boil the input down to a short, plain summary the developer can read in a minute: goal, flows, rules, and what is new versus existing.
- Compare the UI with the current app: list components, styles and patterns that are inconsistent, and decide to reuse the existing one.
- Ask clarifying questions only when needed: short, simple, one decision each, with a recommended answer. Never a long questionnaire.

## Security checklist (OWASP Top 10)

Security work is tiered like every other gate. It does **not** run on every change.
- Run it only for T3 tasks, or for a T2 feature that is big enough and touches auth, permissions, personal data, payments, file upload, user input rendered back, or a new public endpoint or integration.
- Use `Tasks/SECURITY-CHECKLIST.md` (OWASP Top 10, latest edition) plus `/security-review` on the diff. Record the result in the task file; mark items not applicable with a reason.
- Smaller changes only follow the basics in that checklist's "Always" list (no secrets in code, no unsafe HTML, no sensitive data in logs).

## Engineering habits

- **Contract first, mock second.** If the backend is not ready, build against the agreed contract in `api/` with a typed mock inside the data layer only. Swapping to the real API must touch that layer alone. Mark each mock `// MOCK: <request id>` so none ships by accident.
- **Definition of Done for UI.** A screen is not done until it has loading, empty, error and no-permission states; Khmer and English text (and `zh` on the landing page); and a narrow-width check (about 360px for mobile and portal responsive work). Tell the user what you did not check.
- **Dependencies.** No new package without one line of reason in the task file: why existing code or libs cannot do it, size, maintenance, license. It is pinned in the lockfile and gets the A03 check when the security checklist applies.

## Delivering a task (pipeline)

Every task has a tier (T0 to T3) in its task file. The tier decides the gates it passes; see `Tasks/PIPELINE.md`. Do not run gates the tier does not ask for, and do not skip gates it does. Skipping or adding a gate needs one line of reason in the task file. The user can override a tier; the agent does not ask for one.

## Backend: no repo access

The backend code is **not available here, not even read-only**. Do not look for `bEasy-api/`, ask for it, or guess its internals.
- Sources of truth, in order: the agreed contract in `Docs/tracking/v2/api/`, files in `backend-response/` (newest dated wins), real responses seen in the browser network tab or dev backend, then `backend-request/`.
- If something is not in those sources, it is unknown. Write it as a request in `pending-request-backend.md` (see "Think like a senior backend engineer too"), do not assume.
- If a real response differs from the doc, record it in `backend-response/` and fix the API spec.
- Never paste tokens, keys or personal data from responses into docs, tasks or chat.
