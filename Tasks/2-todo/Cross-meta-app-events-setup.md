# Meta app events: setup and verification

- **ID:** T-0031
- **Repo:** cross (marketing, bEasy-app only if a gap is found)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none (worktree only if an app fix is needed)
- **Tier:** T2 · tracking events verification across marketing and app; re-tier to T3 if event names or params change
- **Claimed by:** none
- **Started / updated:** – / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-utm-meta-tracking.md` · `bEasy-app/lib/app_tracking/facebook_helper.dart`
- **Depends on:** none
- **Touches:** read `bEasy-app/lib/app_tracking/facebook_helper.dart`, `lib/controller/payment_controller.dart`, checkout and category screens; edit only if a bug is proven

## Goal
Meta receives the 7 standard app events correctly so marketing can build retargeting audiences.

## Scope
1. Confirm iOS + Android app is registered in Meta Developers and linked to the Business and ad account; app ID matches the app.
2. Events Manager > Test Events with a dev build: ViewContent, SelectService, InitiateCheckout, AddPaymentInfo, ViewBooking, CompleteRegistration, Purchase. Check `value`, `currency`, service name.
3. Check consent: iOS ATT runs before `initialize()`; decide Android behaviour when the user declines.
4. AppsFlyer dashboard: enable Meta as an integrated partner so installs are credited (October report showed 0 installs).
5. Events Manager: set Purchase and InitiateCheckout as priority events; marketing creates audiences (viewed not booked, checkout not purchased, past purchasers).
6. Confirm events also arrive in production after release.
Keep Meta standard names; do not rename to custom ones.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G3/G5 if no code changes.

## Done when
- [ ] All 7 events seen in Test Events with correct parameters (screenshots, personal data masked)
- [ ] AppsFlyer Meta partner enabled; an install is credited in a test
- [ ] Audiences created by marketing
- [ ] Production events confirmed
- [ ] Root `CHANGELOG.md` entry added (if code or shared docs changed)
