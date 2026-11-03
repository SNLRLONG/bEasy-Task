# Cross cancel reschedule contract

- **ID:** T-0080
- **Repo:** cross
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T3 · tracking contract with backend; money records (fees, refunds) (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** – / 2026-11-02
- **Docs:** `Docs/tracking/v2/backend-request/App_Tracking_Backend_Handoff_3Nov2026.md` · `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Cancellation_And_Reschedule_Spec.md` §3
- **Depends on:** T-0062 approved
- **Touches:** `Docs/tracking/v2/backend-request/pending-request-backend.md` §11 (replaces 3.3 and 3.4 when agreed), `Docs/tracking/v2/api/Journey_Backend_API_Spec.md`, `Docs/tracking/v2/backend-response/`, `Docs/tracking/v2/architecture/bEasy-tracking-architecture-guide.md` §11.1 (event params)

## Goal
Send the cancellation and reschedule contract to backend, agree table columns, optional request fields, event-log payloads, counting modes and endpoints, get dates, record the reply. If `bEasy-api/` is cloned later, check each existing field and route against code and cite `path:line`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Backend reply stored in `backend-response/` (dated) and verified against code if available
- [ ] API spec lists each new endpoint with request, response and status; `afterPurchase.rescheduled/cancelled` shapes updated
- [ ] Pending items 3.3 and 3.4 removed once answered; §11 trimmed to open items
- [ ] Agreed: reason enum, fee rules owner, who may waive, refund fields, `order_id` meaning
- [ ] Dates agreed: records and event log first, endpoints next, fee preview later
