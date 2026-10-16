# Customer journey analytics — implementation report

The portal prototype is mock data. Every response sets `dataSource: "mock"` and `connectedToBigQuery: false`. It is not production analytics.

> **Status check, 8 Oct 2026.**
> 1. The prototype described below lives on the branch **`draft/tracking`** (commit `9cff83a feat: test`, 121 files changed against `main`, mock data). It is **not** on `feat/journey-cms` or `main`, and `bEasy-portal/app/lib/journey-analytics/`, `app/components/journey-analytics/` and `app/routes/test/` do not exist in the working tree.
> 2. Its screens (Overview, Booking funnel, Sign-up & campaigns, Customers, Follow-up, Audiences with templates, and the data explorer) are **superseded by the approved UI v2.1** ([Journey_Tracking_UI_Flow_v2.1.md](../v2/Journey_Tracking_UI_Flow_v2.1.md)). Do not merge the branch as is.
> 3. What can be reused: the funnel arithmetic (`metrics.ts`), the clean/raw/quarantine compile (`compile.ts`), the fixture world (`seed.ts`) and `tests/journey-analytics.test.mjs`, as fixtures for the v2.1 pages. The data explorer is a developer tool and can stay behind the test flag. See `bEasy-portal/TODO.md` section 2.
> 4. Rules in this report that v2.1 changes: single 60-minute abandonment (now per step), `browsed` status, ad click not a stage, registration step names, audience templates, and the follow-up tab.

## What a reviewer can trace

`service_viewed` for Instant Cleaning on journey `journey-story-checkout` is a raw client event with a null customer id, then a clean event resolved to customer `1102` through the 30-day identity link. The journey’s furthest stage is checkout, status is abandoned at the frozen clock `2026-10-02T09:00:00+07:00`, and payment outcome is `not_attempted`. That same journey is absent from the purchase stage and present in the checkout-abandonment count. The segment preview can select the same population without saving an audience.

## Routes

Both routes sit inside the existing signed-in layout. They have no extra permission handle, so any signed-in user can open them in development.

| Route | Screen |
| --- | --- |
| `/test/customer-journey-data` | Data explorer: raw vs clean events, journeys, orders, quality, schema |
| `/test/customer-journey-analytics` | Marketer dashboard |

The sidebar group **Test tools** is rendered only when `import.meta.env.DEV` or `VITE_JOURNEY_ANALYTICS_TEST=true`. In other production builds the route modules return a disabled notice and do not load the dashboard chunk.

Local dev server: `cd bEasy-portal && npm run dev`, then open the paths above after login.

## Documents

| File | Contents |
| --- | --- |
| [Journey_Tracking_Flow_Breakdown.md](../v2/Journey_Tracking_Flow_Breakdown.md) | Mobile use cases and separate Mermaid diagrams |
| [Journey_Tracking_Requirements_Matrix.md](../v2/Journey_Tracking_Requirements_Matrix.md) | Traceability and unresolved conflicts |
| [Journey_Tracking_Analytical_Model.md](../v2/Journey_Tracking_Analytical_Model.md) | Layers, grains, and metric formulas |
| [Journey_Tracking_API_Contract.md](../v2/Journey_Tracking_API_Contract.md) | Proposed HTTP contract matching the mock service |

`Journey_Tracking.docx` was not in the workspace and was not found on this machine. [Journey_Tracking.md](../v2/Journey_Tracking.md) is the functional baseline. Conflicts are listed in the matrix and were not silently merged.

## Code

Shared library: `bEasy-portal/app/lib/journey-analytics/` (`types`, `time`, `catalog`, `compile`, `metrics`, `seed`, `service`, `index`).

UI: `bEasy-portal/app/components/journey-analytics/`.

Routes: `app/routes/test/customer-journey-data.tsx`, `app/routes/test/customer-journey-analytics.tsx`.

Wiring: `app/routes.ts`, `app/components/common/app-sidebar.tsx`.

Tests: `bEasy-portal/tests/journey-analytics.test.mjs`. Script: `npm run test:journey`.

The existing per-customer journey page was not changed.

## Assumptions that are labeled in the UI

- Funnels use journey start time. Bookings and revenue use order time. Those counts are allowed to differ.
- Abandonment is 60 minutes. Journey id reuse is 24 hours.
- Existing customer means the first succeeded order is before the period.
- Cohorts are Monday weeks in the demo. The automation guide’s production grain is monthly.
- Campaign credit is the 7-day last touch on the journey, one campaign per journey. It is not holdout lift and not the 24-hour push rule.
- `payment_failed` is proposed. It is not a funnel stage.
- Missing intermediate funnel steps are counted and flagged as imputed.
- Lifecycle “at risk” uses 32 days as a stand-in for 1.5 × a 21-day gap.
- Segment preview is not a saved, exported, or activated audience.
- Demo promo codes do not change prices.

## Known gaps

- No BigQuery, Postgres, or live API. Architecture endpoints for saving audiences, export, playbooks, holdout, and ad spend are not implemented.
- `zone_id` is still missing from the address API and is not planned for now (the app sends `area_cell`). Push payloads still lack `notification_id`, `send_id`, and `action_id`; that is deferred to the notification phase. Legacy event names are quarantined here, but the backend skips them. The backend contract is reviewed in [tracking-backend.md](../v2/tracking-backend.md); open backend requests are in [backend-request/pending-request-backend.md](../v2/backend-request/pending-request-backend.md).
- Add-on, pair-service, and `push_opened` events are in the schema catalog with `inDemo: false`.
- A service filter scopes segment behaviour to that service’s events, so “has not booked” means “has not booked that service.” The preview text says behaviour uses the reporting period.
- Registration and acquisition funnels ignore the service filter and say so in their definitions. Acquisition can rise between steps; the UI does not show a conversion percent when a step is larger than the one before it.
- `npm run typecheck` for the whole portal still fails on existing files outside this work. The new journey-analytics files have no TypeScript errors.
- Full-repo `npm run lint` was not run. ESLint on the new files passed.

## Validation

`npm run test:journey`: 14 tests passed (funnel math, customer vs journey counts, dedupe, invalid custom dates, service filter, demo monotonicity, the worked example, imputation and quarantine, payment failure vs cancellation, segment AND/OR/exclude, bookings vs repeat rate, cohort rates and blank unfinished weeks, non-overlapping previous period, active vs browsed).

`npm run test:block-time`: 8 tests passed. Those cover blocked-schedule dates and were not changed by this work.

Browser, signed in on the local dev server:

- Analytics route shows the mock banner, 30-day KPIs, and a monotonic booking funnel (246 → 134 → 105 → 82 → 60 → 32). Drop-off and step rates match those counts. Overall conversion is 13.0%.
- Instant Cleaning changes the booking funnel to 60 → 30 → 25 → 21 → 16 → 9, bookings to 9, and revenue to $135 (9 × $15). Registration stays 30 → 23 → 17 → 15 → 7.
- Purchase-stage drill-down lists Instant Cleaning journeys at purchase, including one later cancelled. The sheet pages past 8 rows.
- Lifecycle counts for the unfiltered 30-day view sum to the 138 active people.
- Segment preset “viewed, did not start booking” returns a count, a push-reachable subset, and a sample, and states it is not saved.
- Preview error state replaces the charts with an error and no figures.
- Data explorer clean layer shows the same clock and the active checkout for customer `1103`. Data quality separates raw, clean, meaningful, quarantine, duplicate ids, technical duplicates, skipped steps, and the one person with tracking off.

Not confirmed in the browser: a custom date that fails validation (covered by the unit test), and a production build with the test flag off (the route module returns the disabled notice when `DEV` is false and the flag is unset; that branch was not loaded in the dev server).
