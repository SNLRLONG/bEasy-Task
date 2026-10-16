# Delivery pipeline

Not every task needs every check. A task gets a **tier** by risk; the tier lists its gates. Pick the lowest tier that is honest, and say why in the task file (`Tier: T2 · reason`). The user can override.

## Tiers

| Tier | For | Examples |
|---|---|---|
| **T0 Trivial** | No behaviour change, easy to undo | Docs, copy, comment, typo, config text, rename of a local variable |
| **T1 Standard** | Small change, one repo, low blast radius | UI tweak, small bug, one screen, adding a field |
| **T2 Significant** | New feature or touches shared code, API or tracking | New page, API integration, new tracking events, refactor across files |
| **T3 Critical** | Money, identity, data, release | Payment/refund/credit logic, auth, data migration, tracking contract with backend, store/hosting release, security |

When unsure between two tiers, take the higher one. A change moves up a tier if it touches payments, auth, personal data or a contract another team depends on.

## Gates

| # | Gate | Who validates | T0 | T1 | T2 | T3 |
|---|---|---|---|---|---|---|
| G1 | **Scope check**: diff only touches the task's `Touches`; no unrelated edits | Agent | ✔ | ✔ | ✔ | ✔ |
| G2 | **Static checks**: format, lint, typecheck/`flutter analyze` as the repo defines | Agent (run the command) | – | ✔ | ✔ | ✔ |
| G3 | **Tests**: existing tests pass; new logic has tests | Agent (run) | – | existing | ✔ new | ✔ new + edge cases |
| G4 | **Run it**: see the change work (browser/device/dev backend), compare with Figma if there is a design | Agent (`run` skill / browser) | – | quick | ✔ | ✔ |
| G5 | **Code review**: correctness, quality, best practice, reuse, no shared-component edits | Separate review agent (`/code-review`, then `/simplify`) | – | – | ✔ | ✔ strict |
| G6 | **Security review** | `/security-review` | – | – | if auth/data/input | ✔ |
| G7 | **Docs sync**: API spec / architecture / backend requests updated | Agent | if docs changed | if contract touched | ✔ | ✔ |
| G8 | **PR message / commit** passes the format below | Agent | ✔ | ✔ | ✔ | ✔ |
| G9 | **Human review** of the PR | User | – | – | optional | ✔ |
| G10 | **Release plan**: rollback, flags, backend dependency, who monitors | Agent drafts, user approves | – | – | – | ✔ |
| G11 | **Changelog** entry at root | Agent | – | ✔ | ✔ | ✔ |

The validator is never the same agent run that wrote the code for G5 and G6: use a fresh agent or the review skill so it reads the diff cold.

## Gates by stage

| Stage folder | Gates run |
|---|---|
| `3-in-progress/` | G1 while building |
| `4-testing/` | G2 to G7 (as the tier requires) |
| `5-done/` | G8, G9, G11 passed, merged, worktree removed |
| `6-deploy/` | G10 executed, live and checked |

## Commit and PR message (G8)

- Commit: conventional commits, `type(scope): subject` (`feat`, `fix`, `chore`, `docs`, `refactor`, `test`). Subject in the imperative, no full stop. Mini-app and portal enforce this with commitlint.
- PR description: `Why` (one or two lines) · `What changed` (bullets) · `How tested` (the gates you ran, with results) · `Risk / rollback` (T2 and T3) · `Task:` link to the task file. State what you did not test.
- Never claim a check passed that you did not run.

## Skipping gates

Gates exist to catch real problems, not to add ceremony. Skip a gate only when it is plainly pointless for this change (for example no tests exist for the area and the change is a one-line text fix). Write `Skipped G3: <reason>` in the task file. A skipped gate is visible and reversible; a silent one is not allowed.

## Task file additions

The task file carries `Tier:` and a gate checklist. Copy only the rows the tier needs from this file.
