# Acquisition: get Meta/TikTok ad data on screen

- **ID:** T-0001
- **Repo:** cross (marketing access, backend, bEasy-app, bEasy-portal)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none yet (portal polish gets its own worktree when started)
- **Tier:** T2 · needs a backend change and spans several repos (see `PIPELINE.md`)
- **Claimed by:** none
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-21
- **Docs:** `Docs/tracking/v2/architecture/Acquisition_Ad_Data_Readiness.md` (who owns what, rules to agree) · `Docs/tracking/v2/backend-request/Journey_Acquisition_API_Request.md` · `Docs/tracking/v2/backend-response/Journey_V2_Portal_Review.md` §3, §8
- **Depends on:** Meta/TikTok account access (item A); backend `backlog.md` "Meta Ads import"
- **Touches:** portal `app/components/journey/funnel/stage-card.tsx`, `app/locales/journey/{en,km}/funnel.json`; docs only elsewhere

## Goal
Replace `Reached —` on the Ad clicked stage, the `ad_clicks`/`click_to_buy` KPIs, campaign clicks and ad sets with real numbers. The API shape is done; the data is not, and four parties own a piece.

## Items

| | Owner | Item | Status |
|---|---|---|---|
| A | Marketing / business | Grant backend read access to the Meta ad account (and TikTok); name the token owner | Not asked |
| B | Product + backend | Agree the 6 rules in the readiness doc (click definition, join key, history, filter-not-applicable, refresh, spend) | Open |
| C | Backend | Daily import, backfill, `integrations.meta.connected`, fill ad fields. Move from `backlog.md` to `pending-request-backend.md` once A is granted | Backlog |
| D | Backend + mobile | Campaign names on app opens: `Campaign_Source_Request.md`, OneLink / `campaign_code`, deep-link provider decision | Open |
| E | Portal | Empty state: show reason inline, tooltip `side="bottom"`, separate text for `filter_not_applicable` (EN + KM). Then check against real data | Not started |

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 commit message  - [ ] G11 changelog
Skipped G6: no auth, personal data or input handling. Skipped G9/G10: item E is a copy/layout change; revisit if C changes the contract.

## Done when
- [ ] A granted and B agreed
- [ ] Backend sends one real `GET /funnels/acquisition` and `GET /meta/filter-options` response with ad numbers
- [ ] Portal shows the numbers and the right empty state for each `unavailableReason`
- [ ] Docs synced: pending request item removed, API spec status updated, reply in `backend-response/`, readiness doc archived
- [ ] Root `CHANGELOG.md` entry added
