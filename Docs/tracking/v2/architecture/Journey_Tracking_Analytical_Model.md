# Analytical model and metric definitions

The admin prototype does not add a second warehouse. [bEasy-tracking-architecture-guide.md](bEasy-tracking-architecture-guide.md) already places immutable events in BigQuery and the journey read models in Postgres. This note maps the prototype onto that split and defines every number the screens show.

`dataSource` is `mock`. `connectedToBigQuery` is `false`.

## 1. Where each layer lives

| Layer | Store in the architecture | Grain | What the prototype does |
| --- | --- | --- | --- |
| L0 raw | BigQuery `events_*` | One exported Firebase row | `rawEvents`. Replays and invalid payloads stay here |
| L1 clean | BigQuery extract, plus a quarantine side table | One `event_id` | Dedupe, validate, flag technical duplicates |
| L2 identity and orders | Postgres 3NF (`identity_link`, orders) | One link, one order | `people`, `identityLinks`, `orders` |
| L3 read models | Postgres, rebuildable | One journey, one registration flow, one interest row | `journeys`, `regFlows`, `interests`, lifecycle |
| L4 aggregates | API over those read models | One query result | `app/lib/journey-analytics/metrics.ts` |

No extra BigQuery mart is required for the portal. The API must aggregate before the browser downloads anything. The mock service does that aggregation in process.

Reporting timezone is `Asia/Phnom_Penh`. The demo clock is frozen at `2026-10-02T09:00:00+07:00`. A period’s `to` is exclusive. `7d` is seven calendar dates ending on that day, so it starts six days earlier.

## 2. Tables the pipeline already implies

### `events_raw` (BigQuery)

- **Purpose.** Immutable client export.
- **Grain.** One Firebase row, including replays.
- **Keys.** `event_id`, `event_timestamp`, `user_pseudo_id`.
- **Columns that matter.** `event_name`, `user_id`, `ga_session_id`, `journey_id`, `reg_flow_id`, `service_id`, `platform`, app version, campaign params, `seq`, `schema_version`.
- **Partition.** Day on event date, `Asia/Phnom_Penh`.
- **Cluster.** `event_name`, `user_pseudo_id`.
- **Freshness.** Firebase export latency. Not controlled by this portal.
- **Retention.** Keep raw for audit. Do not use it for CRM counts.
- **Privacy.** No phone, email, advertising id, or street address. Consent-off instances should not be present; if they are, drop them in L1.

### `events_clean` (BigQuery extract or the Postgres `event_log`)

- **Purpose.** One row per `event_id` that analysts can count.
- **Grain.** Earliest row for each `event_id`.
- **Keys.** `event_id` primary. Join to journeys on `journey_id`, to people on resolved customer id or app instance.
- **Flags.** `quarantined`, `quarantine_reason` (`unknown_event_name`, `missing_service_id`), `suspected_technical_duplicate` when the same journey and event name repeat within 2 seconds, `proposed`, `derived`.
- **Partition / cluster.** Same as raw, clustered by `journey_id` and `event_name`.
- **Freshness.** Architecture schedule: daily extract is enough for the portal. The prototype is a full rebuild of a static fixture.
- **Privacy.** Resolved customer id is an internal id. Segment preview returns at most eight sample labels, not an export file.

### `identity_link`

- **Grain.** One link from an app instance to a customer.
- **Rule.** Resolve an event to the customer when a link is active at the event time, or when the link is created within 30 days after the event. Do not rewrite the raw timestamp or campaign.

### `journey` (read model)

- **Grain.** One `journey_id` (one person, one service, until purchase or 24 hours idle).
- **Keys.** `journey_id`. Foreign keys: resolved customer or app instance, `service_id`, `session` ids via events, `order_id`, `campaign_id`.
- **Measures.** `started_at`, `last_activity_at`, `furthest`, `skipped_stages`, `status`, `payment_outcome`, `order_value`.
- **Status.** `completed` and `cancelled` and `converted` come from the order when that timestamp is at or before the reporting clock. Otherwise idle ≤ 60 minutes is `active`. Otherwise furthest ≥ `booking_started` is `abandoned`. Otherwise `browsed`.

### `reg_flow`

- **Grain.** One `reg_flow_id`.
- **Stages.** `registration_started`, `otp_viewed`, `otp_verified`, `signup_viewed`, `registration_completed`. OTP submit, fail, and resend are attempts.

### `service_interest`

- **Grain.** Latest journey for one identity and one service.
- **Bands.** `post_purchase` if cancelled. `converted` if converted or completed. `very_high` if checkout and last activity within 24 hours. `high` if details-or-later within 7 days. `medium` if booking started, or at least two `service_viewed` sessions in the 7 days before the clock, and activity within 30 days. Otherwise `low`.

### `order_fact`

- **Grain.** One order.
- **Measures.** `payment_status`, `value`, `created_at`, `cancelled_at`, `completed_at`, `service_id`, `campaign_id`.
- **Derived events.** On success, emit `purchase_completed`. Emit `order_cancelled` or `service_completed` when those timestamps are at or before the clock. Do not emit purchase on failure.

### Aggregates

Computed per request from the read models. Not stored as their own BigQuery tables in this design: overview, daily trend, three funnels, service performance, retention split, weekly cohorts, campaign rows, segment estimate.

## 3. Counting units

| Word on screen | What is counted | What it is not |
| --- | --- | --- |
| Active people | Distinct resolved identity with a clean, non-quarantined event in the period. Customer id if linked, otherwise app instance | Not sessions, not events |
| Sessions | Distinct `session_id` on those events | Not journeys |
| Service views | Meaningful `service_viewed` rows | Not unique viewers |
| Journeys | Rows whose `started_at` is in the period, after filters | Not people |
| People in a funnel | Distinct `identity_key` among those journeys | Never larger than the journey count for the same stage |
| Bookings | Succeeded orders whose `created_at` is in the period | Not the funnel’s purchase stage |
| Revenue | Sum of those order values, USD | Demo promos do not change the price |
| Registration flows | Flows whose `started_at` is in the period | Not booking journeys |

Meaningful excludes quarantined rows, technical duplicates, proposed events, and `app_opened`.

## 4. Metric formulas

Filters that apply, unless a row says otherwise: period, service, platform, app version, audience, campaign. Audience `registered` drops guests. `guest` keeps only unresolved identities. `existing_customer` keeps customers whose first succeeded order is before the period start. That last definition is proposed.

**Previous period.** The same number of calendar days, ending when the current period starts. Delta is `(current - previous) / previous`. If the previous value is 0 and the current is not, delta is null rather than an infinite percent. If both are 0, delta is 0.

### Active people, sessions, service views

Formulas are the counting units above. Date basis is event time. `app_opened` does not add to “meaningful” activity, but it still counts toward active people and sessions because those use every non-quarantined clean event.

### Booking conversion

`purchase journeys / service-viewed journeys` among journeys started in the period. Null when the first stage is 0. Unit is journeys (or people, if view-by is customers). This is not bookings ÷ active people.

### Booking funnel

A journey is counted at every stage up to its furthest stage. If the event for an intermediate stage is missing, the stage is still counted and `imputed` increases.

For stage `i` after the first:

- `fromPrevious = count[i] / count[i-1]` (null if the previous count is 0)
- First stage `fromPrevious` is 1 when its count is positive
- `dropped = count[i] - count[i+1]` when the series does not rise
- `droppedPct = dropped / count[i]`
- Last stage has no drop-off
- If the next count is higher, `increase` is set and `dropped` is null

Overall conversion is last stage ÷ first stage.

Median minutes from the previous stage uses journeys that reached the later stage and have both meaningful events, in order. Sample size is `timingSample`.

### Registration funnel

Same drop-off math over registration stages. Counting unit is flows, or people if view-by is customers. OTP attempts are not stages. A service filter is ignored and returned in `filtersIgnored`.

### Acquisition funnel

Counting unit is people.

- `all_opens`: anyone with `app_opened` in the period.
- `first_opens`: people whose earliest `app_opened` falls in the period.

Registration steps count only flows started in the period. Booking steps count only journeys started in the period. A later step may exceed an earlier one when returning customers book without registering in the window. The UI shows the increase. Ad click is not a step, because it is not an in-app event. The service filter is ignored.

### Checkout without purchase, payment failed, cancelled, completed

Among journeys started in the period:

- Checkout without purchase: furthest stage is at least checkout, and payment outcome is not `succeeded`. Failures are included.
- Payment failed: `paymentOutcome = failed`.
- Cancelled / completed: journey status.

### Time between steps

Median minutes, as on the booking funnel. Service performance also reports median minutes from `service_viewed` to `purchase_completed` for journeys that purchased.

### First-time versus returning

Among customers with at least one succeeded order in the period (order time): returning if their first succeeded order is before the period start. Repeat rate is returning ÷ those customers. This is customers, not orders. Customers with a purchase cannot exceed the bookings count, because each of them has at least one order.

### Cohorts

**Demo definition, labeled on screen.** Week starts Monday. A customer is in the cohort whose week contains their first succeeded order, considering orders that pass the filters. The cohort is included when that week overlaps the reporting period.

Week N retention is the share of that cohort with another succeeded order in the calendar week N weeks later. A week that has not ended before the reporting clock is null, not zero.

**Production definition this does not replace.** The automation guide groups by first-order month and measures later months. Switching the demo to months would leave most cells empty inside 90 days.

Limitation: a service filter defines “first order” inside that service, not the customer’s lifetime first order.

### Campaigns

Last-touch campaign stored on the journey from the 7-day client touch. One campaign per journey. Organic is a null id.

- App opens: distinct people with `app_opened` for that campaign in the period.
- Registrations: completed registration flows in the period.
- Journeys and purchases: journeys started in the period; purchases are those with a succeeded payment.
- Journey conversion: purchases ÷ journeys.
- Revenue: sum of those journeys’ order values.

Not computed: holdout lift, coupon-redemption credit, 24-hour push credit, spend, CAC, ROAS.

### Segments

Population: identities with a clean event in the filtered period. Include-rules combine with AND or OR. Any matching exclude-rule removes the identity. An empty include list matches the whole population.

| Field | Match |
| --- | --- |
| `service` | A journey in the period for that service id |
| `behaviour` | A meaningful event of that name. Optional `windowHours` measures back from the reporting clock |
| `journey_status` | A journey in the period with that status |
| `furthest_step` | `IS` means furthest equals the stage. Other ops mean furthest is at least that stage |
| `customer_type` | `guest`, `registered`, or `existing_customer` |
| `session_count` | Distinct sessions on the person’s events in the period. `AT_LEAST`, `AT_MOST`, `EXACTLY` |
| `campaign` | Journey or event campaign id |
| `crm_segment` | Lifetime flag: `one_order_only`, `repeat_customer`, `lost_90d` |

`estimatedCount` is people. `reachableByPush` is the subset with a customer id and marketing push on. `saved` is false. The preview does not write a segment, export ids, or arm a campaign.

### Lifecycle mix

People active in the period. Guests are `visitor`. Customers use succeeded orders as of the clock:

- No order: `registered_no_order`
- Last order ≥ 90 days ago: `lapsed`
- Last order ≥ 32 days ago: `at_risk` (proposed stand-in for 1.5 × a 21-day gap)
- Else 3 or more orders in 90 days: `loyal`
- Else 2 or more lifetime orders: `repeat`
- Else: `first_order_done`

`won_back` from the automation diagram is not assigned.

## 4a. v2.1 additions (UI of 7 Oct 2026)

Screen behaviour: [Journey_Tracking_UI_Flow_v2.1.md](../requirements/Journey_Tracking_UI_Flow_v2.1.md). The sections above describe the earlier prototype; where they disagree with this section, this section wins.

### Dropped versus Abandoned

- **Dropped here (stage `i`, period P)** = identities that reached stage `i` in P and did not reach stage `i+1` in P. A report; depends on the date range. Reached counts are cumulative as in the booking funnel (furthest ≥ stage).
- **Continued** = `count[i+1] / count[i]`. The rate pill and the table column.
- **Abandoned** = status of one journey, evaluated now: furthest step `s`, no next step, and `now − last_activity_at ≥ rule[s]`. Otherwise *Not abandoned yet*. A purchase clears it at once. `abandoned_for = now − (last_activity_at + rule[s])` is shown as "Abandoned · 35 min" in the sense "idle 35 min" (Figma shows time since last activity; use `now − last_activity_at`).
- Rules (`abandon_rule`, admin editable): `service_viewed` 24 h · `booking_started` 1 h · `details_completed` 1 h · `slot_selected` 1 h · `checkout_started` 30 min · `payment_failed` 30 min.
- A rule change recomputes status for open journeys only. Funnel counts do not change.
- The earlier single threshold `abandon_after_minutes = 60` and the `browsed` status are replaced (see matrix N3).

### Recoverable now

Open value of journeys that are Abandoned, not purchased, whose customer is reachable by push or CS. The KPI card shows the count of customers and the sum of `booking_value`.

### Counting units added

| Word on screen | What is counted |
| --- | --- |
| People / customers | Resolved customer id. A customer appears once per stage |
| Devices | App instance id with no linked customer. Before sign-up a person is a device. Merged into the customer when `identity_link` is created |
| Journeys | As in section 3 |
| Reach tiles | Push + CS contact + Ads only = list count. Push: registered, push token valid, marketing allowed, not suppressed. CS contact: has phone, not push-reachable, not suppressed. Ads only: guests / devices. Suppressed people stay in the count, are tagged, and are in no tile |

### Registration funnel

Stages: `registration_started` (page viewed) → OTP requested (backend `OTP_SENT`) → `otp_verified` → `signup_viewed` → `registration_completed`. Counting unit: `reg_flow_id` (devices) or customers.

**Why OTP fails** partitions the dropped set of the OTP-requested stage, each flow in exactly one reason, first match wins:

1. SMS not delivered: gateway reported undelivered.
2. API / gateway error: send call or gateway returned an error code.
3. Blocked after 3 wrong tries: attempt count ≥ 3 and a lock.
4. Wrong code entered: at least one `otp_failed` and not blocked.
5. Code expired or left: none of the above.

The five counts must add up to the dropped count.

### Acquisition funnel

Nine stages; *Ad clicked* comes from `ad_stat_daily.clicks` for the filter (campaign, ad set, channel). It has no identity, so no list and no rate pill from the stage before it other than total / total. From *App opened* the unit is devices before registration and customers after. The "can rise between steps" rule stays; the UI does not show a conversion percent for a step larger than the one before.

**After purchase** (order records, buyers in the funnel's filter and period):

| Row | Count | Detail |
| --- | --- | --- |
| Payment failed, then paid | Orders with ≥ 1 failed payment and a later success | Share retried within 30 min |
| Auto promo applied | Orders where the promo was applied by the system | Versus manual codes |
| Rescheduled | Orders with ≥ 1 reschedule | Done / upcoming / paid a fee |
| Cancelled | Orders cancelled | With fee (amount kept) / free |
| Refunded | Orders refunded | Amount returned to the original payment |
| Service completed | Orders completed | % of purchases; orders still upcoming |

### Customer interest and AI summary

Interest level keeps the bands in section 2 until Marketing confirms thresholds (Figma Q2). The AI summary is generated per customer from the clean event log plus orders, with a stored `written_at`; it is a derived text, never a source for any metric.

### CS metrics

| Metric | Definition |
| --- | --- |
| Case owner | The CS user who first picks a case up from the inbox |
| Picked up (reg · existing) | Cases owned by the user, split by registered-without-order and existing customers |
| Not contacted / Due today / Overdue | Picked-up cases with no logged contact / follow-up due today / past due |
| On time | Cases closed or contacted by their follow-up time ÷ cases due |
| Attempts → reached | Logged contacts (any result) → contacts with result *Spoke to the customer* |
| Problems found | Contacts with *Found a problem?* ≠ None, by type. Weight in the CS KPI: 5% |
| Contacted by CS (first sales) | Registered customers with a sales-follow-up contact **before** their first purchase, in the same registration group and 30-day window |
| No CS contact | Same registration group, no sales-follow-up contact, bought in the app |
| First-sale conversion | Customers with a first sale ÷ customers in the group. Target shown 30% (config) |
| Revenue | Completed service value after discounts and refunds, excluding tax; deposits alone excluded. Credited to the last sales follow-up before the order |
| Repeat conversion | Existing customers ordering again within 30 days of the follow-up ÷ followed up. Extra calls in the window do not count the same order twice |
| Other sales without CS contact | First sales from Telegram, walk-in and partner referrals, kept apart |

Booking-confirmation contacts are never sales follow-ups. Rows can add up to more than the team total when a customer was contacted by two people; the team total counts each customer once. N/A = nothing to divide by. "Unavailable" = outcomes not recorded, which is not 0. Results inside the 30-day window are marked provisional.

## 5. Privacy and retention

- Raw rows stay for audit. CRM and the dashboard read the clean and read-model layers.
- Consent-off people contribute no events.
- Do not return raw event pages to the marketer dashboard. The data explorer is a test route and still paginates.
- Segment preview is not an audience export. Export for ads would need hashing and a consent check; that endpoint is not implemented.
