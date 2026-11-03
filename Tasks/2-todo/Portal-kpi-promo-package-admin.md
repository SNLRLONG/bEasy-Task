# Promo and Package Deal admin

- **ID:** T-0086
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2027-01-18
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2027-01-06 → 2027-01-18 (9 working days, order 11 of 12 · spills past November)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T3 · discount, refund and usage-limit rules (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (scanned); KPI plan: November 2026, master task 2
- **Depends on:** backend work by Bunlong. Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
Promotion creation, eligibility, usage limits, package setup, redemption, refunds, performance reporting.

## Notes
Figma scanned 2 Nov 2026: section `661:20253` (v2 frames: promo create `848:174`, detail `848:46392`, package list `850:2`, create `850:46771`, detail `852:582`, purchase detail `852:46592`). Compensation frames `861:47262`, `856:190` are marked not started.
Builds on `Portal-promo-codes-list-detail` and `Portal-promo-codes-create`; count only the new package, redemption, refund and reporting screens.
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
