# Customer Journey and Tracking for new features

- **ID:** T-0094
- **Repo:** bEasy-portal + backend
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-11-06
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2026-11-04 → 2026-11-06 (3 working days, order 2 of 10)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T3 · tracking contract with backend (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (no frames for this task); KPI plan: November 2026, master task 11
- **Depends on:** backend work by Bunlong. Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
Confirm events and tracking exist for each new feature.

## Notes
Figma scanned 2 Nov 2026: No frames for this task in the file (pages `1:2` UI | handoff and `1:4` Design were scanned). Estimate is unchanged from the KPI plan; ask for the design.
Covers vouchers, push, chat, cancellations. Follows `Tasks/_plans/Cross-tracking-coverage.md`.
November KPI task (web frontend, Seavlong). Estimate is provisional until the Figma frames are scanned.

## Gates (T3, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Web frontend implemented against the agreed design and API contract
- [ ] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
