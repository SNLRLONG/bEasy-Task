# Portal location insights

- **ID:** T-0078
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-location-insights/` · **Branch:** `feat/location-insights`
- **Tier:** T2 · new tab and API integration; no auth or money (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md` §C.3
- **Depends on:** T-0071 App Usage shell; T-0077 data; backend `GET /app-usage/locations`
- **Touches:** `app/routes/marketing/app-usage*.tsx` (Locations tab), `app/components/app-usage/locations/*`, `app/locales/marketing/{en,km}/`

## Goal
Add the Locations tab: tiles (address adds, out-of-range rate, permission denied rate), map and table of out-of-range cells (demand outside coverage), add-address method breakdown, add-address success funnel, with the standard filters.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Tab renders loading, empty, error and data states; Khmer and English strings
- [ ] Map shows cell centres only (no exact addresses); table sortable and exportable
- [ ] Filters work: period, platform, app version, language, auth state, source/campaign
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
