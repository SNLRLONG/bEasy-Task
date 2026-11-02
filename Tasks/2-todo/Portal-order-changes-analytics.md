# Portal order changes analytics

- **ID:** T-0081
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-order-changes-analytics/` · **Branch:** `feat/order-changes-analytics`
- **Tier:** T2 · new page and API integration; shows money amounts read-only (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Cancellation_And_Reschedule_Spec.md` §4
- **Depends on:** T-0080 reply (field names; mock adapter until live); `Portal-rds-orders` landed or coordinated via `Tasks/_inbox/`
- **Touches:** new `app/routes/order/changes*.tsx`, `app/components/order-changes/*`, `app/lib/order-changes/*`, `app/routes.ts` (one route), sidebar entry via the foundations task, `app/components/journey/funnel/*after-purchase*` and `app/components/journey/customer/*orders*` (fill existing rows, order timeline), `app/types/journey.ts`, `app/locales/{journey,order}/{en,km}/`

## Goal
Build Orders → Changes (Cancellations, Reschedules, Signals tabs), the order changes timeline in order detail and the Customer Orders tab, and fill the existing After-purchase rows for cancelled and rescheduled. Show records and app signals separately; null is "—", never 0.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] KPIs, reasons, lead-time histogram, who/where, trend, lists with export and "Send to CS inbox" work against the mock adapter and the live contract
- [ ] Cancel funnel (signals) with save rate and failures, plus a data-quality tile (records vs signals, share with a recorded reason)
- [ ] `byCancelDate` / `cohort` toggle; filters per spec §3.6
- [ ] Order and Customer timeline shows every reschedule, cancellation, fee and refund with actor and channel
- [ ] Khmer and English labels from `/meta/cancellation-reasons`; responsive; no horizontal page scroll
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
