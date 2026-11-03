# SEO: visibility, service pages, metadata, enquiry tracking

- **ID:** T-0093
- **Repo:** bEasy-landing-page-v2
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-11-13
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2026-11-12 → 2026-11-13 (2 working days, order 3 of 12)
- **Worktree:** `bEasy/<repo>-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T2 · metadata, tracking events (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) nodes `1:2`, `1637:36622` (no frames for this task); KPI plan: November 2026, master task 10
- **Depends on:** backend work by none. Needs requirements, designs and API contract confirmed.
- **Touches:** to be set when planned

## Goal
Search visibility, service pages, metadata, tracking of enquiries from unpaid search results.

## Notes
Checked 2 Nov 2026: the audit fixes (T-0045) and the week 2 article (T-0040) are done and in `6-deploy/`. Remaining: enquiry tracking from organic search, Search Console follow-ups (resubmit sitemap, positions 8-20), commercial-intent articles. Cut to 2 days.
Figma scanned 2 Nov 2026: No frames for this task in the file (pages `1:2` UI | handoff and `1:4` Design were scanned). Estimate is unchanged from the KPI plan; ask for the design.
Ongoing; continues `Landing-Page-seo-audit-fixes` and `Landing-Page-seo-article-week2` (already in 6-deploy, do not recount).
November KPI task (web frontend, Seavlong). Estimate is provisional until the Figma frames are scanned.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Web frontend implemented against the agreed design and API contract
- [ ] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
