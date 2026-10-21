# Acquisition: what it takes to show ad data

**Updated:** 19 Oct 2026 · Task: [`Tasks/1-backlog/Cross-acquisition-ad-data.md`](../../../../Tasks/1-backlog/Cross-acquisition-ad-data.md)

Today the Acquisition tab shows `Reached —` on **Ad clicked** with the tooltip "ad platform (Meta) is not connected". The API shape is done; only the data is missing. It is **not only a backend wait**: four parties each own a piece.

Contract: [`Journey_Acquisition_API_Request.md`](../backend-request/Journey_Acquisition_API_Request.md) · what backend built: [`Journey_V2_Portal_Review.md`](../backend-response/Journey_V2_Portal_Review.md) §3, §8 · endpoints: [`Journey_Backend_API_Spec.md`](../api/Journey_Backend_API_Spec.md).

## What is blocked, and what is not

| Part of the Acquisition tab | Needs Meta/TikTok? | State |
|---|---|---|
| App opened to Purchased stages, drop-offs, people lists | No | Works (data starts the day guest mode went live; earlier periods show 0) |
| After-purchase block | No | Works; reschedule/cancel/refund amounts `null` until backend stores them (request 3.3) |
| **Ad clicked** stage, `ad_clicks` and `click_to_buy` KPIs | Yes | `null`, `unavailableReason: meta_not_connected` |
| Campaign table: `adClicks`, `clickToBuy` | Yes | `null` |
| Ad set filter and ad set on people rows | Yes | `adsetId` returns `422` |
| Per-campaign funnel (campaign names on app opens) | No, but needs mobile | Mostly "Organic" until the app sends campaign info |

## Who does what

| # | Party | Work | Blocks | Where it is tracked |
|---|---|---|---|---|
| 1 | **Marketing / business owner** | Give the backend read access to the Meta ad account (and TikTok). Name who owns the Business Manager and who can issue a system-user token. | Everything below on the ad side | Not requested yet. Task item A |
| 2 | **Backend** | Daily import of clicks, impressions and spend per campaign and ad set, stored with a date. Backfill of past periods. Set `integrations.meta.connected = true`. Fill `ad_clicked.reached`, KPIs, `campaigns[].adClicks`, `filterCounts.adsets`, filter-options ad sets. Keep `unavailableReason` for the cases it still applies to. | Portal values | `backlog.md` ("Meta Ads import"), not yet in `pending-request-backend.md` |
| 3 | **Backend** | Use the stored attribution touch when `app_opened` has no campaign fields | Per-campaign funnel | [`Campaign_Source_Request.md`](../backend-request/Campaign_Source_Request.md) |
| 4 | **Mobile app** | Send real campaign names (OneLink / `campaign_code`) on app opens; keep `X-Campaign-Code` at account creation. Decide the deep-link provider (Firebase Dynamic Links shut down in 2025; see architecture guide §4.5). Set Firebase `user_id` on login. | Campaign names, end-to-end sign-off | `pending-request-backend.md` (mobile note), architecture guide §4.5 |
| 5 | **Portal** | Already wired to the contract. Remaining: clearer empty state (inline reason, tooltip below the card), then a real-data check once values arrive. No new endpoint needed. | Polish only | Task item E |

## Rules to agree before backend builds (decisions, owner: product + backend)

1. **Which click number?** Meta reports "clicks (all)" and "link clicks". Pick one and label it the same on both sides. TikTok has its own definition; the `channel` filter keeps them apart.
2. **Join key.** Meta rows and app opens meet on the campaign key `meta:<campaign name>`. Names must match exactly between the ad platform and what the app sends. Confirm with request 4.3 (ids) and 4.2 (a real `filter-options` sample).
3. **History.** How far back the backend backfills. The ad platforms keep history but not forever, and our own app-open data starts at guest-mode go-live, so earlier ad clicks have no matching opens. Decide whether to show them, or start the Ad clicked stage from go-live.
4. **Filters that cannot split ad totals.** `serviceIds`, `device` and `customerKind` return `reached: null` with `unavailableReason: filter_not_applicable`. The portal must show a different message for this than for `meta_not_connected`.
5. **Refresh and lateness.** How often the import runs and how late ad numbers may lag (`dataAsOf` already exists).
6. **Spend.** Not in the Acquisition contract (CAC and ROAS are the later `/attribution` page). Decide whether to import spend now while the job is being built, to avoid a second pass.

## Order of work

1. Marketing grants access (1). Nothing else on the ad side can start.
2. Product and backend settle the rules above.
3. Backend builds import and backfill (2), and the attribution-touch fix (3) in parallel.
4. Mobile confirms campaign names and deep links (4).
5. Backend sends one real `GET /funnels/acquisition` and `GET /meta/filter-options` sample. Portal verifies against the UI (5) and closes the items.

## When it is live

Per the sync rules in `AGENTS.md`: delete the answered item from `pending-request-backend.md`, set the API spec status, add the reply under `backend-response/`, then move this file to `archive/` with a banner.
