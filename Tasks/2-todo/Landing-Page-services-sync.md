# Landing page: services list driven by the app

- **ID:** T-0100
- **Repo:** bEasy-landing-page-v2
- **Owner:** SNLRLONG (Seavlong)
- **Due:** –
- **Started / updated:** – / 2026-11-03
- **Worktree:** `bEasy/bEasy-landing-page-v2-services-sync/` (create when claimed) · **Branch:** `feat/services-sync`
- **Tier:** T2 · feeds nav, sitemap, JSON-LD (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** [Plan](../_plans/Landing-Page-public-api-content-sync.md)
- **Depends on:** T-0099
- **Touches:** `app/components/sections/ServiceSection.tsx`, navbar/footer service links, `app/sitemap.ts`, `lib/services.ts`, `lib/jsonld.ts`

## Goal
Show a service only when the API returns its category, `isComingSoon` is false, and the static slug allow-list also includes it. Mapping only: the API says which category is live; name, order, icon, banner/hero and all copy stay static (owner decision 3 Nov 2026) - slugs, copy, hero, keywords stay static. Sitemap and JSON-LD follow the same list. URLs and redirects unchanged. Run before T-0101 (same files).

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Disabling a category in the API hides it from home, nav, footer, sitemap within the revalidate window
- [ ] API down: output identical to today
- [ ] Pest Control stays hidden (static gate wins)
- [ ] `EVENTS.md` unaffected or updated
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
