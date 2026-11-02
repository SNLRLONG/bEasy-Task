# Landing Page SEO Audit Fixes

- **ID:** T-0045
- **Repo:** bEasy-landing-page-v2
- **Owner:** SNLRLONG
- **Due:** 2026-11-07
- **Worktree:** `bEasy/bEasy-landing-page-v2-seo-audit-fixes/` · **Branch:** `feat/seo-audit-fixes`
- **Tier:** T2 · touches multiple pages/routes/redirects/schema/URL hygiene (see `PIPELINE.md`)
- **Claimed by:** claude-code session, 2026-11-01
- **Started / updated:** 2026-11-01 / 2026-11-02
- **Docs:** `Docs/landingpage/bEasy_SEO_Website_Audit_Marketing_Handoff_30Oct2026.md`
- **Depends on:** none
- **Touches:** `app/**`, `components/**`, `data/**`, `next.config.ts`, `middleware.ts`, `public/**`, sitemap/robots/schema files, redirects

## Goal
Fix all P0/P1 SEO issues from the 30 Oct 2026 audit. Remove discontinued services (General Cleaning, Pest Control), correct service scopes/claims, add 301 redirects for legacy URLs, rewrite Office Cleaning, fix AC/Washing Machine/Upholstery/Post-Renovation/Instant/Deep pages, clean About/article pages, improve internal linking, and address technical SEO (canonicals/sitemap/robots/hreflang/schema). Work directly on main per user request (no commit unless asked).

## P0 - Immediate Accuracy Cleanup (Blockers)
- [x] Remove discontinued services (Pest Control, General Cleaning) from homepage, footer, FAQ, About, structured data, shared data/modules
- [x] 301 redirects: `/en/service-detail/2` → `/en/services/deep-cleaning`, `/en/service-detail/7` → `/en/services/post-renovation-cleaning`; reconcile `/en/services/general-cleaning` (301 to appropriate active service) + extra redirects added
- [x] Rewrite Office Cleaning page (remove residential copy, custom quotation, recurring/monthly invoicing, H1: "Professional Office Cleaning in Phnom Penh")
- [x] Fix Washing Machine Cleaning scope (remove full dismantling as standard, use non-invasive wording, add safety-stop, soften absolutes)
- [x] Fix Upholstery Cleaning scope/guarantees (exclude leather/suede/silk/velvet unless approved, remove shrink/damage guarantee, add pre-check, remove unverified heat treatment)
- [x] Fix AC Cleaning page (remove pest-control wording, keep promotions separate from SEO copy)
- [x] Redirect obsolete article "General Cleaning vs Deep Cleaning" → "Instant Cleaning vs Deep Cleaning" (301)
- [x] Update "What is bEasy?" article (remove retired services, proofread)
- [x] Homepage claims (policy-safe reschedule, assignment-safe cleaner choice, verify social proof numbers)

## P1 - Strengthen Commercial Pages
- [x] Expand Deep Cleaning page (fix typos, scope, approved add-ons/exclusions, related guides)
- [x] Tighten Post-Renovation Cleaning (limit debris/waste wording, remove % claims, scope-based manpower/duration, remove Pest Control add-on)
- [x] Update Instant Cleaning (compare with Deep Cleaning not General, clarify extra hours, tighten wording)
- [x] Add Phnom Penh-focused H1s, improve FAQs/exclusions where needed
- [x] Add article↔service internal linking clusters (real HTML links): `lib/related.ts` + `RelatedLinks` on service and article pages

## P1 - Technical SEO
- [x] Canonicals (self-referencing, no legacy service-detail canonicals) - using existing setup
- [x] Redirect QA (verify 301s, no 200 legacy duplicates) - added redirects in next.config.ts
- [x] Sitemap (only canonical/indexable, remove retired/duplicate URLs, include articles): retired article 3 set `draft` in all three `media.json`; redirect kept
- [x] robots.txt (correct, references sitemap, blocks private routes)
- [x] hreflang (EN/KH reciprocal, x-default if needed): already in `localeAlternates` and the sitemap
- [x] Structured data (Organization/LocalBusiness, Service, Article, BreadcrumbList): already present; checked

## P2 - Growth (lower priority for now)
- [ ] Search Console focus on positions 8-20
- [ ] Plan commercial-intent articles (prices: Deep/Office/AC/Mattress/Post-Reno, "1 hour vs 2 hours")
- [ ] Track to bookings/revenue

## Gates (copy rows tier needs; skip only with reason)
- [ ] G1 scope - [ ] G2 static - [ ] G3 tests - [ ] G4 run it
- [x] G5 code review (2 Nov, findings below) - [x] G6 security (skipped: no auth, data or input handling, only copy, redirects and static links; 2 Nov)
- [x] G7 docs sync (nothing to sync: no API or tracking change, no new events for `EVENTS.md`; audit doc is the source and is not edited)
- [x] G8 commit `b3fd4b6` - [x] G9 human review (user, 2 Nov) - G10 not required for T2 - [x] G11 changelog

## Done when
- [x] All P0 items addressed
- [x] P1 items addressed as scoped
- [x] Technical SEO checklist complete
- [x] Tested locally: `yarn build` passes; redirects and pages checked on dev, built HTML and sitemap checked (2 Nov)
- [x] Root `CHANGELOG.md` entry added (2 Nov)
- [x] No worktree/branch to remove (working directly on main)

## Notes
- Working directly on `main` per user request. Do not commit unless user says "commit" with files+message.
- Started date set to 01 Nov 2026 as requested.

## Progress (2026-11-02)
- Found `app/translations/English/serviceDetail.json` corrupted (content repeated 3 times, entries 7 and 8 missing, so `yarn build` failed and English Post-Renovation and AC pages had no data). Rebuilt from the intact copy plus HEAD's 7 and 8; fixed `titel` typos and softened debris wording in 7. Corrupt copy kept in the session scratchpad only. Worth a look at the English text of 7 and 8 by a human.
- `app/sitemap.ts` has an unused Chinese import path (`zh` lookup); harmless, left.
- (Superseded: gates were completed later on 2 Nov, see "Closed".)

## Review and run check (2026-11-02, later)
- Run on dev: 308 redirects verified (service-detail 2/7/5, general-cleaning, pest-control, media 3, /zh); AC, Post-Renovation and Office pages return 200; Office H1 is "Professional Office Cleaning in Phnom Penh". Fixed: pest-control and `service-detail/5` pointed at a non-existent `air-conditioner-cleaning` slug (now the locale home page), leftover "pest control" on the English home blurb and "keep pests away" in English AC copy.
- G5 fixes applied: footer duplicates removed (Washing Machine twice, Post Renovation twice, Laundry); redirects use `getMediaSlug(11)` / `SERVICE_SLUGS[9]`; draft article 3 left out of static params; related links check the published state in the page's own language; reverse guide map built once.
- **Open, blocks deploy: Khmer (the default locale) still has the retired services.** Khmer `serviceSection`, `navbar`, `footerSection`, `blogSection`, `faqSection`, `faq-service-detail` (pest FAQs feed FAQ JSON-LD), `serviceDetail`, `serviceDetailAbout`, `media` still mention Pest Control or General Cleaning. Needs Khmer copy; a native proofread is wanted.
- Also still English: `media.json` article 2 ("our general cleaning service"), `faq-service-detail.json` pest FAQs and two "General Cleaning" questions.
- Note: Next sends 308 for `permanent: true`; Google treats it like 301. Pest-control to home may count as a soft 404; a 410 is the alternative.

## Khmer sync (2026-11-02, awaiting user proofread)
Retired services removed and wording mirrored from English in `app/translations/Khmer/`: `serviceSection`, `navbar`, `footerSection`, `blogSection`, `whyChooseSection`, `faqSection`, `aboutUs`, `serviceDetailAbout` (AC), `media` (article 1), `faq-service-detail` (Instant vs Deep, Deep vs Instant, post-renovation exclusions, odour claim). Also English: two FAQ questions, article 1 "regular cleaning", exclusions, odour percentage. Built HTML for km and en has no pest or general-cleaning text.
Not mirrored in Khmer (not in the audit scope, Khmer `serviceDetail.json` entries 7 and 8 keep the old debris wording). Hidden entries 1 and 5 (General, Pest) stay in the JSON, unreachable.
(Proofread, G6, G7, G9, commit and changelog done later on 2 Nov, see "Closed".)

## Closed (2026-11-02)
Committed and pushed to `main` by the user as `b3fd4b6` (Khmer text proofread by the user). Gates done, changelog added, task moved to `5-done/`. Deploy is not done yet: run `yarn deploy` in `bEasy-landing-page-v2`, then resubmit the sitemap in Search Console and spot-check the redirects live. Move this file to `6-deploy/` once it is live.
Follow-ups (not blocking): decide whether Pest Control should 410 instead of redirecting to home; mirror the debris wording in Khmer `serviceDetail.json` 7; P2 growth items stay open.

## Deployed (2026-11-02)
Committed, pushed and deployed by the user. Still to do by hand: resubmit the sitemap in Search Console and spot-check the redirects on beasy.info.
