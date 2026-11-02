# Portal app usage analytics

- **ID:** T-0071
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-app-usage-analytics/` · **Branch:** `feat/app-usage-analytics`
- **Tier:** T2 · new page and API integration; no auth or money (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §7
- **Depends on:** T-0070 contract (mock data until live); `Portal-marketing-foundations-ui` and T-0076 (same marketing routes), `Portal-rds-marketing` landed or coordinated via `Tasks/_inbox/`
- **Touches:** new `app/routes/marketing/app-usage*.tsx`, `app/components/app-usage/*`, `app/lib/app-usage/*` (api, query keys, store), `app/routes.ts` (one route), sidebar entry via the foundations task, `app/locales/marketing/{en,km}/`

## Goal
Build Marketing → App Usage: Overview, Screens (with side panel), Elements, Flows, Search tabs with the shared filter bar and compare toggle, plus the Customer page "Show taps" toggle. Use RDS v2 components only; do not edit shared components.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] All tabs render loading, empty, error and data states; mock adapter switchable to live
- [ ] Filters: period, platform, app version, language, auth state, source/campaign, service, audience, screen group
- [ ] Khmer and English strings
- [ ] Responsive at phone, tablet, desktop; no horizontal page scroll
- [ ] Export button calls `POST /app-usage/export`
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
