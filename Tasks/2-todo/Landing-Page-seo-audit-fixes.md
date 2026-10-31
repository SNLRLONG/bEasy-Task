# Landing Page SEO Audit Fixes

- **ID:** T-0045
- **Repo:** bEasy-landing-page-v2
- **Owner:** mate
- **Due:** 2026-11-07
- **Worktree:** `bEasy/bEasy-landing-page-v2-seo-audit-fixes/` · **Branch:** `feat/seo-audit-fixes`
- **Tier:** T2 · touches multiple pages/routes/redirects/schema/URL hygiene (see `PIPELINE.md`)
- **Claimed by:** claude-code session, 2026-11-01
- **Started / updated:** 2026-11-01 / 2026-11-01
- **Docs:** `Docs/landingpage/bEasy_SEO_Website_Audit_Marketing_Handoff_30Oct2026.md`
- **Depends on:** none
- **Touches:** `app/**`, `components/**`, `data/**`, `next.config.ts`, `middleware.ts`, `public/**`, sitemap/robots/schema files, redirects

## Goal
Fix all P0/P1 SEO issues from the 30 Oct 2026 audit. Remove discontinued services (General Cleaning, Pest Control), correct service scopes/claims, add 301 redirects for legacy URLs, rewrite Office Cleaning, fix AC/Washing Machine/Upholstery/Post-Renovation/Instant/Deep pages, clean About/article pages, improve internal linking, and address technical SEO (canonicals/sitemap/robots/hreflang/schema). Work directly on main per user request (no commit unless asked).

## P0 - Immediate Accuracy Cleanup (Blockers)
- [ ] Remove discontinued services (Pest Control, General Cleaning) from homepage, footer, FAQ, About, structured data, shared data/modules
- [ ] 301 redirects: `/en/service-detail/2` → `/en/services/deep-cleaning`, `/en/service-detail/7` → `/en/services/post-renovation-cleaning`; reconcile `/en/services/general-cleaning` (301 to appropriate active service)
- [ ] Rewrite Office Cleaning page (remove residential copy, custom quotation, recurring/monthly invoicing, H1: "Professional Office Cleaning in Phnom Penh")
- [ ] Fix Washing Machine Cleaning scope (remove full dismantling as standard, use non-invasive wording, add safety-stop, soften absolutes)
- [ ] Fix Upholstery Cleaning scope/guarantees (exclude leather/suede/silk/velvet unless approved, remove shrink/damage guarantee, add pre-check, remove unverified heat treatment)
- [ ] Fix AC Cleaning page (remove pest-control wording, keep promotions separate from SEO copy)
- [ ] Redirect obsolete article "General Cleaning vs Deep Cleaning" → "Instant Cleaning vs Deep Cleaning" (301)
- [ ] Update "What is bEasy?" article (remove retired services, proofread)
- [ ] Homepage claims (policy-safe reschedule, assignment-safe cleaner choice, verify social proof numbers)

## P1 - Strengthen Commercial Pages
- [ ] Expand Deep Cleaning page (fix typos, scope, approved add-ons/exclusions, related guides)
- [ ] Tighten Post-Renovation Cleaning (limit debris/waste wording, remove % claims, scope-based manpower/duration, remove Pest Control add-on)
- [ ] Update Instant Cleaning (compare with Deep Cleaning not General, clarify extra hours, tighten wording)
- [ ] Add Phnom Penh-focused H1s, improve FAQs/exclusions where needed
- [ ] Add article↔service internal linking clusters (real HTML links)

## P1 - Technical SEO
- [ ] Canonicals (self-referencing, no legacy service-detail canonicals)
- [ ] Sitemap (only canonical/indexable, remove retired/duplicate URLs, include articles)
- [ ] robots.txt (correct, references sitemap, blocks private routes)
- [ ] hreflang (EN/KH reciprocal, x-default if needed)
- [ ] Structured data (Organization/LocalBusiness, Service, Article, BreadcrumbList)
- [ ] Redirect QA (verify 301s, no 200 legacy duplicates)

## P2 - Growth (lower priority for now)
- [ ] Search Console focus on positions 8-20
- [ ] Plan commercial-intent articles (prices: Deep/Office/AC/Mattress/Post-Reno, "1 hour vs 2 hours")
- [ ] Track to bookings/revenue

## Gates (copy rows tier needs; skip only with reason)
- [ ] G1 scope - [ ] G2 static - [ ] G3 tests - [ ] G4 run it
- [ ] G5 code review - [ ] G6 security - [ ] G7 docs sync
- [ ] G8 PR message - [ ] G9 human review - [ ] G10 release plan - [ ] G11 changelog

## Done when
- [ ] All P0 items addressed
- [ ] P1 items addressed as scoped
- [ ] Technical SEO checklist complete
- [ ] Tested locally (run it) where feasible
- [ ] Root `CHANGELOG.md` entry added (if/when committed per user instruction)
- [ ] No worktree/branch to remove (working directly on main)

## Notes
- Working directly on `main` per user request. Do not commit unless user says "commit" with files+message.
- Started date set to 01 Nov 2026 as requested.
