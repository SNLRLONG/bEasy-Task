# Cross registration funnel stages

- **ID:** T-0075
- **Repo:** cross
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T3 · tracking contract with backend; funnel definition (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-02
- **Started / updated:** – / 2026-11-02
- **Docs:** `Docs/tracking/v2/backend-request/App_Tracking_Backend_Handoff_3Nov2026.md` · `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md` §A, B
- **Depends on:** T-0062 approved (and decision D-6)
- **Touches:** `Docs/tracking/v2/backend-request/pending-request-backend.md` §10, `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` (stage list, `/funnels/registration`), `Docs/tracking/v2/requirements/{Journey_Tracking_UI_Flow_v2.1,Journey_Tracking_Flow_Breakdown}.md` (UC-07, §3.2), `Docs/tracking/v2/architecture/bEasy-tracking-architecture-guide.md` §4.4/§11.1

## Goal
Fix the docs to the real registration order, send backend request §10, agree stage order and entry fields, and record the reply. The old order stays in `archive/`-style notes only where history matters (no history logs in docs).

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] UC-07, UI flow §3.2 and API spec list the new stage order and the `entry` taxonomy
- [ ] Backend reply stored in `backend-response/`; request items answered are removed from §10
- [ ] Date agreed for backend stage reorder (items 1 and 3) and for app release alignment
- [ ] Confirmed with backend: stage 4 comes from `otp_verified`, not inferred
