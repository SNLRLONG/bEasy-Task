# Portal: show UTM ID and content in Acquisition

- **ID:** T-0036
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-utm-acquisition/` · **Branch:** `feat/utm-acquisition`
- **Tier:** T2 · new API fields and filters on an existing screen
- **Claimed by:** none
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-utm-meta-tracking.md` · `Docs/tracking/v2/api/Journey_Backend_API_Spec.md`
- **Depends on:** T-0035 (real sample from dev)
- **Touches:** `app/components/journey/` campaign table and filters, `app/locales/journey/{en,km}/`

## Goal
Campaign table and filters show UTM ID and content per placement.

## Scope
Add columns/filter for UTM ID and content, EN + KM, using the agreed field names; empty state when a touch has none.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Table and filter show real dev data
- [ ] EN and KM strings added
- [ ] Root `CHANGELOG.md` entry added (if code or shared docs changed)
