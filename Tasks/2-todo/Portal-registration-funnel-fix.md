# Portal registration funnel fix

- **ID:** T-0076
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-registration-funnel-fix/` · **Branch:** `feat/registration-funnel-fix`
- **Tier:** T2 · portal funnel page change; API integration; no auth or money (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md` §A.7, B
- **Depends on:** T-0075 (backend reply, field names); `Portal-rds-marketing` landed or coordinated through `Tasks/_inbox/`
- **Touches:** `app/lib/journey/stages.ts`, `app/types/journey.ts`, `app/components/journey/funnel/{registration-funnel-tab,stage-icon,otp-reasons-card}.tsx`, `app/components/journey/list/*` (Entry column/filter), `app/components/journey/customer/{registration-summary-card,journey-tab,event-log-group-card}.tsx`, `app/lib/journey/customer.ts`, `app/locales/journey/{en,km}/{funnel,shared,customer}.json`

## Goal
Make the Registration funnel match the app: reorder stages (spec A.7), correct insight and label text in English and Khmer, add the Registration entry card (bars per entry, Checkout grouped with drill-down), the Entry column and filter on lists, and entry plus OTP mode on the customer registration card. Reuse the existing funnel and list components; do not edit shared RDS components.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Stages render in the new order in funnel, stage sheet, customer timeline and tips
- [ ] Insight texts correct in EN and KM (no 'verified the code but never started sign-up')
- [ ] Entry card, filter and column work; clicking a bar opens the matching people list
- [ ] Works against the backend's reply; mock adapter for the new fields until live
- [ ] Responsive; no horizontal page scroll
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
