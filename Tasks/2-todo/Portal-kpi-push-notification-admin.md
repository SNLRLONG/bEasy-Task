# Push Notification admin

- **ID:** T-0087
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2027-02-03
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2027-01-19 → 2027-02-03 (12 working days, order 12 of 12 · spills past November)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T2 · new feature, new API (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (scanned); KPI plan: November 2026, master task 3
- **Depends on:** backend work by Bunlong. Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
Promotion and package push messages, saved CRM group targeting, English/Khmer messages, scheduling, analytics.

## Notes
Figma scanned 2 Nov 2026: section `577:16028` (about 45 frames: campaigns list, 8-step promo push wizard, package and general push, automations, 4 analytics tabs, templates, send-test states). Largest design in the file.
Needs saved CRM customer groups (task 1) for targeting.
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
