> **Archived 2026-10-31.** Answered in `../backend-response/CS_Overview_Backend_Answers_31Oct2026.md`. Open items left: D1 (case storage date) and D2 (problems found) wait for the lead; A1 waits for the dev deploy.

# CS Overview: questions for backend

Updated: 2026-10-31
From: portal (task `Portal-cs-overview`, branch `feature/cs-kpi`)
Replying to: `backend-response/CS_Overview_Frontend_Handoff_29Oct2026.md`

Thank you for the handoff. The portal starts integrating the CS Overview report on 31 Oct (due 4 Nov). Most of the contract is clear. The items below are what we cannot settle from the document. Each has the answer we will build on if we hear nothing, so none of them blocks the start.

## A. Needed before we can test

**A1. Target storage on dev.** The portal tests against the **dev** API (`api-dev`), not production. Is the deployed dev backend on the `feature/cs-kpi` version, and is `sql/cs-overview-targets.sql` applied on the dev database? We will call `GET /api/admin/cs-overview/targets/2026-10` on day 1. If it returns 503 we build the editor but keep it disabled.

## B. Decisions we need confirmed

**B1. Who protects summary, people and export?** Handoff §2 says `GET /cs-overview`, `GET /cs-overview/people` and `POST /fileExport/cs-overview` only require `authAdmin`, with no `CS-OVERVIEW` check, while the targets endpoints do check it. Will backend add `CS-OVERVIEW / VIEW` to those three? The portal gates the route and sidebar with `CS-OVERVIEW` VIEW (confirmed as the module name), so a role without it sees no page, but the API would still answer. People lists return full phone numbers (§6), so we think this should be enforced server-side. *Default if no answer:* portal gates in the UI only and we note the gap in the release notes.

**B2. Targets outside a single month.** Targets exist per month only. When the user picks a date range or Overall, we plan to show rates **without** an on/under-target indicator, with the note "Targets are set per month". We will not average monthly targets. Is that what you expect? *Default:* as described.

**B3. Which target applies to which rate.** We plan this mapping, please confirm:
- `firstSaleConversionTarget` vs `firstSales.contacted.rate` (team) and each row of `firstSales.contacted.byEmployee`.
- `repeatConversionTarget` vs `repeatSales.total.rate` and each row of `repeatSales.byEmployee`.
- `completedOnTimeTarget` is editable but has no actual value yet (no task history), so we show the target only.

Compare is `rate * 100 >= target`, so equal meets target.

## C. Questions about data

**C1. Employee list.** `employees` is "report-derived" (§5). Does it include a CS employee with no contacts in the period? If not, the filter and the by-employee tables will hide anyone with no activity. Is there an existing endpoint that lists all CS staff (users with a CS role) that the portal should use for the employee filter?

**C2. Segment counts at scale.** To show a live count per saved segment we make one `countOnly` request per segment (§8). If a team has 20 or more segments that is 20 or more requests per page load. Is there, or can there be, one batch call that takes segment ids and returns counts? *Default:* we cache per segment, limit concurrent requests to 4 and load counts lazily when the segment list is visible.

**C3. Order links.** For order rows the handoff says to link by `bulkOrderId` (`/order?bulkOrderId=...`). Can `bulkOrderId` be `null` on `DIRECT_SALE` orders? If so, what should the portal link to? *Default:* show the order id as plain text with no link when `bulkOrderId` is null.

## D. Roadmap (so we can plan what the design shows)

The Figma Super Admin view has the follow-up work block (assigned, not contacted, due today, overdue, on time), Send to CS and the CS inbox. Backend lists these as not implemented (§1, §14).

**D1.** Is there a target date or sprint for CS case storage and the `/cs/*` endpoints? The portal shows "Not implemented yet" until then. We need to know whether to plan a second phase in November.

**D2.** Is the exclusion of "Problems found by CS" permanent, or only for this release? The design still shows it with a 5% weight in the CS KPI. If it is permanent, we will ask design to remove it from the portal view.

**D3.** Per-list export (Export list on the people sheet) has no contract yet (§11, §14). Please add it to the backend backlog; the portal keeps the button disabled.

## E. For information (no answer needed)

- We reuse `app/api/api.ts` (it already unwraps `data`), so no second unwrapping.
- We send the same report query to summary, people and export, and reset list pages to 1 when the scope changes.
- We show null money as "Unavailable", null reached as "Unavailable", and not-implemented items as "— Not implemented yet", as in §5.
- After staging numbers are available we will compare a few months against source records (§14 step 5) and send any mismatch back here.
