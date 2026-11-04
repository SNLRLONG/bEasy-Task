# CS Performance (CS Overview V3, follow-up queues, action plans, CRM)

- **ID:** T-0085
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-12-24
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2026-12-16 → 2026-12-24 (7 working days, order 9 of 12 · spills past November)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T2 · new views, API calls (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (scanned); KPI plan: November 2026, master task 1
- **Depends on:** backend work by Bunlong. Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
CS Overview V3, follow-up queues, customer action plans, first/repeat sales KPIs, contact outcomes, CRM integration.

## Notes
**Superseded 4 Nov 2026 (confirmed 5 Nov, work started)** by `Tasks/_plans/Portal-cs-action-performance.md` (T-0104, T-0105, T-0106). Do not start this task; ask before deleting.
Figma scanned 2 Nov 2026: section `1726:104530` (28 frames: overview states, follow-up queues, My cases, contact log, Send to CS from Customers). Overview and targets already belong to T-0058; this task is the follow-up queues, contact history, Send to CS and CRM hooks.
Continues T-0058 (`Portal-cs-overview`, due 2026-11-04); do not recount it.
November KPI task (web frontend, Seavlong). Estimate is provisional until the Figma frames are scanned.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Web frontend implemented against the agreed design and API contract
- [ ] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
