# CS Action & Management Performance Dashboards

Developer + UI handoff  |  Version 1.0  |  3 November 2026

Scope: two dashboards only. The Unified Inbox remains the communication tool; existing CRM segment lists remain the configuration source. This brief defines task generation, outcomes, reporting and integration. Proposed defaults requiring approval are flagged as such.


## 1. Confirmed structure


|Module|Audience|Purpose|

|---|---|---|

|CS Action Dashboard|Designated CS staff (currently 1–2); management read/override|Single working queue: what to contact, confirm, follow up or resolve.|

|Management Performance Dashboard|Management / Super Admin|Sales results, operational follow-up, Chat closing rate, review collection.|

|Unified Inbox / Chat|CS team|Messages and manual conversation outcomes; creates/updates relevant action records.|

|CRM Customers / Registered Users|CS and authorized management|Existing customer lists, filters and saved segments; do not rebuild their segment editors in dashboard.|




## 2. CS Action Dashboard — UI and common task logic

Top counters: To Do | Due Today | Overdue | Completed Today. Main working categories are tabs/filters on one dashboard, not four dashboards.


|Working category|Trigger / task created|CS action and closure|

|---|---|---|

|New Orders|Confirmed paid App order OR confirmed Direct Sale/deposit. Never pending or failed payment.|Mandatory call to confirm service scope and job details; record verification result. Retry/escalate unanswered calls.|

|Registered Users|Eligible saved CRM segments; checkout/booking abandonment; Chat sales follow-up.|Contact and help with first booking; close on verified conversion, qualified lost decision, or recorded permitted closure.|

|Existing Customers|Saved customer segments; service-specific rebooking due/overdue/lost; cross-sell/recovery; Chat sales follow-up.|Contact about repeat booking or new service; track opportunity and verified repeat conversion.|

|After-Service|Eligible completed orders (review collection); complaints, low ratings, refunds, re-clean and unresolved support.|Request review fairly and record outcome; handle issues through linked support/recovery cases.|




### 2.1 Task fields and states

Each task should display: category/type; customer or unregistered lead; contact method; service interest or order/service; trigger/source segment; urgency and due time; latest contact/outcome; next action; linked conversation/order/ticket; created time; temporary staff handling indicator.

Task states: To Do → In Progress → Waiting / Follow-up Scheduled → Completed. Escalated is an open state or flag, not completed. Overdue is calculated from an actionable due time. Manual override to release a stuck task is available to management with audit history.

- Shared queue: no permanent customer assignment. Staff taking a task are shown as temporarily handling it; another authorized staff member can resume when appropriate.

- Avoid duplicate open tasks for the same actionable event/opportunity. Preserve multiple source segments and conversation links on one task.

- A conversation closed in Chat does not automatically complete its outstanding CS task. Task completion needs an explicit resolution/outcome.

- CRM segment refresh must check conversion, staleness, customer opt-outs and task eligibility; suppress ineligible outreach while preserving historical evidence.


## 3. New Orders — mandatory scope-confirmation calls

- Generate exactly one initial scope-confirmation task per newly confirmed App/Direct Sale order. Multi-service orders may need service-line checklists, but retain one order-level task and record each line as verified or pending.

- Checklist by service: booked scope, property/access instructions, date and time, equipment/access requirements, add-ons, special needs, and any service-specific restrictions.

- Record call attempts with employee, timestamps, reached/not reached, details confirmed, discrepancies and follow-up. An attempted unanswered call is not a confirmed scope.

- If scope materially changes after booking/rescheduling, create or reopen an appropriate verification action without duplicating the order.

- Do not classify an App order as CS-assisted merely because CS called after purchase to verify scope.

Developer configuration to confirm with Operations: initial call SLA, retry cadence, overdue/escalation threshold and any critical pre-service cutoff. Do not hardcode unapproved values.


## 4. CRM / sales follow-up generation

- Registered Users: reuse existing saved segment names and actual filter definitions, including abandoned booking/checkout, service interests, lead interest levels and unconverted/inactive registrations.

- Existing Customers: use service-specific approved expected rebooking intervals to identify Due Soon, Overdue and Lost. Do not apply a single assumed interval across all services.

- Chat-origin leads may be unregistered. Keep their external identity and enquiry in the shared sales queue; link to CRM profile only when the match is verified.

- For sales contact, record service interest, attempt vs reached, outcome, next action and whether customer purchased within the approved pre-order attribution window.

- Manual CS follow-up date/time is mandatory if follow-up is Yes; maintain open task until due/resolved and avoid duplicate tasks with CRM-generated ones.


## 5. Chat integration — use existing developer amendment

The Chat enhancement brief has already been sent to development. Do not rebuild the inbox. The data contract here is what the dashboards need to receive.


|From existing / enhanced Chat|Destination / reporting use|

|---|---|

|Channel, conversation ID, first message and first human reply|Contact log, response-time reporting and customer history|

|Acquisition-source response, service-interest auto-reply|Source and lead-interest attributes (separate fields)|

|Conversation purpose + simplified outcome|Sales/support classification, qualified opportunity status|

|Not-booked reason|Lost-sales reason breakdown; not guessed for pending leads|

|Follow-up Yes/No + next date/time + remarks|Create/update the appropriate action task, never duplicate|

|CS handler, timestamps, linked customer, quotation, order|Employee activity; verified conversion and revenue attribution|



Chat outcomes currently agreed: Booked; Interested / Pending; Not Interested; No Response; Resolved. Not-booked reasons: Price; Schedule; Service Unavailable; Chose Competitor; Not Ready; No Response; Other / Unknown. The system verifies Booked against an actual order; do not recognize self-reported Booked as revenue.

Existing rules: no AI needed in Phase 1; do not count an automated reply as a human CS reply. Human pre-order sales assistance may count toward assisted App sales, whereas post-order scope calls, general support and review requests do not count for that same order.


## 6. After-Service — reviews and complaints

- For each eligible completed App or Direct Sale order, create one review-request action unless a review already exists or contact is disallowed. Include all eligible customers fairly, not only satisfied customers.

- Possible review outcomes: Requested / Awaiting, Received, Declined, No Response, Negative Feedback. Negative Feedback should create/link an issue case and prioritize service recovery over further review prompting.

- Link service review to order and mark review task complete on verified receipt or explicit decline. Do not repeatedly contact customers who decline or opt out.

- Complaints, re-clean requests, low ratings, refunds and disputes remain linked to their issue records; escalation and financial resolution must not be falsely marked complete because Chat closed.

Review channel(s), follow-up interval, number of attempts and eligibility rules are configuration decisions still to be confirmed. Use bEasy in-app review when available; do not assume public review links exist.


## 7. Management Performance Dashboard — four reporting sections


|Section|Headline measures / segmentation|

|---|---|

|A. Direct Sales|All Direct Sale orders and revenue; split by one accountable sales owner: Sales or CS. Show counts and values without double counting.|

|B. App Non-Assisted|Unique customers; all non-assisted App orders (first + repeat); revenue; post-purchase CS contact yes/no; drill-down for contact timestamps.|

|C. App Assisted|First-time unique customers assisted before first App order; returning customers assisted before repeat App order; assisted App order count and revenue.|

|D. Follow-up Performance|Registered-user and existing-customer outreach without sale (distinguish pending); contacts attempted vs reached; due/completed/overdue; new-order verification coverage; reviews; chat opportunity closing rate, channel and lost reason.|




### 7.1 KPI definitions — developer formulas


|Metric|Definition / rule|

|---|---|

|Chat closing rate|# eligible unique sales opportunities resulting in a verified order ÷ # eligible unique sales opportunities in the selected cohort. Show pending separately; document conversion window.|

|App assisted orders|Verified App orders with qualifying human sales contact before purchase and within the approved attribution window. One order belongs to exactly one assisted/non-assisted class.|

|App non-assisted orders|Verified App orders without qualifying pre-purchase assistance; include first and repeat purchases.|

|Direct Sale CS vs Sales|Each Direct Sale order attributed to one approved primary sales owner. No duplicate totals.|

|New-order scope confirmed %|# confirmed orders with scope verification completed ÷ # confirmed orders requiring verification; show unanswered/overdue separately.|

|Review request coverage|# eligible completed orders with a logged request ÷ # eligible completed orders.|

|Review conversion|# orders with review received ÷ # orders for which a review was requested (same eligible reporting cohort).|

|Follow-up completion %|# due action tasks completed with resolution ÷ # tasks due during selected period; overdue shown separately.|

|Revenue|Report order value/confirmed sales separately from completed net recognized revenue. Reflect cancellations/refunds and explicitly label VAT treatment.|



KPI safeguards: use unique order IDs and opportunity IDs; show customers and orders separately; prevent double counting across Chat and CRM; distinguish attempts vs actual contact; keep pending conversion windows outside finalized loss counts. Any reporting clock must use Cambodia time.


## 8. Reporting UI, filters and permissions

- Action UI: one shared sortable task list, tabs for four categories, count cards, due/overdue filters, quick access to customer, order, Chat and tickets, and a task action modal.

- Performance UI: four sections A–D, monthly/date range filter, source/channel, service, customer first/repeat, staff and App/Direct Sale filtering where meaningful; drill-down and export.

- Management-only dashboard; CS may view own task progress without revenue-level management reports unless authorized. Avoid employee weighted scores/targets in this phase.

- Record audits for outcome edits, task completion/reopen, contact records, customer linking, assistance classification overrides and Direct Sale attribution corrections.


## 9. Acceptance tests for development / QA

- Confirmed App booking → exactly one scope-call task; unpaid/failed checkout → none.

- Confirmed Direct Sale/deposit → scope-call task; duplicated confirmation event → no second task.

- Unanswered call → attempt logged and task remains open, not scope confirmed.

- Chat Interested/Pending + follow-up date → one task in correct working category; duplicate Chat save does not duplicate task.

- Unregistered Telegram lead → task remains actionable; later verified account match merges history safely.

- Chat marked Booked with no verified order → no converted sale/revenue.

- Post-order scope confirmation Chat → non-assisted App order stays non-assisted.

- One eligible completed order → review task; existing review → automatically satisfied/no duplicate request.

- Negative review → linked support case and review prompting suppressed during issue handling.

- No-show/failed contact → remains pending under configured retry, not silently lost.

- Direct Sale owner changed from Sales to CS → aggregate still counts order/revenue exactly once.

- Refund/cancellation → management revenue outputs reflect selected accounting definition and change history.


## 10. Configuration / approvals still needed

Do not implement guesses for the following. UI can expose settings or use management-approved values once supplied:

- Order scope-call SLA, attempts, retry timing and escalation threshold.

- Service-specific repeat booking intervals for Due Soon / Overdue / Lost.

- CRM refresh frequency, outreach suppression/contact frequency limits.

- Sales assistance attribution window and what constitutes a qualifying human sales interaction.

- Chat sales-opportunity deduplication and reporting cohort/conversion window.

- Post-purchase contact-reporting window and which contacts qualify.

- Review eligibility, preferred request channel, timing, retry limits and opt-out rules.

- Revenue recognition basis, VAT treatment and primary Direct Sale sales-owner governance.


## 11. Recommended delivery order

1. Unified task record + Action Dashboard list/statuses and New Orders calls.  2. CRM and Chat follow-up ingestion, matching and deduplication.  3. After-Service reviews/issues.  4. Management KPI calculations, audit, filters and export.  5. End-to-end QA using the acceptance tests above.

Implementation principle: one source of truth for each order, opportunity, contact and task. Chat records communications, CRM records segment membership, Action Dashboard records accountable next steps, and Performance Dashboard reads verified underlying events.
