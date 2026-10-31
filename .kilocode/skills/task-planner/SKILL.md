---
name: task-planner
description: Turn a feature idea, Figma design or requirement into a finalized plan and scoped task files. Use before building anything new or unclear: reviews the UI, reads the requirements and backend docs, maps the flow, finds gaps, then breaks the work into tasks with scope, dependencies and an automatic tier. Writes to Tasks/_plans and Tasks/2-todo.
---

# Task planner

You plan; you do not build. Output is a plan file plus task files that another agent can pick up without asking questions. Rules for tasks, tiers and parallel work: root `AGENTS.md`, `Tasks/README.md`, `Tasks/PIPELINE.md`.

## 1. Gather (read, do not guess)
- **UI:** the Figma frames (`get_design_context`, `get_screenshot`; follow the Figma skills), or the screenshots given. Note every screen, state (empty, loading, error, success) and interaction.
- **Requirements:** `Docs/tracking/v2/requirements/`, the matching `api/` spec, `backend-response/`, `backend-request/`, `architecture/`, and the repo's own `CLAUDE.md`/`TODO.md`.
- **Code:** find what already exists (screens, components, stores, API hooks, events) so the plan reuses it. Check `git worktree list` and other tasks' `Touches` for overlap.

## 2. Understand the flow
Write the end-to-end flow in plain steps: who does what, which screen, which API call, which tracking event, what the backend returns, what can fail. Mark each step: **exists / change / new**.

## 3. Finalize
- List **gaps and conflicts** between design, requirements, backend status and code (missing endpoint, different field names, undefined empty state, unclear rule).
- Decide what you can by following the docs and project rules; state the decision and why. Put only true product decisions (that change scope or behaviour and cannot be settled from the docs) under **Open questions**, each with your recommended answer. Do not stop planning for them.
- Backend dependencies go to `Docs/tracking/v2/backend-request/pending-request-backend.md` (small edit, follow the sync rules), not just into the plan.

## 4. Write the plan: `Tasks/_plans/<Prefix>-<feature>.md`
Sections: Goal · Flow (with exists/change/new) · Screens and states · Data and API · Tracking events · Gaps and decisions · Open questions (with recommendation) · Task breakdown (table: task, repo, depends on, tier) · Out of scope.

## 5. Break into tasks
- One task = one worktree/branch = one reviewable PR, roughly 1 to 2 days. Split by repo, then by vertical slice (a screen with its data), not by technical layer.
- Order by dependency; mark what can run in parallel. Tasks that would touch the same files are sequenced or merged, not parallel.
- Create each from `Tasks/_TEMPLATE.md` in `Tasks/2-todo/` with: goal, `Touches` (specific paths), `Depends on`, acceptance checks ("Done when"), design/doc links, and the **auto tier** (picker in `PIPELINE.md`) with its gate rows.
- Do not claim, create worktrees or start building. That is the delivery agent's job (`project-manager` skill).

## 6. Report
Give the plan path, the task list in order, parallel groups, the tier of each, and the open questions with recommendations. Commit the new task and plan files in the root repo with specific paths.

## Principles
- Reuse before new. Respect project rules (shadcn, no shared-component edits, small files, English + Khmer).
- Smallest scope that delivers the flow. Anything extra goes to `1-backlog/`.
- Never invent design or API details; if it is not in the sources, it is a gap.
