# Cross instant cleaning contract

- **ID:** T-0083
- **Repo:** cross
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T3 · tracking contract with backend; new tables and endpoints (request only) (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** – / 2026-11-02
- **Docs:** `Docs/tracking/v2/backend-request/App_Tracking_Backend_Handoff_2Nov2026.md` · `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Instant_Cleaning_Tracking_Spec.md`
- **Depends on:** T-0062 approved
- **Touches:** `Docs/tracking/v2/backend-request/pending-request-backend.md` §12, `Docs/tracking/v2/api/Journey_Backend_API_Spec.md`, `Docs/tracking/v2/backend-response/`

## Goal
Send request §12, agree names, tables, endpoints and the booking-mode values, record the reply.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Reply stored in `backend-response/` and API spec updated with each `/instant/*` endpoint
- [ ] Error-code list and booking-mode values confirmed
- [ ] Answered items removed from §12
