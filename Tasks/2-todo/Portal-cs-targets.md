# Portal: monthly targets per CS, set by management

- **ID:** T-0107
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Worktree:** `bEasy/bEasy-portal-cs-action/` (same worktree and branch as T-0104 to T-0106) · **Branch:** `feat/cs-action`
- **Tier:** T3 · management-only write, staff performance data, permissions (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-05
- **Docs:** Plan `Tasks/_plans/Portal-cs-action-performance.md` (decision log) · backend contract `Docs/tracking/v2/backend-request/CS_Action_Performance_Backend_Request.md` section D5
- **Depends on:** T-0106 (Performance page); backend D5 for the real API. Build and review on sample data first, as with T-0104 to T-0106.
- **Touches:** `app/components/cs-performance/cs-targets-*.tsx`, `app/components/cs-performance/cs-performance-sections.tsx` (activity table columns), `app/hooks/query/use-cs-action-query.ts`, `app/hooks/mutations/use-cs-action-mutation.ts`, `app/api/cs-action.ts`, `app/api/cs-action-mock.ts`, `app/api/endpoint.ts`, `app/constants/query-key-enum.ts`, `app/types/cs-action.ts`, `tests/cs-targets*.test.mjs`

## Goal
Management sets a monthly target for each CS employee and sees it beside the actual numbers on the Performance page (tab D, Contact activity table). Plain numbers only: no weighted score, no ranking.

## Scope (simple version)
- **Two targets per employee per month:** `reached` (whole number) and `completionPercent` (0 to 100). Sales targets are added later when sales attribution is approved.
- **Contact activity table:** next to Reached and Completion show the target and a status: **Met** (actual at or above target, equal counts), **Under**, or **No target**. A target of 0 is valid and always met. Never show a missing target as 0.
- **Set targets (management only, `CS_PERFORMANCE` UPDATE):** a button above the table opens a side sheet: month picker, one row per CS employee with the two fields, Save. Empty field clears a target. Past months are read-only (backend returns 422). Validation message shown inline.
- **CS employees** do not see other people's targets (backend gate).
- Targets are monthly: for a custom range or all-time the status shows "Monthly targets only".

## Gates (T3, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Targets save per employee and month (sample data now, real API after backend D5); 403 and 422 handled
- [ ] Table shows target and Met / Under / No target; equal meets; 0 is a valid target; null is "No target"
- [ ] Only users with `CS_PERFORMANCE` UPDATE see Set targets
- [ ] Tests for the status rule (equal, zero, null, range)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
