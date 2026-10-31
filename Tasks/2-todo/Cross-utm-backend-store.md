# Backend: store UTM fields and group Acquisition by them

- **ID:** T-0035
- **Repo:** cross (request to backend; bEasy-api is read-only)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T3 · tracking contract with backend, customer attribution data
- **Claimed by:** none
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-utm-meta-tracking.md` · `Docs/tracking/v2/backend-request/pending-request-backend.md` item 7 · `Campaign_Source_Request.md`
- **Depends on:** T-0033
- **Touches:** `Docs/tracking/v2/backend-request/`, `api/Journey_Backend_API_Spec.md`

## Goal
Backend saves the new UTM fields on the customer attribution touch and on app opens, and returns them in Acquisition.

## Scope
1. Check `bEasy-api/` (read-only) for what exists; cite `path:line` in the request.
2. Request is in `pending-request-backend.md` item 7.
3. After the reply: add it to `backend-response/`, verify against code, update API spec, delete the answered item.

## Gates
- [ ] G1 scope  - [ ] G7 docs sync  - [ ] G8 commit message  - [ ] G9 human review

## Done when
- [ ] Backend replied and fields live on dev
- [ ] One real `GET /funnels/acquisition` sample shows `utm_id` and content
- [ ] Docs synced (request removed, spec updated, reply filed)
- [ ] Root `CHANGELOG.md` entry added (if code or shared docs changed)
