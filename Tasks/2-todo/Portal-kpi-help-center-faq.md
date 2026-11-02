# Help Center FAQ management (web)

- **ID:** T-0090
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-11-25
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2026-11-19 → 2026-11-25 (5 working days, order 5 of 10)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T2 · new feature, content CMS (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (scanned); KPI plan: November 2026, master task 7
- **Depends on:** backend work by Tom (backend), Seyha (mobile). Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
Create, edit, organize and publish English/Khmer service FAQs.

## Notes
Figma scanned 2 Nov 2026: section `687:21336` frames `691:22350` service FAQs, `693:21854` help centre; the shared content editor and review flow (`689:21441`, `691:21760`) is much larger, so only FAQ and help centre screens are counted.
Needs approved FAQ structure, publish/language rules and EN/KM content. Search and chat prep are out of scope.
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
