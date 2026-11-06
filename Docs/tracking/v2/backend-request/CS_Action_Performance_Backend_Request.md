# CS Action and CS Performance: what is still open with backend

**To:** Backend · **From:** Portal · **Updated:** 6 Nov 2026
**Portal tasks:** T-0104, T-0105, T-0106, T-0107 (branch `feat/cs-action`) · **Plan:** `Tasks/_plans/Portal-cs-action-performance.md`
**Backend reply (authoritative for what is answered):** [CS_Admin_Portal_Implementation_Guide_5Nov2026.md](../backend-response/CS_Admin_Portal_Implementation_Guide_5Nov2026.md)

Answered on 5 Nov and removed from this file: C1 to C5 (display fields, last contact, paging, open filter, names), D1 (segment follow-up), D2 (support from an existing ticket), D3 (review outcomes), D4 (shape accepted), D5 (monthly targets), A1 (staging clean-up). The portal is now built to that guide on sample data; nothing is verified against a live response yet.

## Open

| # | Ask | Blocks |
|---|---|---|
| B1 | Deploy the branch to the api-dev service (needs the api-dev deployment profile), then confirm an authenticated `GET /api/admin/cs/actions?page=1&limit=1` returns 200. Also confirm the exact permission strings `CS_ACTION` and `CS_PERFORMANCE` appear in the role editor list. | Turning off sample data, any real test |
| A2 | Production segment definitions: a read-only export or DB profile, plus environment ID mappings. Until then staging has only the two protected test segments. | Saved segments with real data |
| B2 | Scheduler: date, cadence, operator account and lookback for `/api/scheduler/cs-actions`, and the cron on api-dev. Until then tasks need the manager "Sync new orders" button. | Tasks appearing without a manager |
| E1 | Policy values: scope-call SLA, retry timing, escalation threshold, sales-assistance window, review eligibility/timing, revenue and VAT treatment. | Due times, sections A to C |
| N1 | **New.** Resolution names for closing a sales follow-up: the guide mentions converted, lost and "sales decline" (permanent no-further-contact) but not the `resolution` strings to send in `complete`. Please list them. The portal keeps Complete disabled for sales tasks until you do. | Closing sales tasks |
| N2 | **New.** Chat save adapter: when does Chat call the segment/conversation ingestion? Unregistered Chat leads stay out of the queue until then. | Chat sales tasks |

## Portal notes for backend

- The portal sends no `afterId` on the first reconcile call and follows `nextAfterId` only while `mayHaveMore`.
- Past-month target edits are disabled in the UI; a 422 is shown with your message.
- Review prompt eligibility uses the top-level `reviewPaused` only.
