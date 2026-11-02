# Portal app health monitor

- **ID:** T-0072
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-app-health-monitor/` · **Branch:** `feat/app-health-monitor`
- **Tier:** T2 · new tab and API integration (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §7
- **Depends on:** T-0071 shell; T-0068 data; backend `/app-usage/errors`
- **Touches:** `app/routes/marketing/app-usage*.tsx` (Health tab), `app/components/app-usage/health/*`

## Goal
Add the Health tab: errors grouped by type, endpoint and version with trend, affected users, crash-free users tile and top crashes.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Health tab shows grouped errors and trend with filters
- [ ] Drill to affected customers list uses masked identifiers
- [ ] Empty and error states; Khmer and English strings
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
