# Post week 2 SEO article set

- **ID:** T-0040
- **Repo:** bEasy-landing-page-v2
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-10-23
- **Worktree:** `bEasy/bEasy-landing-page-v2-seo-week2/`  · **Branch:** `feat/seo-week2`
- **Tier:** T0 · article copy and static assets, no logic (see `PIPELINE.md`). Re-tier up if slug maps or `lib/media.ts` logic change.
- **Claimed by:** claude-code session, 2026-10-19
- **Started / updated:** 2026-10-19 / 2026-10-27
- **Docs:** `bEasy-landing-page-v2/IMAGE_SEO_GUIDE.md`, `TRANSLATIONS.md`, `EVENTS.md`
- **Depends on:** none (content and banners were already in the repo)
- **Touches:** `app/translations/{English,Khmer}/media.json` (remove `draft`, set date), two banner images; `lib/media.ts` slug already locked

## Goal
Publish the week 2 SEO article on the landing page on Friday 23 Oct 2026 (moved from 20 Oct, confirmed with the marketing team), in km and en.

## Article (week 2 = media id 11, was `draft: true`, now released in the worktree)
- **EN:** Instant Cleaning vs Deep Cleaning: Which Do You Need?
- **KM:** សេវាសម្អាតរហ័ស និង សេវាសម្អាតស៊ីជម្រៅ៖ តើអ្នកត្រូវការមួយណា?
- **Slug (locked):** `instant-cleaning-vs-deep-cleaning` · content `data/guides/instant-vs-deep.ts`
- **Banners:** `/images/media/instant-vs-deep-cleaning.webp` (en), `-km.webp` (km)
- Week 1 (id 10, "How Much Does House Cleaning Cost in Phnom Penh?") is already live; the rest of the series is weekly through 28 Dec (ids 12 to 21), all still drafts.

## Done in the worktree (commit on `feat/seo-week2`, not pushed)
- Removed `draft`, set `fullBanner: true`, date and sortDate 23 Oct 2026 (uncommitted edit in the worktree on top of commit 7351a01, which used 19 Oct) (Khmer date ២៣ តុលា ២០២៦) in both `media.json`.
- Banners `instant-vs-deep-cleaning.webp` and `instant-vs-deep-cleaning-km.webp` (renamed from the untracked "Instant Cleaning vs Deep Cleaning - En/Km.webp" in the main checkout) in `public/images/media/`, same pattern as week 1.

## Status 2026-10-19
- Worktree edit verified: both `media.json` parse, id 11 has no `draft`, date and sortDate 23 Oct 2026 (km date ២៣ តុលា ២០២៦), slug in `lib/media.ts:20`, both banners present. Uncommitted, waiting for the developer to commit.
- **Warning:** `main` already contains PR #119 (53825dc, "Feat/seo week2"), which has the article **undraft and dated 19 Oct**. Any deploy of `main` now publishes it before 23 Oct. The 23 Oct date fix on `feat/seo-week2` must be merged before the next deploy, or the article goes live early with the wrong date.

## Gates (T0, from `Tasks/PIPELINE.md`)
- [x] G1 scope  - [ ] G7 docs sync (if docs changed)  - [x] G8 message

## Done when
- [x] Article id 11 merged to main (#120), dated 23 Oct, km and en: `draft` removed, date 23 Oct 2026, appears in sitemap, metadata and image alt OK
- [ ] Deployed on 23 Oct, not earlier: removing `draft` makes it live on deploy, so keep the branch unmerged or undeployed until then; checked on the live site after deploy
- [x] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge
