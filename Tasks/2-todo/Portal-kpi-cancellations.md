# Cancellations and rescheduling admin

- **ID:** T-0092
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-12-15
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2026-12-08 → 2026-12-15 (6 working days, order 8 of 12 · spills past November)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T3 · fees, refunds, deposits (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (scanned); KPI plan: November 2026, master task 9
- **Depends on:** backend work by Tom (backend), Seyha (mobile). Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
Eligibility and reschedule views, fee/deposit/refund display, exception overrides and history.

## Notes
Figma scanned 2 Nov 2026: section `11:343` cancel and reschedule frames (about 20 current: confirm, fee, reason list, refund pending/failed/unknown/manual, reschedule fee and link-expired, disabled states). Superseded V3/V3.5 frames are ignored.
Reuse existing direct-sale refund handling; count only November gaps. Needs approved deadlines, charges, refund policy, permissions.
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
