# Portal instant cleaning insights

- **ID:** T-0084
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-instant-cleaning-insights/` · **Branch:** `feat/instant-cleaning-insights`
- **Tier:** T2 · new tab and API integration; no auth or money (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Instant_Cleaning_Tracking_Spec.md`
- **Depends on:** T-0083 reply (mock first); T-0071 App Usage shell
- **Touches:** `app/routes/marketing/app-usage*.tsx` (Instant tab), `app/components/app-usage/instant/*`, Booking mode filter in `app/components/journey/*` filter bars (wrapper only), `app/locales/marketing/{en,km}/`

## Goal
Add the Instant Cleaning tab (funnel, unmet demand, rooms, schedule errors, upsell, live orders) and a Booking mode filter chip on Customer Journey, Orders and After-purchase.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] All views render loading, empty, error and data states against mock and live contract
- [ ] Booking mode filter works on funnel, orders and After-purchase
- [ ] Khmer and English strings; responsive; no horizontal page scroll
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
