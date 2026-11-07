# Public API backend contract for the landing page

- **ID:** T-0098
- **Repo:** Cross (backend requests, no code)
- **Owner:** SNLRLONG (Seavlong)
- **Due:** –
- **Started / updated:** – / 2026-11-03
- **Worktree:** `bEasy/n/a (docs only)/` (create when claimed) · **Branch:** `feat/public-api-contract`
- **Tier:** T3 · backend change, exposed user data, API contract (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** [Plan](../_plans/Landing-Page-public-api-content-sync.md)
- **Depends on:** none
- **Touches:** `Docs/tracking/v2/backend-request/pending-request-backend.md`, `Docs/tracking/v2/api/`

## Goal
Get backend to (A) confirm `customer/user/info` and `customer/order/v5/list` answering 200 for anonymous callers is the intended shared guest user, and trim guest fields (`deviceId`, `fcmToken`), (B) confirm the **Use** routes in the plan are intentionally public and stable, (C) add a public `startingPrice` per category, a slim public version shape, and cache headers.

## Gates (T3, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Item A (guest response) sent to backend and acknowledged
- [ ] Replies recorded in `backend-response/`, API spec updated (sync rules in `AGENTS.md`)
- [ ] Verified against real dev responses, mismatches noted
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
