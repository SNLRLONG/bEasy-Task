# CS Action and CS Performance: what is still open with backend

**To:** Backend · **From:** Portal · **Updated:** 6 Nov 2026 · sent to backend 6 Nov
**Portal tasks:** T-0104, T-0105, T-0106, T-0107 (branch `feat/cs-action`) · **Plan:** `Tasks/_plans/Portal-cs-action-performance.md`
**Backend reply (authoritative for what is answered):** [CS_Admin_Portal_Implementation_Guide_5Nov2026.md](../backend-response/CS_Admin_Portal_Implementation_Guide_5Nov2026.md)

Answered on 5 Nov and removed from this file: C1 to C5 (display fields, last contact, paging, open filter, names), D1 (segment follow-up), D2 (support from an existing ticket), D3 (review outcomes), D4 (shape accepted), D5 (monthly targets), A1 (staging clean-up). The portal is now built to that guide on sample data; nothing is verified against a live response yet.

## Open

Verified on api-dev (6 Nov, read-only GETs): the CS routes answer 200 and match the guides; `CS_ACTION` and `CS_PERFORMANCE` are in the role editor list. Backend replies so far: [6 Nov](../backend-response/CS_Action_Backend_Reply_6Nov2026.md), [6 Nov PM](../backend-response/CS_Action_Backend_Reply_6Nov2026_PM.md). Built in the portal from them: resolutions and Complete options, Chat `csAction` message, Direct Sale counts, scheduler status badge, `valueStates` legend.

### Asks for backend

| # | Ask |
|---|---|
| N1 | `POST /actions/:id/transitions` with `action:"take"` and no note returns 400 "note is required (maximum 2000 characters)" on api-dev. Your 6 Nov checklist says a note is needed only for `complete` (and `not_eligible`). Please make `note` optional for `take`, `schedule` and `escalate`, or confirm it is required. Portal decision (6 Nov, latest): Take is one click with no note field; Schedule and Escalate show an optional note. Until you relax the rule the portal sends a short automatic note ("Picked up in CS Action") when staff type none. Complete, release and reopen keep a required note. |
| S1 | Bulk start of sales follow-ups from a saved segment: one call with `segmentId` plus `customerIds` (or "all current members"), returning created, duplicate and skipped counts with a reason per skipped customer. Today the portal sends one `POST /actions` per customer, up to 100 per run, one after another. |
| S2 | Optional `dueAt` (timezone-qualified, future) on `POST /actions`, so a manager can set the follow-up deadline when starting, for one customer or in bulk. Today scheduling needs the task to be taken first. No default: leave it null when omitted. |
| D1 | Deploy the branch to api-dev and run `sql/cs-action-scheduler-status.sql`, so `/cs/scheduler/status` and `sections.directSales.measured` are live. The status endpoint returns 404 today. |
| F1 | Filters `channel` and `category` on `/performance` and `/performance/records` (no E1 needed). The portal will send none until you confirm. The others (`orderType`, `customerType`, `serviceCategoryId`, `source`) wait for sections A to C. |
| P5 | Policy settings API (`GET /cs/policies`, `PUT /cs/policies/:key`) after management approves values. Until then managers send values to you. Portal settings screen stays in the backlog. |
| A2 | Production segment definitions: a read-only export or DB profile, plus environment ID mappings. Closed for now (6 Nov). |

### Waiting on management (not backend)

| # | Item |
|---|---|
| B2 | Approve the scheduler operator account, lookback and cadence. Until then managers use Sync new orders. |
| E1 | Policy values: scope-call deadline, retry timing, escalation threshold, sales-assistance window, review rules, revenue and VAT. Until then tasks have no deadline. |

## Portal notes for backend

- The portal sends no `afterId` on the first reconcile call and follows `nextAfterId` only while `mayHaveMore`.
- Past-month target edits are disabled in the UI; a 422 is shown with your message.
- Review prompt eligibility uses the top-level `reviewPaused` only.
