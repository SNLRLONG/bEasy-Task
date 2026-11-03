# App tracking meta appsflyer

- **ID:** T-0069
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-meta-appsflyer/` · **Branch:** `feat/tracking-meta-appsflyer`
- **Tier:** T2 · extends ad-platform events; no backend contract (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §5.1, §5.2
- **Depends on:** T-0065 (same checkout/search call sites); T-0031 findings; Meta/AF consent gates unchanged
- **Touches:** `lib/app_tracking/{facebook_helper,appflyer_helper}.dart`, call sites added by T-0064/T-0065 (one line each)

## Goal
Add Meta `Search`, `Schedule`, `Contact`, `AddToCart` and AppsFlyer `af_content_view`, `af_search`, `af_add_to_cart`, `af_initiated_checkout`, `af_add_payment_info`, `af_complete_registration`, `af_login`, `af_schedule`, `af_contact`. Confirm staff exclusion applies (G-14).

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Events visible in Meta Events Manager test events and AppsFlyer dashboard (dev)
- [ ] ATT gate and staff exclusion verified
- [ ] No change to the existing 7 Meta events or to `af_purchase`
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
