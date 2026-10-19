# Post week 2 SEO article set

- **Repo:** bEasy-landing-page-v2
- **Owner:** Suntel
- **Due:** 2026-10-20
- **Worktree:** `bEasy/bEasy-landing-page-v2-seo-week2/`  · **Branch:** `feat/seo-week2`
- **Tier:** T0 · article copy and static assets, no logic (see `PIPELINE.md`). Re-tier up if slug maps or `lib/media.ts` logic change.
- **Claimed by:** claude-code session, 2026-10-19
- **Started / updated:** 2026-10-19 / 2026-10-19
- **Docs:** `bEasy-landing-page-v2/IMAGE_SEO_GUIDE.md`, `TRANSLATIONS.md`, `EVENTS.md`
- **Depends on:** none (content and banners were already in the repo)
- **Touches:** `app/translations/{English,Khmer}/media.json` (remove `draft`, set date), two banner images; `lib/media.ts` slug already locked

## Goal
Publish the week 2 SEO article on the landing page on 20 Oct 2026, in km and en.

## Article (week 2 = media id 11, was `draft: true`, now released in the worktree)
- **EN:** Instant Cleaning vs Deep Cleaning: Which Do You Need?
- **KM:** សេវាសម្អាតរហ័ស និង សេវាសម្អាតស៊ីជម្រៅ៖ តើអ្នកត្រូវការមួយណា?
- **Slug (locked):** `instant-cleaning-vs-deep-cleaning` · content `data/guides/instant-vs-deep.ts`
- **Banners:** `/images/media/instant-vs-deep-cleaning.webp` (en), `-km.webp` (km)
- Week 1 (id 10, "How Much Does House Cleaning Cost in Phnom Penh?") is already live; the rest of the series is weekly through 28 Dec (ids 12 to 21), all still drafts.

## Done in the worktree (commit on `feat/seo-week2`, not pushed)
- Removed `draft`, set `fullBanner: true`, date and sortDate 19 Oct 2026 (Khmer date ១៩ តុលា ២០២៦) in both `media.json`.
- Banners `instant-vs-deep-cleaning.webp` and `instant-vs-deep-cleaning-km.webp` (renamed from the untracked "Instant Cleaning vs Deep Cleaning - En/Km.webp" in the main checkout) in `public/images/media/`, same pattern as week 1.

## Gates (T0, from `Tasks/PIPELINE.md`)
- [x] G1 scope  - [ ] G7 docs sync (if docs changed)  - [x] G8 message

## Done when
- [ ] Article id 11 live in km and en: `draft` removed, date 19 Oct 2026, appears in sitemap, metadata and image alt OK
- [ ] Checked on the deployed site (or preview channel) before 2026-10-20
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
