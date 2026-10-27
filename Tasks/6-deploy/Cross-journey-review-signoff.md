# Journey review: backend verification, reconciliation and sign-off pack

- **ID:** T-0024
- **Repo:** cross (docs and QA; no code)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none (docs only, in the root repo)
- **Tier:** T0 · docs and evidence (see `PIPELINE.md`); the QA runs use real data, mask personal data
- **Claimed by:** none
- **Started / updated:** – / 2026-10-27
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` · the review docx §8 and §9 · handoff "Demo checks before sign-off"
- **Depends on:** part 1 backend deploy; part 2 and 3 need the portal tasks for final evidence
- **Touches:** `Docs/tracking/v2/{backend-response,architecture,api}/`, `Docs/tracking/v2/backend-request/pending-request-backend.md`

## Goal
Turn the backend's claims into evidence, and produce the four sign-off documents the review asks for.

## Scope
**Part 1: start now.**
- Ask backend to deploy `fix/journey-data` to dev and send one real response per changed endpoint (booking, registration, acquisition: funnel and people), plus answers to `Journey_Tech_Review_Backend_Request.md`. Store masked samples in `backend-response/`.
- If `bEasy-api/` gets cloned (read-only), cite `path:line` for each claim and note mismatches.

**Part 2: reconciliation (DQ-03, 04, 11).** For the 12 to 18 Oct window compare GA4/Firebase events, customer DB and order DB: registrations 53 vs 40; first buyers 9 vs purchased 10; orders vs buyers (11 vs 10); multi-device and multi-journey customers. Record in `architecture/Journey_Reconciliation_12-18Oct.md`.

**Part 3: sign-off pack.**
- `architecture/Journey_Metric_Definitions.md`: per stage the event, population, denominator, abandonment rule, conversion rule, attribution window, timezone. It is the source for the portal Definitions popover.
- QA matrix T01 to T12: result, evidence, owner (T01, T06, T07, T10 come from `App-journey-event-qa`).
- Issue-by-issue response: Fixed / Expected behaviour / Out of scope / Planned, with owner and ETA, for DQ-01 to DQ-11, UX-01 to UX-14, tracking and finance rules.
- Corrected-page screenshots or video with personal data masked.
- List of unresolved limitations (OTP reasons, ad data, refund amounts, assisted conversion, consent).

## Gates (T0)
- [ ] G1 scope  - [ ] G7 docs sync (if docs changed)  - [ ] G8 commit message

## Done when
- [x] Samples for every changed endpoint are in `backend-response/` and `Journey_Tech_Review_Backend_Request.md` is answered or re-asked
- [x] The 12 to 18 Oct numbers are reproduced and each gap is explained by a query or corrected
- [x] Definition sheet, QA matrix, response sheet and screenshots exist and are linked from `Docs/tracking/v2/README.md`
- [x] The eight demo checks in the handoff pass, or are listed as limitations
