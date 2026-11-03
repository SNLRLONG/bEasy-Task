# Landing page: public API client with static fallback

- **ID:** T-0099
- **Repo:** bEasy-landing-page-v2
- **Owner:** SNLRLONG (Seavlong)
- **Due:** –
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Worktree:** `bEasy/bEasy-landing-page-v2-public-api-client/` (create when claimed) · **Branch:** `feat/public-api-client`
- **Tier:** T2 · new API integration (see `PIPELINE.md`)
- **Claimed by:** Claude (session 2026-11-03)
- **Docs:** [Plan](../_plans/Landing-Page-public-api-content-sync.md)
- **Depends on:** none (routes already live)
- **Touches:** `lib/public-api.ts` (new), `lib/services.ts` (id map only), `types/`, `tests/`

## Goal
Server-side fetch layer for `customer/category/list` and `customer/category/{id}/equipments`: allowlist types (no spreading API objects), `zh`→`Cn` language mapper with English then static fallback, site id → API `categoryId` map, `revalidate` caching, never throws (returns null so pages render static content). No UI change.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (3 Nov 2026)
Built in worktree `bEasy-landing-page-v2-public-api-client` (uncommitted): `lib/public-api.ts`, `SERVICE_CATEGORY_IDS` in `lib/services.ts`, tests. Checked live on prod: 7 categories, task groups in km/zh, 3 equipments for AC. ESLint clean; `tsc` shows 2 existing errors (missing `public/images/compare/*.webp` in this worktree), none in new files. Not yet run: G5 review, G6, `yarn build` with API unreachable (module is not imported by any page yet).

## Done when
- [x] Unit tests: mapper, fallback on 5xx/timeout/bad shape, language fallback (`tests/public-api.test.mjs`, 5 pass)
- [ ] `yarn build` passes with API unreachable
- [x] Only the fields in the plan are exposed; no calls to `user/`, `order/`, `auth/`
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
