# Promo codes: list, filters, detail, pause

- **ID:** T-0015
- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Started / updated:** – / 2026-10-21
- **Tier:** T3 · coupon/price logic and use counts
- **Claimed by:** none
- **Worktree:** `bEasy/bEasy-portal-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Docs:** Figma file `8bDJcg0dDmieddfLLjkloh`: `199:115523` list, `199:121008` detail, `199:117141` pause; notes `199:120403` `199:120419` · plan: `Tasks/_plans/Portal-marketing-suite.md`
- **Depends on:** Portal-marketing-foundations-ui · backend ledger fields (`backend-request/Marketing_Suite_Backend_Request.md` §1)
- **Touches:** `app/routes/promo-code/*` (new), `app/api/` promo hooks (wrap existing `coupon/*`), locales

## Goal
Rebuild coupon list and detail to the Promo codes design: status, uses left, who can use, dates; detail with ledger (applied, redeemed, returned, lost), pushes that carry the code, pause dialog. Uses left = limit − redeemed (returned go back). Show only what the API returns; fields missing from the API show "Not available", never 0.

## Gates (T3 · coupon/price logic and use counts, from `Tasks/PIPELINE.md`)
- [ ] G1–G8  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog (G6 security review required)

## Done when
- [ ] List filters and search match the frame
- [ ] Detail shows ledger counts from the API
- [ ] Pause dialog respects permission
- [ ] Loading, empty and error states from the Figma state rules
- [ ] Compared against Figma frames `199:115523`, `199:121008`, `199:117141`
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
