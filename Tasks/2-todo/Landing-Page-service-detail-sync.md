# Landing page: service tasks and equipment from the app

- **ID:** T-0101
- **Repo:** bEasy-landing-page-v2
- **Owner:** SNLRLONG (Seavlong)
- **Due:** –
- **Started / updated:** – / 2026-11-03
- **Worktree:** `bEasy/bEasy-landing-page-v2-service-detail-sync/` (create when claimed) · **Branch:** `feat/service-detail-sync`
- **Tier:** T2 · new block, API integration (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** [Plan](../_plans/Landing-Page-public-api-content-sync.md)
- **Depends on:** T-0099, T-0100 (same files); design OK for the equipment block
- **Touches:** `app/[lang]/services/[slug]/page.tsx`, service detail components, `app/translations/*/serviceDetail.json` (fallback only)

## Goal
Banner/hero is **not** touched and does not use API data (owner decision 3 Nov 2026); the service is only matched to its category via the id map.

Render the 'what is included' list from `taskInfoXx` and add a 'Tools we bring' strip from `equipments`, in the existing layout and card style. Hero, intro, FAQ, prices and OneLinks stay static; static task list is the fallback.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] km, en, zh render API text; missing language falls back to English, then static
- [ ] Equipment images load via allowed `next/image` hosts (`storage.googleapis.com`, `api.beasy.info`)
- [ ] Compared with the approved design for the new block
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
