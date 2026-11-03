# Cross tracking backend ingest

- **ID:** T-0070
- **Repo:** cross
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T3 · tracking contract with backend; new tables and endpoints (request only) (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** – / 2026-11-02
- **Docs:** `Docs/tracking/v2/backend-request/App_Tracking_Backend_Handoff_3Nov2026.md` · `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §6
- **Depends on:** T-0062
- **Touches:** `Docs/tracking/v2/backend-request/pending-request-backend.md`, `Docs/tracking/v2/api/Journey_Backend_API_Spec.md`, `Docs/tracking/v2/backend-response/`

## Goal
Send `pending-request-backend.md` §9, agree table and endpoint contract with backend, get a staging date and a sample response per endpoint, record the reply in `backend-response/` and mark live status in the API spec. If `bEasy-api/` is cloned later, cite `path:line` for each claim.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Backend reply stored in `backend-response/` (dated)
- [ ] API spec lists each `/app-usage/*` endpoint with request, response and status
- [ ] Answered items removed from `pending-request-backend.md`
- [ ] Retention and volume estimate recorded
