# Requirements traceability

Maps each requirement from the source documents to the mobile event, the analytical model, the mock API, the admin UI, and the test that locks it. Status values:

| Status | Meaning |
| --- | --- |
| Supported | The source states it, and the prototype implements that reading |
| Conflict | Two sources disagree. The prototype uses the resolution in the Conflicts section and labels it |
| Needs confirmation | A backend or product decision is still open |
| Proposed | Not in the original scope. Present so the gap is visible |
| Out of scope | Specified for a later product phase. Not built here |

The original `Journey_Tracking.docx` was not found. [Journey_Tracking.md](Journey_Tracking.md) is the functional specification used in its place. If the Word file differs from that markdown, this matrix cannot see the difference.

Mock responses set `dataSource: "mock"` and `connectedToBigQuery: false`. Nothing in the portal reads BigQuery.

---

## Conflicts and the resolution used

These were not silently merged.

| Topic | Source A | Source B | Resolution used in the prototype |
| --- | --- | --- | --- |
| Purchase event name | Functional spec §2 lists `order_created`, `payment_success`, `booking_confirmed` as conversion events | Architecture §4.4: `purchase_completed` is backend-owned and is the purchase record. `order_confirmation_viewed` is UI only | Funnel stage 6 is `purchase_completed` from the order. The other names are not emitted |
| Naming | Functional spec uses Title Case labels | Architecture requires `snake_case` event names | Stored names are `snake_case`. The UI shows the spec’s labels |
| Abandonment clock | Spec story and follow-up table: no purchase after 30–60 minutes, with 60 minutes in the worked example | Architecture: `abandon_after_minutes = 60` for status, and `journey_idle_hours = 24` for reusing a `journey_id` | Status uses 60 minutes. Journey reuse uses 24 hours. They are different clocks |
| Browsed status | Spec status list: active, abandoned, converted, cancelled, completed | Architecture adds `browsed` for a view that never started booking | `browsed` is included and labeled as the architecture addition |
| Funnel date | Spec does not say whether a journey is in “last 7 days” by its start, its latest event, or its purchase | — | **Proposed:** funnels use `journey.started_at`. Activity KPIs use event time. Bookings and revenue use order time |
| Existing customer | Spec filter “Existing Customer” is undefined | — | **Proposed:** first succeeded order is before the period start |
| Cohort grain | Automation guide: monthly cohorts on first-order month | A 90-day demo cannot show a useful monthly grid | **Proposed for the demo:** Monday-start weeks. Production text in the UI still says the guide is monthly |
| Campaign credit | Architecture client rule: last touch stored for 7 days | Automation §9.5: coupon redemption, else last push open within 24 hours, else organic. Lift comes from a holdout | Prototype uses the 7-day last touch, one campaign per journey. The dashboard states it is not the playbook rule and not holdout lift |
| `payment_failed` | Not in the architecture dictionary | [backend-request-journey-tracking.md](backend-request-journey-tracking.md) asks whether to whitelist it | **Proposed.** Present in the demo, flagged `proposed`, not a funnel stage, not in meaningful counts |
| Legacy event names | Architecture quarantines unknown names | Mobile request: do not quarantine the ~40 legacy events, because they carry `event_id` | Prototype quarantines unknown names, including `legacy_view_content`. The backend has agreed to skip them without quarantine; the prototype is unchanged |
| `X-App-Instance-Id` | Architecture §4.6 asks for the header on OTP, register, login, and create-order | Mobile contract: the header is not sent; Firebase `user_pseudo_id` is the instance id | Prototype keys guests on the app instance id already on the event. No header is required |
| Interest score | Spec §6: furthest step + recency + repeat sessions, no 0–100 score | Thresholds such as “recent” are guidance, not numbers | Numeric windows are proposed: very high = checkout within 24 hours; high = details or later within 7 days; medium = booking started or 2+ view sessions, within 30 days |
| At-risk lifecycle | Automation guide: 1.5 × the median reorder gap for that service | No per-service median is in the demo data | **Proposed stand-in:** at risk at 32 days, lapsed at 90 days, loyal at 3 succeeded orders in 90 days |

---

## v2.1 conflicts and new questions (added 8 Oct 2026)

Source: [Journey_Tracking_UI_Flow_v2.1.md](Journey_Tracking_UI_Flow_v2.1.md). The earlier conflict table above is kept; rows it settles are marked **Superseded**.

### Earlier conflicts settled by v2.1

| Topic | Earlier state | v2.1 resolution |
| --- | --- | --- |
| Abandonment clock (30 vs 60 min) | Spec said 30–60, architecture used a single `abandon_after_minutes = 60` | **Superseded.** One rule per step, admin editable: Viewed 24 h, Booking 1 h, Details 1 h, Slot 1 h, Checkout 30 min, Payment failed 30 min. `journey_idle_hours = 24` (journey id reuse) is unchanged and is a different clock |
| Dropped vs Abandoned | Not distinguished | Dropped = reporting for the date range. Abandoned = CRM status. Funnels count Dropped; push and CS use Abandoned |
| `payment_failed` | Proposed, not a stage | Needed: it drives the *Payment failed* abandon rule and the *Payment failed, then paid* line. Source of truth is the backend payment record; the client event stays a diagnostic. The backend already lists `payment_failed` as accepted |
| Registration stage names | Spec: Started, OTP viewed, OTP verified, Sign-up viewed, Completed | Business names used in the UI (see mapping in Journey_Tracking.md §14.2). Dev brief names stay as event names |
| Acquisition funnel | Ad click was "not an in-app event, not a stage" | Ad clicked **is** the first stage, from Meta totals only, locked (no list) |
| Starter audiences | Six audiences built as templates | Templates hidden. Builder from a funnel list stays |
| Guest recipients | Customers only in P1 | Unchanged for push. Guests appear as **devices** in lists, exports and ad audiences, but get no push and no CS call |

### New conflicts and questions

| # | Topic | Detail | Needs |
| --- | --- | --- | --- |
| N1 | **Guest events are skipped by the importer** | [tracking-backend.md](../architecture/tracking-backend.md) §13: "Events with no resolvable registered user are skipped". v2.1 needs device-level data for guest booking, the registration funnel (people / devices), acquisition devices and pre-sign-up history merged at sign-up | Backend decision. Blocker for the Registration and Acquisition tabs and for Customer type = Guest |
| N2 | Phone saved at OTP request for people who never register | Needed for the OTP-failed CS call list. Personal data of non-customers | Legal / consent wording, retention period, masking in the portal |
| N3 | `browsed` status | Not shown in v2.1. The Viewed rule (24 h) makes a viewer-who-left *Abandoned* | Recommendation: drop `browsed` as a status and report it as *Abandoned at Service viewed*. Confirm with Marketing, since the spec says no hard sell for viewers |
| N4 | Ad clicked data source | Meta totals per campaign / ad set per day. Not in the system | Meta Marketing API import (spend, clicks, impressions) and a daily table. Also needed for CAC / ROAS |
| N5 | Event log needs events that are not in the dictionary | Banner clicked, search performed, search result clicked, reschedule started, cancellation started, cancellation fee shown. `banner_clicked` and `order_rescheduled` are sent today and skipped by the backend | Whitelist decision, app work if missing |
| N6 | Clarity recordings per session | Needs the Clarity SDK in the app and a way to map a session to a recording (`ga_session_id` as a Clarity custom tag, or the reverse) | App + Marketing decision |
| N7 | AI summary | Model, refresh rule, languages open (Figma Q5). Personal data in the prompt, cost per refresh, caching | Product decision; send no phone or email to the model |
| N8 | Figma frame `1:27128` | Still shows template cards although the review hid them | Ask design to update. Notes are treated as authoritative |
| N9 | Default counting unit | Spec §3: unique journeys by default. Booking tab in v2.1 counts **customers** by default, with a Journeys toggle | Follow the UI (customers default). Update API default `view_by` for this page |
| N10 | Weekly push limit vs automation caps | UI shows 3 per week; the automation guide proposes 1 promo per day and 3 per week | One shared cap in the decision service; recovery pushes count toward it. Value open (Figma Q1) |
| N11 | Meta Custom Audience sync | Matched by phone or device, daily. Hashing and consent rules apply | Marketing + legal; backend job; Setup → Integrations screen |
| N12 | Naming clash for CS pages | Portal already has `/customer-overview` ("Overview") and `/customer` | Name the new page CS Overview and decide route (see `bEasy-portal/TODO.md`) |
| N13 | CS targets | 30% conversion target shown; who edits it is open (Figma Q3) | Product decision |
| N14 | Interest thresholds | Very high / High / Medium wording used in lists; thresholds open (Figma Q2). Proposed windows in this matrix (24 h / 7 d / 30 d) stay as the working default | Marketing |

---

## Matrix

| Source | Requirement | Mobile event or data | Model | API | Admin UI | Test |
| --- | --- | --- | --- | --- | --- | --- |
| Spec §1 | Track service viewed through service completed, with user, service, time, and journey or session | `service_viewed` … `service_completed` | `event_log`, `journey`, order facts | `listEvents`, `listJourneys`, `listOrders` | Data explorer Events, Journeys, Orders | Worked example, cancel, complete |
| Spec §1 | Service id is the stable key; name is display | `service_id` | `service` reference | Filter `serviceId` | Service filter and service table | Service filter test |
| Spec §2 | Keep raw events; do not let technical duplicates inflate CRM counts | `event_id`; 2-second same-name flag | Raw export vs clean; `excludedFromMeaningful` | `listEvents?layer=raw\|clean`, `getDataQuality` | Raw / clean toggle and quality flags | Duplicate event id test |
| Spec §2 | Backend is the source of truth for purchase, cancel, payment, completion | Backend rows, `derived: true` | Orders synthesise those three events | Journey detail includes the order | Explorer notes “Backend derived” | One purchase from one order |
| Spec §2 | No street address in analytics | `zone_id` or `area_cell` | Not a column of street text | — | Not shown | Needs confirmation: `zone_id` is not in the address API yet |
| Spec §3 | Six-stage booking funnel, unique journeys by default, unique customers optional | Stages through `purchase_completed` | Furthest stage, monotonic imputation | `getBookingFunnel` | Booking funnel, view-by control | Mini-world counts `[3,1,1,1,1,1]`; demo monotonic; customer ≤ journey |
| Spec §3 | From-previous rate, drop-off count and percent, overall conversion | Derived | `withDropOff` | Funnel stage fields | Funnel bars | Dropped 2 and 2/3 on the mini-world |
| Spec §3 | Filters: service, period, audience, view by | — | `AnalyticsFilters` | Every envelope echoes `filters` | Filter bar | Service filter changes booking only |
| Spec §4 | Furthest step is per journey and service, not a lifetime field | Highest stage on `journey_id` | `journey.furthest` | Journey rows | Journey table and drill-down | Skip journey imputes the gap |
| Spec §4 | Status separate from furthest step | Calculated | `journey.status` | Journey rows | Status badge | Active checkout vs morning browse |
| Spec §5 | Event log grouped by journey, with time, event, service, details | Clean events | `event_log` | `getJourneyDetail` | Explorer timeline | Story journey timeline data is the same world as the dashboard |
| Spec §5 | Filters for service, event, date, status | — | Explorer query | `listEvents`, `listJourneys` | Explorer filters | Date window covered by period tests |
| Spec §6 | Interest from furthest step, recency, and repeat sessions. No 0–100 score | Derived | `service_interest` | `getFollowUp` | Follow-up table | Story journey is very high and abandoned, not a purchase |
| Spec §7 | Audience rules with AND / OR / exclude and an estimated count before any save | Derived | Segment evaluation over the filtered population | `previewSegment` | Segment builder | AND, OR, exclude; lost customer and repeat viewer |
| Spec §7 | Saved segments and CRM flags combined with behaviour | `one_order_only`, `repeat_customer`, `lost_90d` | Person CRM flags, lifetime | `crm_segment` rule | CRM condition | Lost-90d customer included, then excluded |
| Spec §8 and §14.5 | Ad → register → checkout → no purchase, then abandoned | Story fixture | Journey `journey-story-checkout`, customer `1102` | Journey detail | Both routes can open that journey | Worked-example test |
| Spec §9 | Follow-up action per journey situation, with a do-not | Derived | Seven situations, each an audience rule | `getFollowUpSituations` | Analytics · Follow-up playbook, counts equal the builder's | Each count equals `previewSegment` for the same rules |
| Spec §10 | Six retargeting audiences | Derived | `STARTER_AUDIENCES` | `previewSegment` | Analytics · Audiences, "Edit these rules" | Every starter audience runs |
| Spec §11 | Re-check order, tokens, suppression, caps, deep link, promo before sending | — | Backend only | — | Audiences · "Checked before sending": only the push-reachable count is computed | Needs backend. No send path |
| Spec §14.6 | Drop-off point, meaning, action, retargeting use | Derived from funnels | `registrationDropOffs` | `getRegistrationDropOffs` | Analytics · Sign-up "Where people stop" | Counts never negative; ad click is not measured |
| Spec §14.8 | CRM tells apart six states | Derived | `CrmStage` | `getPeople` | Data · Customers CRM stage | All six present in the demo |
| Spec §14 | Registration funnel, separate from booking | Registration events | `reg_flow` | `getRegistrationFunnel` | Registration funnel | Service filter ignored |
| Spec §14.3 | Acquisition funnel from app open to purchase | `app_opened` plus both funnels | Identity counts | `getAcquisitionFunnel` | Acquisition section, all opens vs first opens | Ad click is documented as not an in-app event. UI shows non-monotonic increases |
| Spec §14.4 | Link anonymous history without rewriting it | Identity link, 30-day claim | Raw customer id vs resolved id | Raw vs clean identity columns | Explorer identity column | Story events resolve to `1102` |
| Architecture §4.4 | Full event dictionary, including add-ons and push | Catalog | `EVENT_CATALOG` | `getSchema` | Schema tab | Add-on and push events are `inDemo: false` |
| Architecture §5 | Raw BigQuery, clean extract, quarantine, then journey read models | Pipeline | See the analytical model | Mock compile, not BigQuery | Banner says mock | `connectedToBigQuery` is false |
| Architecture §6–7 | Portal APIs for funnels, audiences, actions, export | — | Read models | Mock covers the read side only | Test routes | Audience save, export, and actions are out of scope |
| Automation §4.2 | Lifecycle stages | Derived from orders | `lifecycle` | `getRetention.lifecycle` | Lifecycle list | Thresholds marked proposed |
| Automation §9 | Cohort retention | Succeeded orders | Weekly demo cohorts | `getRetention.cohorts` | Cohort grid | Rate equals retained / size; unfinished weeks are null |
| Automation §9.5 | Holdout and playbook attribution | Not in the demo | — | Campaign attribution string | Campaign note | Out of scope |
| Backend request | `zone_id`, push ids, legacy-event policy, guest events | Partial | Area cell not generated | — | — | `zone_id` stays as is, push ids deferred, legacy events skipped by the backend; guest events open (now N1, a blocker) |
| UI v2.1 §3 | Funnel to list: every Reached / Dropped number opens people | Journey state per stage | Stage membership | `GET /funnels/{f}/stages/{stage}/people` (new) | Stage sheet and full page | Row count equals the number |
| UI v2.1 §5.6 | Per-step abandon rules, admin editable, purchase clears | Derived status | `abandon_rule`, `journey_state.abandoned_at` | `GET/PUT /abandon-rules` (new) | Abandon rules screen, list strip | Rule change affects lists, not funnel counts |
| UI v2.1 §3.2 | OTP failure reasons (5) | Gateway responses, `otp_failed`, `otp_verified` | `otp_attempt` | Registration funnel `reasons[]` (new) | Why OTP fails card | Five reasons add up to the dropped count |
| UI v2.1 §3.3 | Acquisition 9 steps, Ad clicked locked, After purchase | Meta totals, order records | `ad_stat_daily`, order facts | Acquisition funnel `afterPurchase` (new) | Acquisition tab | Ad clicked has no list |
| UI v2.1 §4 | Reach tiles and suppression | Consent, push token, suppression | `marketing_suppression` | List response `reach` (new) | Tiles and footer | Tiles add up to the count |
| UI v2.1 §5.1 | Bulk push with live eligibility, weekly cap, re-check | Orders, caps | Decision service | `POST /actions/recovery-push/preview` and `/send` (new) | Confirm dialog | Just-bought people dropped |
| UI v2.1 §5.2, §5.5, §7 | CS inbox, contact log, problems found, CS overview | Contact log | `cs_case`, `cs_contact` | `/cs/cases`, `/cs/contacts`, `/cs/overview` (new) | Send to CS, Log contact, CS Overview | First owner owns; confirmation calls excluded |
| UI v2.1 §6 | Customer page: AI summary, canvas, sessions, Clarity, event log with orders | Event log + order events | `ai_summary`, `session_recording` | `/customers/{id}/summary`, `/journey-canvas`, `/event-log` | Customer page | Guest history merged at sign-up |
| UI v2.1 §5.7 | Audiences from a funnel list; Push, Meta, TikTok, CSV | Rule tree | `audience` (+ sync status) | `/audiences` with `destinations[]` | Audiences tab | Name unique; saving sends nothing |

---

## Supported by the documents and the prototype

Booking funnel, registration funnel, acquisition funnel, furthest step, journey status, interest bands, raw versus clean, backend purchase, identity stitching, period comparison, service and audience filters, segment preview, campaign last-touch, and the worked example.

## Missing or ambiguous

- Word baseline missing, so this matrix cannot prove parity with `Journey_Tracking.docx`.
- “Existing customer”, funnel date basis, and interest time windows are not numeric in the spec.
- Per-service median reorder gap is not computable from the published documents alone.
- Whether registration events before `setUserId` are kept is asked in the backend request and not answered there. The prototype keeps them and links them.

## Needs backend confirmation

- Return `zone_id` on address APIs: **not planned for now**; the app sends `area_cell` and the backend must not quarantine such events.
- Put `notification_id`, `send_id`, and `action_id` on push payloads: **deferred to the notification phase**.
- ~~Decide whether unknown legacy event names are quarantined~~: **resolved**, the backend skips them.
- Map `option_id` / `pair_service_id` / `addon_id` to backend product IDs ([pending-request-backend.md](../backend-request/pending-request-backend.md) §1).
- Confirm `user_pseudo_id` is retained on the BigQuery import so guest linking needs no new header.
- Confirm only create-order validates `X-Journey-Id`: **confirmed**, and the backend is asked to read it on the other booking endpoints too ([pending-request-backend.md](../backend-request/pending-request-backend.md) §2).
- Whitelist decision for `payment_failed` and the add-on events.

## Proposed, not original scope

`payment_failed` as a client event; imputed missing funnel steps; weekly cohort grain; the 32-day at-risk stand-in; the data explorer as a developer tool; the error-state preview toggle.

## Explicitly not built

Playbooks, offers, holdout experiments, frequency caps and suppression (listed in the UI, enforced by the backend), audience save and export, ad spend and ROAS, supply-and-demand by zone, ABA Mini App inbox, and a live BigQuery or Postgres connection.
