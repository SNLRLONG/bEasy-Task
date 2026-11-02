# Landing Page: Enable Simplified Chinese again

- **Repo:** bEasy-landing-page-v2
- **Owner:** SNLRLONG
- **Worktree:** none (working directly on `main`, same as T-0045)
- **Tier:** T2 · locale routing, middleware, redirects, sitemap and hreflang for a third language; 20+ translation files (see `PIPELINE.md`)
- **Claimed by:** claude-code session, 2026-11-02
- **Started / updated:** 2026-11-02 / 2026-11-02
- **Docs:** `bEasy-landing-page-v2/TRANSLATIONS.md`; removal commit `4cb67bc` ("remove zh from entire web", 3 Oct 2026)
- **Depends on:** none (follows T-0045, which changed the English and Khmer copy)
- **Touches:** `middleware.ts`, `next.config.ts`, `lib/seo.ts`, `lib/services.ts`, `app/[lang]/**`, `app/contexts/LanguageContext.tsx`, `app/components/**` (NavBar, InstallAppSection, RelatedLinks), `data/guides/*`, `app/translations/Chinese/*`, `app/translations/Khmer/serviceDetail.json`

## Goal
Bring `/zh` back with full parity: same services, claims and guides as English and Khmer, no retired services (General Cleaning, Pest Control).

## Done
- [x] Re-enabled `zh` in `LOCALES`, `middleware.ts` (removed the `/zh` redirect), `generateStaticParams`, language switcher, NavBar, install mock-up, OG locale, redirect regexes in `next.config.ts`
- [x] Chinese copy mirrors the T-0045 fixes: removed General Cleaning and Pest Control (service list, nav, footer, FAQ, about, article 1), softened claims (reschedule, cleaner choice, 90-100% odour), rewrote `serviceDetail` entries 2, 3, 4, 6, 7, 9 (Office custom quote, washing-machine non-invasive scope, upholstery exclusions, Instant, Deep, Post-Renovation), AC header (was Traditional Chinese with a translator note), `sectionA` alt text
- [x] Added the 12 SEO guides (ids 10-21) to Chinese `media.json` and a `zh` block to each `data/guides/*.ts`; guides fall back to English if a `zh` block is missing
- [x] zh metadata: home, FAQ crumbs and JSON-LD, service FAQ JSON-LD, Instant Cleaning snippet, Related links
- [x] Khmer gap found on the way: Khmer `serviceDetail.json` entries 2, 3, 4, 6, 7, 9 still had the pre-audit text (residential Office copy, washing-machine dismantling, upholstery guarantees). Fixed with the same changes as Chinese
- [x] `TRANSLATIONS.md` already lists Chinese; root `CLAUDE.md` line about "only Khmer and English" updated

## Gates
- [x] G1 scope  - [x] G2 static (`yarn build`, eslint on changed files)  - [ ] G3 tests (none exist for this repo)
- [x] G4 run it: built HTML for `/zh` (home, services, FAQ, about, contact, media, guides) has no retired services; sitemap and hreflang include zh
- [x] G5 code review (2 Nov; fixed: zh keywords, sitemap zh images, `zh` guide block now required by the type)  - [x] G6 skipped: no auth, data or input handling  - [x] G7 `CLAUDE.md` updated; no API or tracking change
- [x] G8 commit  - [x] G9 human review (user)  - G10 n/a for T2  - [x] G11 changelog

## Known gaps (read before deploy)
- Chinese text is machine-written Simplified Chinese; it needs a native read, especially the 12 guides and the service pages.
- Guide article images are the English banners (the guides have Khmer banners only), so the text in the banner image is English on `/zh`.
- Chinese `serviceDetailAbout` entries other than the AC header were not re-checked against English (English only changed entry 5 and 8 in T-0045).
- The washing-machine guide (id 17) says a professional clean "typically involves opening up the machine". It is the same in English and Khmer and conflicts with the non-invasive wording on the service page. Left as is; decide whether to soften it in all three languages.
- `/zh` URLs were redirected to `/km` since 3 Oct; Google may still treat them as redirects for a while. Resubmit the sitemap.
- Not fixed from review (low): `serviceDetailAboutMap` in the service page still maps km and zh to the English about entry (only used for the JSON-LD about block); no explicit CJK font stack (browser falls back to the system font); Khmer `media.json` has older structure differences from English in articles 2 and 6 (not from this task).
- Washing Machine cards fixed (2 Nov): Top Load $25, Front Load $35 (from the app screenshot), both use `wash_machine.webp`. Swap in real photos later if wanted.

## Deployed (2026-11-02)
Committed, pushed and deployed by the user. Still to do by hand: resubmit the sitemap in Search Console and spot-check the redirects on beasy.info.
