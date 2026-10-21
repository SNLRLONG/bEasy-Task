# Journey acquisition: new-user cohort, source provenance, purchase definitions

- **ID:** T-0017
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-acquisition/` · **Branch:** `feat/journey-acquisition-cohort`, from `origin/feat/journey-cms` after `Portal-journey-contract-sync` merges
- **Tier:** T2 · new filters on live API calls, several shared funnel components (see `PIPELINE.md`)
- **Claimed by:** agent (claude), 2026-10-20. Owner said: work directly on `feat/journey-cms`, no separate worktree, portal side only
- **Started / updated:** 2026-10-20 / 2026-10-19
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` (DQ-03, 04, 06, 10, 11, UX-11, UX-12) · handoff "Acquisition" · `Docs/tracking/v2/backend-request/Journey_Acquisition_API_Request.md`
- **Depends on:** `Portal-journey-contract-sync`; backend answers to plan open questions 1 and 3 (ad_clicked with default `new`, new vs returning rule)
- **Touches:** `app/components/journey/funnel/{acquisition-filters,customer-type-menu,acquisition-funnel-tab,kpi-row,kpi-card,after-purchase-card,after-purchase-orders-sheet,compare-campaigns-card,counting-mode-banner}.tsx`, `app/components/journey/filters/source-filter.tsx`, `app/components/journey/list/row-chips.tsx`, `app/lib/journey/{filters,acquisition-options,filter-options}.ts`, `app/locales/journey/{en,km}/{funnel,funnel-extra,list}.json`. Overlaps with `Cross-acquisition-ad-data` item E (`stage-card.tsx`)

## Goal
The Acquisition tab answers "how many new people came, from where, and how many bought" with definitions on screen, and does not treat returning customers as registration drop-offs.

## Scope
1. **Cohort (UX-11, DQ-03).** Default selection is "New customers" (backend default `customerKind=new`); options New / Returning / All. A short cohort line under the filters: "People whose first app open is in the period, followed for 7 days" (use backend window fields when present). Returning people are not shown as sign-up drop-offs.
2. **Source (DQ-06, UX-12).** Source filter gains Attributed / Organic / Unknown (`sourceType`). `other:Unknown` is labelled "Unknown", never "Organic", in the menu, campaign table, list rows and filter chips. Campaign row applies that campaign as the filter and opens its list (reuse `campaignId`).
3. **Purchases (DQ-04).** The first-purchases KPI opens a small breakdown from `purchaseCounts`: first-time buyers, buyers, orders, purchase events, each with one line of definition.
4. **After purchase (DQ-10).** Each row shows its disclosed denominator ("5 of 11 orders") from the backend field. Unavailable amounts stay "—" with the `not_tracked` reason.
5. **Identity (DQ-11).** Unit text from `identityUnit`: "customer or device". Rows get chips New / Returning and First purchase / Bought before (`customerKind`, `firstPurchase`, `everPurchased`).
6. If `ad_clicked` becomes "not applicable" under the default filter, show the existing reason text; do not hide the stage. Copy for empty states stays with `Cross-acquisition-ad-data` item E.

## Gates (T2)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog
Skipped G6: no new personal data.

## Done when
- [ ] Switching New / Returning / All changes funnel and list consistently; chips show and clear
- [ ] One 7-day window reconciles: first buyers, buyers, order IDs, purchase events (sign-off demo 4)
- [ ] An Unknown-source person is never labelled Organic (sign-off demo 7)
- [ ] After-purchase ratios recompute against the shown base
- [ ] Filter state is in the URL and in the export query
- [ ] Khmer strings added (native read pending)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress 2026-10-20 (agent, on `feat/journey-cms`, uncommitted)
Backend response 20 Oct confirms `unit: people`, `identityUnit`, `purchaseCounts`, `other:Unknown`, `sourceType` AND `sources`, after-purchase `denominator`. Done: Customer select (default = New customers, then Returning and All, via `customerKind`; no value sent for the default); new Source type select (Attributed, Organic, Unknown, `sourceType`); "Who bought" card with first-time buyers, buyers, orders and purchase events, the cohort sentence and the unit sentence; after-purchase rows print "5 of 11 orders (45%)"; person rows get New / Returning and First purchase chips; the portal forces `unit=people` on acquisition calls (a stale `devices` would be a 422). The select no longer uses `all` as its "no filter" value, because `customerKind=all` is real.
Open: the default cohort is `new` only per the 19 Oct handoff (the 20 Oct response does not repeat it): check the label "New customers" on dev. Campaign row opening its list is not done. Unknown source labelling comes from `sourceRowLabel` (shows "Unknown"): check with a real `filter-options`. G4, G5, G8, G11 open.
