# Chat: Unified Inbox, Telegram/Meta, auto replies

- **ID:** T-0091
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-12-24
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2026-12-15 → 2026-12-24 (8 working days, order 8 of 10 · spills past November)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T2 · new feature, integrations (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (no frames for this task); KPI plan: November 2026, master task 8
- **Depends on:** backend work by Bunlong. Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
Unified Inbox, Telegram/Meta integration, rule-based auto replies, guest enquiries, CS takeover.

## Notes
Figma scanned 2 Nov 2026: No frames for this task in the file (pages `1:2` UI | handoff and `1:4` Design were scanned). Estimate is unchanged from the KPI plan; ask for the design.
Large scope; split further once the Figma is scanned.
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
