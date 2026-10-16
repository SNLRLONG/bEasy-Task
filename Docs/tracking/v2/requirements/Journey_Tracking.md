# bEasy Customer Journey Tracking

**Developer Functional Specification — Funnel, Event Log, Interest Level & Retargeting Flow**

| Objective: combine historical CRM data with real-time customer behaviour to understand where each user is in the journey, measure funnel drop-off, identify service interest, create retargeting audiences, and enable Sales/Marketing to follow up with the right action. |
| --- |

> **Updated 8 Oct 2026 to follow the UI/UX v2.1 handoff (business review of 7 Oct).** Screen behaviour is described once in [Journey_Tracking_UI_Flow_v2.1.md](Journey_Tracking_UI_Flow_v2.1.md). This document keeps the business rules. Sections changed by the review are marked **[v2.1]**; the list below is the summary.

# 0. What changed in v2.1

| # | Change | Where in this document |
| --- | --- | --- |
| 1 | **Dropped here** (reporting) and **Abandoned** (CRM status) are two different things. Abandoned uses an admin-set inactivity time **per step**, not one global time | §4, §9 |
| 2 | Booking funnel keeps six stages. Lists show *Abandoned · 35 min* or *Not abandoned yet · 5 min*. Recovery push goes only to Abandoned | §3, §4, §11 |
| 3 | Registration funnel step names are the business names: Registration page viewed → OTP requested → OTP verified → Sign-up started → Registration completed. A **Why OTP fails** breakdown is added | §14.1, §14.2 |
| 4 | Acquisition funnel is nine steps (adds Details completed and Slot selected) and starts at **Ad clicked** (Meta totals only). A new **After purchase** phase reads order records | §14.3 |
| 5 | Every funnel number opens a customer list; the list is the place to act (push, CS inbox, export, audience) | §9 |
| 6 | Customer page is one page with tabs, an AI summary, a journey canvas, device info and a Clarity recording link per session | §5, §6 |
| 7 | CS has no pre-assignment: **Send to CS inbox**, whoever picks a case up owns it. A CS overview measures first sales, repeat sales, follow-up work and *problems found* | §15 |
| 8 | Audience **templates are hidden**. Audiences are built from a funnel list and can be made available to Push, Meta, TikTok or CSV | §7, §10 |
| 9 | Customer type filter: All · Registered, no order yet · Existing customer · Guest · Saved segment. Before sign-up, people are counted as **devices** | §3, §14 |

# 1. Core Event Tracking

**Track the following events against User ID and Journey/Session ID. **Service ID should be the stable key; service name is for display only.

| **#** | **Event** | **Minimum data to save** | **Purpose** |
| --- | --- | --- | --- |
| 1 | Service Viewed | User ID + Service ID + Timestamp + Journey/Session ID | Shows initial service interest |
| 2 | Booking Started | User ID + Service ID + Timestamp + Journey/Session ID | Shows intent beyond browsing |
| 3 | Booking Details Completed | User ID + Service ID + Timestamp + Journey/Session ID + Estimated Value | Shows qualified booking intent |
| 4 | Slot Selected | User ID + Service ID + Timestamp + Journey/Session ID + Selected Booking Date/Time | Shows high booking intent |
| 5 | Checkout Started | User ID + Service ID + Timestamp + Journey/Session ID + Booking Value | Shows very high purchase intent |
| 6 | Purchase Completed | User ID + Service ID + Timestamp + Journey/Session ID + Order ID + Final Value | Conversion |
| 7 | Booking Cancelled | User ID + Service ID + Timestamp + Order ID + Value | Post-purchase outcome |
| 8 | Service Completed | User ID + Service ID + Timestamp + Order ID + Final Value | Actual fulfilled order |

# 2. Supporting Event Categories

| **Category** | **Events / parameters** |
| --- | --- |
| Discovery | app_opened; service_list_viewed; service_viewed |
| Booking intent | booking_started; address_selected; date_selected; time_selected; duration_selected; service_option_selected |
| Strong intent | booking_summary_viewed; promo_applied; checkout_started |
| Conversion | order_created; payment_started; payment_success; booking_confirmed |
| Post-booking | order_cancelled; service_completed |
| Marketing attribution | source; campaign; ad/campaign ID where available; promo_code |

### Important:

Raw analytics events must be retained, but duplicated technical firing should not inflate CRM activity counts.

bEasy backend/order data is the source of truth for purchase, cancellation, payment and service completion.

Full street address should not be duplicated into analytics; use an internal address/zone reference where needed.

# 3. Funnel — How It Should Be Displayed

**Funnel = aggregated journey performance across users/journeys. **It should show the number of unique journeys reaching each stage, the conversion rate from the previous stage, and drop-off.

| **Service Viewed** | **Booking Started** | **Details Completed** | **Slot Selected** | **Checkout Started** | **Purchase Completed** |
| --- | --- | --- | --- | --- | --- |

↓

| **Funnel stage** | **Definition** | **Display** | **Key use** |
| --- | --- | --- | --- |
| Service Viewed | Unique journey viewed service | Count + 100% baseline | Service interest / service-page reach |
| Booking Started | Journey entered booking flow | Count + % from viewed | Measures service-page → booking conversion |
| Booking Details Completed | Required booking/service inputs completed | Count + % from previous | Detects form/details friction |
| Slot Selected | Date/time selected | Count + % from previous | Strong intent / availability friction |
| Checkout Started | Reached checkout/payment | Count + % from previous | Very high intent |
| Purchase Completed | Order successfully created/confirmed | Count + % from previous + overall CVR | Final conversion |

## Recommended funnel controls **[v2.1]**

Service: All / specific service

Period: Today / Last 7 / 30 / 60 / 90 days / Custom, with compare to the previous period

Customer type: All / Registered, no order yet / Existing customer / Guest / Saved segment. Guests are shown as devices

Count by: Customers (default in the approved UI) / Journeys (one customer once per service visit). Lists always show people

Show drop-off between stages: count and percentage

Each stage shows a **Reached** number and a **Dropped here** number. Both open the customer list behind them.

**Dropped here** = reached the step in the selected date range and did not reach the next one. It is a report, not a status. It is separate from **Abandoned** (section 4).

## Example Funnel Display

| **Stage** | **Journeys** | **From previous** | **Dropped before next stage** |
| --- | --- | --- | --- |
| Service Viewed | 1,000 | 100% | 580 (58%) |
| Booking Started | 420 | 42% | 151 (36%) |
| Details Completed | 269 | 64% | 59 (22%) |
| Slot Selected | 210 | 78% | 50 (24%) |
| Checkout Started | 160 | 76% | 50 (31%) |
| Purchase Completed | 110 | 69% | — |

# 4. Furthest Step — Person-Level Journey Position

**Furthest Step = the highest funnel stage reached within a specific service journey. **It is not a lifetime customer field and should be calculated per Journey/Session ID + Service ID.

| **Level** | **Furthest Step** | **Meaning** |
| --- | --- | --- |
| 1 | Service Viewed | Browsing / low intent |
| 2 | Booking Started | Initial booking intent |
| 3 | Booking Details Completed | Qualified intent |
| 4 | Slot Selected | High intent |
| 5 | Checkout Started | Very high intent |
| 6 | Purchase Completed | Converted |

### Journey Status should be shown separately from Furthest Step:

Active — recent journey still in progress

Abandoned **[v2.1]** — the customer's last step has no next step within the **inactivity time set for that step** (admin setting, "Abandon rules"). Defaults: Service viewed 24 h · Booking started 1 h · Details completed 1 h · Slot selected 1 h · Checkout started 30 min · Payment failed 30 min. Before the time has passed the list shows "Not abandoned yet". A purchase removes the status at once. Changing a rule applies from then on; funnel counts do not change. Recovery pushes and CS follow-up go only to Abandoned customers.

Browsed — open point: the old "Browsed" status is covered by the Service viewed rule (24 h). See the requirements matrix, question N3.

Converted — Purchase Completed

Cancelled — order later cancelled

Completed — service fulfilled

# 5. Event Log / Activity — Recommended Display

**Interactions should be represented by the Event Log. **The top-level activity count is only a summary; clicking it should open the chronological journey log.

## Main customer card

| **Activity: 18 meaningful events  •  4 sessions  •  3 services  •  View Event Log →** |
| --- |

## Event Log layout

Group by date and Journey/Session ID.

Latest 2–3 sessions expanded by default; older sessions collapsed.

Each session header shows Service, session time, number of events, Furthest Step and conversion status.

Rows show Time, Event, Service and contextual Details.

Technical values such as event ID/session ID/source can be hidden under row expansion.

Add filters: Service, Event, Date Range, Journey Status.

### Example:

| **Time** | **Event** | **Service** | **Details** |
| --- | --- | --- | --- |
| 8:31 PM | Service Viewed | Instant Cleaning | — |
| 8:34 PM | Booking Started | Instant Cleaning | — |
| 8:36 PM | Booking Details Completed | Instant Cleaning | Estimated value: $30 |
| 8:38 PM | Slot Selected | Instant Cleaning | 27 Sep · 2:00 PM |
| 8:42 PM | Checkout Started | Instant Cleaning | Booking value: $30 |

Session summary: Instant Cleaning · 26 Sep, 8:31–8:42 PM · 5 events · Furthest: Checkout Started · No Purchase

# 6. Person-Level Service Interest

**Do not use interaction count alone as the interest score. **Interest should use a combination of Furthest Step + Recency + Repeat Sessions.

| **Interest level** | **Typical behaviour** | **Furthest Step** | **Recency guidance** | **Marketing meaning** |
| --- | --- | --- | --- | --- |
| Low | Viewed service once / light browsing | Service Viewed | Recent or older | Awareness; no hard sell |
| Medium | Repeated service views or Booking Started | Booking Started | Prefer recent | Interested; remind / educate |
| High | Details Completed or Slot Selected | Details Completed / Slot Selected | Recent | Strong booking intent; retarget |
| Very High | Checkout Started and no purchase | Checkout Started | Very recent | Immediate recovery opportunity |
| Converted | Purchase completed | Purchase Completed | Any | Stop acquisition push; move to post-purchase/repeat flow |

## Recommended person-level display

| **Service** | **Interest** | **Furthest Step** | **Last Activity** | **Status** |
| --- | --- | --- | --- | --- |
| Instant Cleaning | Very High | Checkout Started | 35 min ago | Abandoned |
| AC Cleaning | High | Slot Selected | 2 days ago | Abandoned |
| Deep Cleaning | Low | Service Viewed | 12 days ago | Browsed |

### Optional detail shown on click:

Views, number of sessions, first seen, last seen, latest selected slot, latest booking value

Do not create a numeric 0–100 score initially. Collect clean data first and later weight behaviours using actual conversion performance.

# 7. CRM Audience Builder Requirements

Marketing must be able to combine historical CRM segments with journey behaviour without asking Tech to query the database.

| **Filter group** | **Examples** | **Operators** | **Notes** |
| --- | --- | --- | --- |
| CRM | Saved Segment, Customer Type, Order Count, Last Order | IS / IS NOT | Uses existing saved segments |
| Service | Instant, General, Deep, AC, Fabric Care, etc. | IS / IS NOT | Use Service ID |
| Behaviour | Viewed, Booking Started, Slot Selected, Checkout, Purchase | HAS / HAS NOT | Based on meaningful events |
| Time | Last 1h, 6h, 24h, 3d, 7d, 30d, custom | WITHIN / BEFORE / AFTER | Recency is critical |
| Frequency | Viewed ≥ 2 times, sessions ≥ 2 | EXACTLY / AT LEAST / AT MOST | Use deduplicated events |
| Journey | Furthest Step, Status | IS / AT LEAST | Useful for retargeting |

Support AND / OR / EXCLUDE logic.

Show estimated audience count before saving/sending.

Allow Saved Segment + Behaviour to be combined.

# 8. End-to-End Example: Ad → App → Registration → Booking Intent → Retargeting

**This flow should be visible conceptually in CRM and usable for campaign attribution. **The objective is to connect marketing source to customer actions and follow-up.

| **Step** | **Customer behaviour / event** | **Data / system action** |
| --- | --- | --- |
| 1 | Meta Ad Click | Campaign: Instant Cleaning |
| 2 | App Open / Deep Link | Lands on Instant Cleaning |
| 3 | Guest Service View | Anonymous/session tracking |
| 4 | Registration | Anonymous journey linked to User ID |
| 5 | Service Viewed | User ID + service + timestamp |
| 6 | Booking Started | Intent begins |
| 7 | Slot Selected | High intent |
| 8 | Checkout Started | Very high intent |
| 9 | No Purchase | Journey becomes abandoned after threshold |
| 10 | Retarget / Follow-up | Push / ad / CS action |

## Example customer story

| **Time** | **Event** | **CRM interpretation** |
| --- | --- | --- |
| 10:02 | Customer clicks Meta ad | source=Meta; campaign=Instant_15 |
| 10:03 | App opens via deep link | service=Instant Cleaning |
| 10:03 | Service Viewed | anonymous/session ID recorded |
| 10:05 | Customer registers | journey linked to User ID |
| 10:06 | Service Viewed again | User ID now attached |
| 10:08 | Booking Started | Medium interest |
| 10:10 | Booking Details Completed | High interest |
| 10:11 | Slot Selected: 28 Sep, 2 PM | High interest |
| 10:13 | Checkout Started: $30 | Very High interest |
| 10:15 | Customer exits | No order |
| 11:15 | No purchase after 60 min | Journey status = Abandoned |

# 9. Potential Sales & Marketing Follow-up

| **Journey situation** | **Example rule** | **Primary action** | **Secondary action** | **Do not do** |
| --- | --- | --- | --- | --- |
| Viewed only | Service Viewed ≥2 sessions within 7d; no Booking Started | Retarget ad with service benefit / proof | Light push if eligible | Do not offer aggressive discount immediately |
| Booking Started, no details | Booking Started; no Details Completed after threshold | Push: continue booking | Retarget ad to same service | Do not send generic brand message |
| Details/Slot selected | Slot Selected; no Purchase | High-intent push with deep link | CS follow-up for selected/high-value cases | Do not send unrelated services |
| Checkout abandoned | Checkout Started; no Purchase after 30–60 min | Immediate recovery push | Retarget with exact service; optional incentive after non-conversion | Do not keep pushing after order exists |
| Purchased | Purchase Completed | Stop acquisition recovery | Cross-sell / repeat reminder later | Do not send abandoned message |
| Cancelled | Order Cancelled | CS recovery / understand reason | Targeted rebooking later | Do not treat as normal new lead |
| Completed service | Service Completed | Review / repeat / cross-sell flow | CRM lifecycle campaign | Do not continue acquisition recovery |

# 10. Retargeting Audience Examples

> **[v2.1] Templates are hidden in the UI** (they overlap with CRM segments and dynamic push). Audiences are created from a funnel list ("Create audience") with the list's rules pre-filled, then made available to Push, Meta ads, TikTok (not connected yet) or CSV. Saving never sends anything. The rules below stay as examples of what to build, and as the starting point when templates return with dynamic push.

| **Audience** | **Rule example** | **Message direction** |
| --- | --- | --- |
| Repeated Viewers | Service=Instant + Viewed ≥2 sessions in 7d + Has Not Booking Started | Explain flexibility, $15/hr value, proof |
| Booking Abandoners | Booking Started + Has Not Purchase within 24h | Continue booking / remove friction |
| High-Intent Abandoners | Slot Selected OR Checkout Started + Has Not Purchase | Strong recovery message; deep link to exact service |
| One-Order Customer + New Interest | Saved Segment=1 Order Only + Service Viewed + recent | Re-introduce bEasy with relevant service |
| Lost Customer + Current Interest | Saved Segment=Lost 90d+ + Service Viewed within 7d | Comeback message specific to service |
| Cross-sell | Service Completed=A + Viewed Service B | Relevant cross-sell instead of generic blast |

# 11. Push / Retargeting Safety Logic

Before sending, re-check backend order status. If customer has converted, cancel the pending recovery push.

Exclude invalid/disabled push tokens and manually suppressed customers.

Apply campaign-level and global marketing frequency caps.

Deep link to the relevant service or booking context; do not send high-intent users back to the home page.

Where promo is used, support automatic application of the selected promo code.

# 12. Recommended Build Priority

| **Phase** | **Build** | **Outcome** |
| --- | --- | --- |
| P0 | Event schema, Service ID, Journey/Session ID, deduplication, source/campaign attribution | Reliable data foundation |
| P0 | Customer Event Log grouped by journey + Furthest Step + Journey Status | Readable person-level journey |
| P0 | Funnel with stage counts, stage conversion and drop-off | Understand where customers leave |
| P0 | Person-level service interest using Furthest Step + Recency + Repeat Sessions | Identify high-intent customers |
| P1 | CRM audience builder combining Saved Segments + Behaviour + Time + Frequency | Marketing self-service |
| P1 | Push/deep-link/promo/exclusion engine | Behavioural recovery campaigns |
| P1 | Push/ad → order attribution | Measure bookings and revenue |
| P2 | Automated triggers, A/B tests, best-send-time, advanced scoring | Optimisation after clean data accumulates |

# 13. Developer Acceptance Checklist

☐ Each service event contains User ID (where known), Service ID, timestamp and Journey/Session ID.

☐ Anonymous pre-registration activity can be linked to User ID after registration/login where technically and legally appropriate.

☐ Duplicate technical events do not inflate CRM event counts.

☐ Furthest Step is calculated per journey/service, not lifetime customer history.

☐ Funnel uses unique journeys by default and shows stage conversion/drop-off.

☐ Event Log is grouped by session/date and supports filtering.

☐ Interest level uses Furthest Step + Recency + repeat sessions, not raw event count alone.

☐ Purchase/cancellation/completion status comes from bEasy backend.

☐ CRM audience builder supports Saved Segment + Behaviour + Has/Has Not + Time Period + Frequency + AND/OR/EXCLUDE.

☐ Marketing can trace source/campaign → app → registration → service journey → purchase.

# 14. Registration Journey & Acquisition Funnel (Added)

**Registration must be tracked as a multi-step journey, not as one single event. **This is required so Marketing/Product can identify whether users drop at OTP verification or at the sign-up form.

## 14.1 Registration Events

| **Stage** | **Event** | **Minimum data to save** | **Purpose** |
| --- | --- | --- | --- |
| Registration Started | registration_started | Anonymous/Session ID + Timestamp + Source/Campaign | User entered registration flow |
| OTP Page Viewed | otp_viewed | Anonymous/Session ID + Timestamp | User reached OTP page |
| OTP Submitted | otp_submitted | Anonymous/Session ID + Timestamp | Verification attempt made |
| OTP Verified | otp_verified | Anonymous/Session ID + Timestamp | OTP successfully verified |
| Sign-Up Page Viewed | signup_viewed | Anonymous/Session ID + Timestamp | User reached account/profile sign-up page |
| Registration Completed | registration_completed | User ID + Session ID + Timestamp | Registered account successfully created |

## 14.2 Registration Funnel

**Display as a separate funnel: **Registration page viewed → OTP requested → OTP verified → Sign-up started → Registration completed. **[v2.1]** These are the business step names used in the approved UI. Mapping to events: page viewed = `registration_started`; OTP requested = backend `OTP_SENT` (the send-OTP call, which is also where the phone number is saved); OTP verified = `otp_verified`; sign-up started = `signup_viewed`; completed = `registration_completed`. `otp_viewed` stays as a diagnostic event.

Counted as **people / devices**: before sign-up a person is a device.

**Why OTP fails [v2.1]** — for those who requested an OTP and never verified: SMS not delivered, API / gateway error (both from the SMS gateway response), wrong code entered, blocked after 3 wrong tries (both from app events), code expired or left (no verify and no failure). Each reason opens the list. The dropped list shows masked phone, device and app version, source, OTP result, gateway code, lock and resend count. These people are not registered, so there is no push; CS can call because the phone was saved at the OTP request.

**For each stage show: **unique users/journeys, conversion from previous stage, and drop-off count/percentage.

## 14.3 Full Acquisition Funnel

**Acquisition Funnel [v2.1]: **Ad clicked → App opened → Registered → Service viewed → Booking started → Details completed → Slot selected → Checkout started → Purchased (nine steps), then an **After purchase** phase.

*Ad clicked* is a Meta total only: it is shown but cannot be opened as a list. From *App opened* on, every Reached / Dropped opens a list; before sign-up the rows are devices (device ID, app version, sessions, where they stopped), usable as an ad audience but not for push or CS.

*After purchase* is read from **backend order records**, never from screen events: payment failed then paid, auto promo applied (vs manual code), rescheduled (done / upcoming / paid a fee), cancelled (with or without fee), refunded (amount returned to the original payment), service completed. Filters: channel, campaign, ad set, service, customer (new / existing), date. A *Compare campaigns* table loads a campaign's funnel on click.

Keep this separate from the Service Booking Funnel. The Acquisition Funnel answers where paid/organic traffic is lost from campaign entry to purchase; the Registration Funnel diagnoses account-creation friction; the Booking Funnel diagnoses service-booking friction.

## 14.4 Anonymous-to-Registered Identity Linking

Before registration, record the journey against Anonymous ID / Session ID.

After Registration Completed, associate the eligible pre-registration journey with the new User ID so the CRM can show the complete path from ad/app entry to registration and booking.

Do not overwrite the original timestamps, source, campaign or service viewed.

Keep the original anonymous/session identifier for audit/debugging.

Apply the linking only where technically and legally appropriate under the product's consent/privacy setup.

## 14.5 Example: Ad → Registration → Booking

| **Time** | **Customer action** | **Tracked event/data** | **CRM interpretation** |
| --- | --- | --- | --- |
| 10:02 | Clicks Instant Cleaning Meta ad | source=Meta; campaign=Instant_15 | Campaign acquisition |
| 10:03 | App opens / deep link | app_opened + service context | App visit |
| 10:03 | Views Instant Cleaning | service_viewed | Service interest |
| 10:04 | Starts registration | registration_started | Registration intent |
| 10:04 | OTP page opens | otp_viewed | OTP stage |
| 10:05 | Submits and verifies OTP | otp_submitted + otp_verified | OTP passed |
| 10:05 | Sign-up page opens | signup_viewed | Sign-up stage |
| 10:06 | Completes sign-up | registration_completed + User ID | Registered user |
| 10:07 | Returns to Instant Cleaning | service_viewed | Known-user service interest |
| 10:08 | Starts booking | booking_started | Medium interest |
| 10:10 | Completes details | booking_details_completed | High interest |
| 10:11 | Selects slot | slot_selected | High intent |
| 10:13 | Starts checkout | checkout_started | Very high intent |
| 10:15 | Leaves without purchase | No purchase event | Pending abandonment |
| 11:15 | Still no purchase | Calculated journey status | Abandoned / retarget eligible |

## 14.6 Follow-up Opportunities by Registration Stage

| **Drop-off point** | **What it means** | **Recommended analysis/action** | **Retargeting use** |
| --- | --- | --- | --- |
| Ad clicked, app not opened | Acquisition/deep-link loss | Check campaign landing/deep-link performance | Ad platform retargeting where audience is available |
| App opened, no registration | Visitor did not enter registration | Review service page/registration gate/value proposition | Retarget service viewers |
| OTP viewed, not verified | OTP/verification friction or abandonment | Check OTP delivery, resend flow and errors | Avoid assuming registered identity |
| OTP verified, sign-up not completed | Sign-up form friction | Review required fields and form completion | Resume registration if eligible |
| Registered, no booking | Acquired user but no booking intent | Service-specific onboarding/push based on views | Registered-no-order audience |
| Booking/slot/checkout abandoned | High-intent registered user | Deep-link recovery push and relevant retargeting | High-priority recovery audience |

## 14.7 Dashboard Views to Keep Separate

| **Dashboard** | **Stages** | **Primary question** |
| --- | --- | --- |
| Acquisition Funnel | Ad/Source → App → Registration → Service → Booking → Purchase | Which acquisition traffic converts into customers/orders? |
| Registration Funnel | Registration Started → OTP → Sign-Up → Completed | Where do users fail during registration? |
| Booking Funnel | Service Viewed → Booking Started → Details → Slot → Checkout → Purchase | Where do interested users abandon booking? |

## 14.8 Additional Developer Acceptance Checks

☐ OTP page and Sign-Up page are tracked as separate registration stages.

☐ Registration Completed creates/attaches the User ID without losing the pre-registration session history.

☐ Marketing source/campaign attribution remains attached through registration and subsequent booking where attribution rules permit.

☐ Registration Funnel can report stage conversion and drop-off independently from the Booking Funnel.

☐ CRM can distinguish: anonymous visitor, registration started, OTP verified, registered/no order, booking intent, converted customer.

# 15. Customer Service, Contacts and Customer Page (Added in v2.1)

**CS inbox.** "Send to CS inbox" from any customer list. There is no pre-assignment: the case waits in one inbox and whoever picks it up owns it until it is closed. Reasons: Sales follow-up, or Help with a booking. Only sales follow-ups count in the CS overview. Customers who asked for no marketing contact are suppressed and left out.

**Log contact.** Tapping a phone number is not a call. After a call the agent records channel (phone, Telegram, in-app chat), result (no answer, busy, spoke, wrong number), what the customer said, and **Found a problem?** (payment, app bug, price or slot, partner, none). "Asked for no more calls" adds the customer to suppressed customers.

**CS overview.** Today's follow-up work per agent (picked up, not contacted, due today, overdue, on time, attempts → reached), problems found this month (5% of the CS KPI), first sales of registered customers with vs without CS contact (same registration group, 30-day window, target 30% shown), repeat sales of existing customers, and sales without CS contact (Telegram, walk-in, partner referral). Revenue = completed service after discounts and refunds, without tax, credited to the last sales follow-up before the order. The comparison does not prove the call caused the sale.

**Customer page.** Tabs Summary, Journey, Event log, Orders, Contacts. Shows device (OS and app version), an AI summary (what happened, next step, best offer; push text is a draft), a journey canvas per service, and "Watch recording" (Clarity) per session. Recovery pushes are written in English and Khmer.

## 15.1 Acceptance checks added in v2.1

☐ Dropped here and Abandoned are separate; lists show Abandoned vs Not abandoned yet.

☐ Abandon time is configurable per step by an admin, and a purchase clears Abandoned immediately.

☐ Every Reached / Dropped number opens a list whose row count equals the number.

☐ Recovery push re-checks orders and Abandoned status right before sending, and respects the weekly marketing limit.

☐ OTP failure reasons are separated (delivery, gateway, wrong code, blocked, expired or left).

☐ After-purchase outcomes (payment failed then paid, auto promo, reschedule, cancellation with fee, refund, completed) come from order records.

☐ Guest activity is kept per device, shown as devices, and merged into the customer at sign-up and login.

☐ Contacts are logged with an explicit result; a tap on a number is not counted.

bEasy Customer Journey Tracking — Developer Functional Specification

# 16. Event Coverage Checklist — Developer Brief v2 (Added 8 Oct 2026)

Source: *bEasy Consumer Journey Event Tracking – Revised Developer Brief v2* (.docx). The brief puts marketing, CRM and operational signals into one consumer journey. This section lists every event and rule in the brief and tracks what the **bEasy-app** already sends.

How to read it:

- ☑ = already sent by the app (`lib/analytics/analytics_helper.dart`). ☐ = not added yet.
- **UI** = the event needs a screen or element that does not exist in the app yet. Add the event in the same release as that UI. **Backend** = fired by the backend, not by the app. The app only needs to send the IDs the backend uses to link it.
- Tags from the brief: **[NEW]** add to the plan, **[UPDATED]** change an existing definition.
- Status checked against the app code on 8 Oct 2026. Re-check a ☑ before relying on its parameters, because the brief adds fields that older events may not carry yet (see 16.10).

## 16.1 Acquisition & app entry

| Done | Event | Key data | Owner / blocker |
| --- | --- | --- | --- |
| ☐ | `ad_click` | campaign_id, source, medium, ad_id / creative_id | Ad platform / AppsFlyer. Not an app event; confirm it reaches the journey. |
| ☑ | `first_open` **[NEW]** | anonymous_id, install source, campaign_id, source, medium | Added. Fires once per install from `trackAppOpened`, before `app_opened`. |
| ☑ | `app_opened` | entry_type (organic / deeplink / push / banner), campaign_id, source, medium | Keep. Check `entry_type` covers all four values. |
| ☑ | `push_received` **[NEW]** | notification_id, campaign_id, segment_id | Added for **foreground** pushes only (`firebase_messaging_service.dart`). Background delivery is not tracked: the background handler runs without Firebase or app state, so it needs a separate change and a device test. |
| ☑ | `push_opened` | notification_id, campaign_id, segment_id, destination | Exists (`trackPushOpened`). Brief adds `campaign_id`, `segment_id`, `destination`. |

## 16.2 Home & discovery

| Done | Event | Key data | Owner / blocker |
| --- | --- | --- | --- |
| ☑ | `homepage_viewed` **[NEW]** | entry_type | Added in `home_screen.dart`. |
| ☑ | `banner_viewed` **[NEW]** | banner_id, campaign_id, placement, service_id | Added in `BannerCarousel`: first slide on build, then each slide on page change, once per banner per carousel. |
| ☑ | `banner_clicked` **[NEW]** | banner_id, campaign_id, destination, service_id | Exists (`trackBannerClicked`) with action / target / category. Add `banner_id`, `campaign_id`, `service_id`. |
| ☑ | `service_list_viewed` | category / entry point | Exists. |
| ☑ | `search_started` **[NEW]** | entry point | Added in `home_service_search_screen.dart`. |
| ☑ | `search_performed` **[NEW]** | search_term, result_count | Added in `home_service_search_screen.dart`. |
| ☑ | `search_result_clicked` **[NEW]** | search_term, service_id, result_position | Added in `home_service_search_screen.dart`. |
| ☑ | `search_no_results` **[NEW]** | search_term | Added in `home_service_search_screen.dart`. |
| ☑ | `service_viewed` | service_id, entry_point (home / search / banner / push / deeplink) | Exists. Add `search`, `banner` and `push` entry points when those exist. |

## 16.3 Registration & login

| Done | Event | Key data | Owner / blocker |
| --- | --- | --- | --- |
| ☑ | `registration_started` | entry_point, campaign_id | Exists. |
| ☑ | `otp_viewed` | registration_id | Exists. |
| ☑ | `otp_submitted` | attempt_number | Exists. |
| ☑ | `otp_failed` | error_code, attempt_number | Exists. |
| ☑ | `otp_resend_clicked` | attempt_number | Exists. |
| ☑ | `otp_verified` | registration_id | Exists. |
| ☑ | `signup_viewed` | registration_id | Exists. |
| ☑ | `registration_completed` | method, customer_id | Exists. Must link anonymous activity to `customer_id` (see 16.9). |

## 16.4 Explore service

| Done | Event | Key data |
| --- | --- | --- |
| ☑ | `task_info_viewed` | service_id |
| ☑ | `equipment_viewed` | service_id |
| ☑ | `service_option_selected` | option_group, option, value |
| ☑ | `addon_group_viewed` | addon_group |
| ☑ | `addon_selected` | addon_id, quantity, value |
| ☑ | `addon_quantity_changed` | addon_id, old_qty, new_qty, value |
| ☑ | `addon_removed` | addon_id, value |

## 16.5 Booking

| Done | Event | Key data | Note |
| --- | --- | --- | --- |
| ☑ | `booking_started` | service_id | |
| ☑ | `address_selected` | zone / rounded area | Never send the full street address. |
| ☑ | `duration_selected` | hours | |
| ☑ | `date_selected` | date | |
| ☑ | `time_selected` | time | |
| ☑ | `booking_details_completed` | estimated_value | |
| ☑ | `slot_selected` | slot_start, value | |
| ☑ | `booking_summary_viewed` | value | |

## 16.6 Checkout & promotion

| Done | Event | Key data | Owner / blocker |
| --- | --- | --- | --- |
| ☑ | `pair_service_viewed` | paired_service_id, option, value | Exists. |
| ☑ | `pair_service_selected` | paired_service_id, option, value | Exists. |
| ☑ | `pair_service_added` | paired_service_id, quantity, value | Exists. |
| ☑ | `pair_service_removed` | paired_service_id, quantity, value | Exists. |
| ☐ | `auto_promo_applied` **[NEW]** | promo_id, promo_type, discount_value, trigger_rule, final_value | **UI**: the app has no automatic promotion yet. `promo_type` must separate auto from code. |
| ☐ | `auto_promo_removed` **[NEW]** | promo_id, removal_reason, previous_discount_value, new_final_value | **UI**: same. Fire when eligibility changes or another promo conflicts. |
| ☑ | `promo_code_entered` **[UPDATED]** | promo_code | App. Manual-code flow only. Fire when the customer submits a code, before the result. |
| ☑ | `promo_applied` **[UPDATED]** | promo_code, promo_id, discount_value, final_value, promo_source=manual | Exists. Change: never fire it for an invalid code; add `promo_source=manual`. |
| ☑ | `promo_rejected` **[NEW]** | promo_code, rejection_reason | App. Reasons: expired, not eligible, usage limit reached. The coupon list (`apply_coupon_list_screen.dart`) is the place to hook. |
| ☑ | `checkout_started` | booking_value, promo_id / promo_code, promo_source, campaign_id | Exists. Add `promo_source` (auto / manual) and `campaign_id`. |

## 16.7 Payment & order

| Done | Event | Key data | Owner / blocker |
| --- | --- | --- | --- |
| ☑ | `payment_started` | method, value | Exists. |
| ☑ | `payment_failed` | method, error_code | Exists. |
| ☑ | `payment_returned_success` **[NEW]** | method, value, transaction_ref | App. Diagnostic only; never used for revenue. Fire when the app returns from the payment provider (ABA / bank deeplink) showing success. |
| ☑ | `order_created` | order_id, value, order_source | Exists. Add `order_source` (App / Direct Sale / Package-Coupon). |
| ☑ | `order_confirmation_viewed` | order_id | Exists. |
| ☐ | `purchase_completed` | order_id, final_value, transaction_id, order_source | **Backend only.** Source of truth for conversion and revenue. Confirm the backend writes it into the journey. |

## 16.8 Reschedule, cancellation, refund, completion

The app has a reschedule sheet (`reschedule_button_click`, `reschedule_button_confirm`, `order_rescheduled`). These older events do not match the brief's names, so the new events were added next to them. **The app has no customer cancel-order screen**: the `cancel_button` event belongs to the register dialog, and `PaymentController.paymentCancelOrder` is not called from any screen. The cancellation events wait for that UI. There is also no fee display or cancellation reason picker yet.

| Done | Event | Key data | Owner / blocker |
| --- | --- | --- | --- |
| ☑ | `reschedule_started` **[NEW]** | order_id, current_slot, reschedule_source | App. Added in `order_detail_screen_v2.dart`. The old `reschedule_button_click` is kept for continuity. Source is `customer`. |
| ☑ | `reschedule_slot_selected` **[NEW]** | order_id, old_slot, new_slot | App. |
| ☐ | `reschedule_fee_shown` **[NEW]** | order_id, fee_amount, fee_rule | **UI**: no fee shown in the app yet. Fire only where fee logic applies. |
| ☑ | `reschedule_completed` **[NEW]** | order_id, old_slot, new_slot, fee_amount, reschedule_source | **Backend** is the final source. App may also send on success. Added in `PaymentController.rescheduleOrder`. The old `order_rescheduled` is kept. |
| ☑ | `reschedule_failed` **[NEW]** | order_id, failure_reason | App / backend. Reasons: slot unavailable, payment issue, policy restriction. |
| ☐ | `cancellation_started` **[NEW]** | order_id, cancellation_source | **UI**: no cancel-order screen yet. |
| ☐ | `cancellation_reason_selected` **[NEW]** | order_id, reason_code, reason_text_if_other | **UI**: no reason picker yet. |
| ☐ | `cancellation_fee_shown` **[NEW]** | order_id, fee_amount, fee_rule, payment_source | **UI**: no fee display yet. |
| ☐ | `cancellation_confirmed` **[NEW]** | order_id, fee_amount, cancellation_source | **UI**: no cancel-order screen yet. Intent signal before the backend's final status. |
| ☐ | `order_cancelled` **[UPDATED]** | order_id, value, cancellation_reason, cancellation_fee, fee_status | **Backend only.** Must separate with-fee from without-fee. |
| ☐ | `cancellation_fee_paid` **[NEW]** | order_id, fee_amount, transaction_id, payment_method | **Backend.** Needed for finance and tax reporting. |
| ☐ | `cancellation_failed` **[NEW]** | order_id, failure_reason | **UI**: no cancel-order screen yet. Reasons: policy restriction, fee payment failure. |
| ☐ | `refund_requested` **[NEW]** | order_id, refund_amount, reason | **Backend / CS tool.** No customer refund screen in the app. |
| ☐ | `refund_completed` **[NEW]** | order_id, refund_amount, transaction_id | **Backend only.** |
| ☐ | `service_completed` | order_id, final_value | **Backend only.** |

## 16.9 Identity and journey rules (mandatory)

☐ `event_id` is unique on every event (deduplication). The app sends a UUIDv4 for events in the contract; confirm it covers every new event and add the new events to `event_contract.dart`.

☐ `anonymous_id` / `device_id` persists long enough to link browsing to the later registration or login.

☐ `customer_id` is on every event once the customer is known.

☐ Identity merge: at registration or login, pre-login activity merges into the customer record (acceptance test 1 below).

☐ `journey_id` is created at `service_viewed` and reused until purchase or the agreed inactivity timeout.

☐ `registration_id` links registration and OTP events, then connects to `customer_id`.

☐ `service_id` is carried through every service stage.

☐ `campaign_id` is carried through entry, banner, push and downstream events where available.

☐ `notification_id` and `banner_id` are present on push and banner events.

☐ `order_source` is on order-level events (App / Direct Sale / Package-Coupon or other agreed values).

## 16.10 Existing events that need extra fields

These events are already sent but the brief expects more data. Confirm each in code before ticking.

☑ `push_opened`: `campaign_id`, `segment_id`, `destination` (sent when the push data carries them).

☑ `banner_clicked`: `banner_id` and `placement` now on every path (video, banner detail, CTA from the carousel, CTA inside the banner detail page). `campaign_id` and `service_id` are accepted but the banner data has no field for them, so they are not sent.

☑ `service_viewed`: `entry_point` `banner` (banner detail, popup and CTA navigation) and `push` (push opens the service) are now sent. `search` waits for the search UI.

☑ `checkout_started`: `promo_source` (always `manual` today) and `campaign_id` (the active last-touch campaign, 7-day window, when there is one).

☑ `promo_applied`: `promo_source=manual` added and no longer fired for invalid codes (they send `promo_rejected`). `final_value` is sent (the preview's `totalPayableAmount`). `discount_value` is **not** sent: the preview only has a display string for the original price, so the backend needs to return a numeric discount first.

☑ `order_created`: `order_source=app`.

## 16.11 Derived CRM statuses (computed, never fired)

The brief says not to fire "abandoned" events. These come from the last event plus a time rule (the per-step Abandon rules in section 4).

☐ Viewed but not booked: `service_viewed`, no `booking_started` in the period.

☐ Booking abandoned: `booking_started`, no `checkout_started` or `purchase_completed` after the window.

☐ Checkout abandoned: `checkout_started`, no `purchase_completed` after the window.

☐ Payment failed: `payment_failed` is the furthest step and no later `purchase_completed`.

☐ Furthest step: highest funnel milestone per journey.

☐ Registered, no order: `registration_completed` and no `purchase_completed`.

☐ Service interest: repeated service views without a booking.

☐ Rescheduled booking: `reschedule_completed` exists; show latest old slot → new slot and source.

☐ Cancellation started, not completed: `cancellation_started` and no backend `order_cancelled` after the action window.

☐ Cancelled with fee: `order_cancelled` with `cancellation_fee > 0`.

☐ Cancelled without fee: `order_cancelled` with `cancellation_fee = 0` or fee waived.

## 16.12 Admin / CRM must show

☐ Customer event timeline in time order (e.g. Push Opened → Deep Cleaning Viewed → Booking Started → Slot Selected → Checkout Started → Payment Failed).

☐ Clickable funnel; each stage opens the customer list.

☐ Filters: Service + Has / Has Not + Time Period + Frequency.

☐ Attribution by campaign, push, banner, search, ad source and order source.

☐ Revenue and conversion use backend `purchase_completed` only.

☐ Retargeting audiences built from these events and statuses, available to the Push Admin Portal and marketing exports.

## 16.13 Minimum acceptance test before release

☐ 1. Ad → browse before registering → register → book and pay: all pre-registration events appear on the final customer record.

☐ 2. Open from a push: the push `campaign_id` stays attributable through `service_viewed` and purchase.

☐ 3. Banner shown, not clicked: `banner_viewed` is still recorded.

☐ 4. Search "aircon", tap a result, book: query and selected service can be traced. *(waits for search UI)*

☐ 5. Invalid promo: `promo_code_entered` + `promo_rejected`, not `promo_applied`.

☐ 6. Reach checkout and leave: CRM shows Checkout abandoned without an abandon event.

☐ 7. Payment fails, retry succeeds: `payment_failed` and `purchase_completed` are in the same journey.

☐ 8. Paid order cancelled with a fee: fee amount and status are available for finance. *(waits for cancellation fee UI)*

☐ 9. Booking rescheduled: new and previous slot are traceable.

☐ 10. Staff and test accounts do not pollute production analytics.

## 16.14 Priority (from the brief)

**Must have before CRM funnel launch:** customer_id mapping, anonymous-to-user merge, service_id, journey_id, campaign_id, banner events, push events, search events, auto-promo and manual promo tracking, clean promo success / failure events, backend `purchase_completed`, full reschedule and cancellation tracking, derived furthest-step logic.

**Important next:** reschedule, refund, cancellation fee details, full order-source attribution.

## 16.15 Summary

| Group | In brief | Done | Not added | Waits for new UI |
| --- | --- | --- | --- | --- |
| Acquisition & entry | 5 | 4 | 1 (`ad_click`, ad platform) | 0 |
| Home & discovery | 9 | 9 | 0 | 0 |
| Registration | 8 | 8 | 0 | 0 |
| Explore service | 7 | 7 | 0 | 0 |
| Booking | 8 | 8 | 0 | 0 |
| Checkout & promotion | 10 | 8 | 2 | 2 (auto promo) |
| Payment & order | 6 | 5 | 1 (`purchase_completed`, backend) | 0 |
| Reschedule / cancel / refund / completion | 15 | 4 | 11 | 6 (cancel screen, fee display, reason picker) |
| **Total** | **68** | **53** | **15** | **8** |

Backend-only events inside "Not added": `purchase_completed`, `order_cancelled`, `cancellation_fee_paid`, `refund_requested`, `refund_completed`, `service_completed` (plus `ad_click` from the ad platform). The app team does not build these.

Implemented on branch `feature/tracking-brief-v2-events` of bEasy-app (8 Oct 2026). Not yet verified on a device or in the backend: the events compile and pass `flutter analyze`, but no event was sent to Firebase from a running build. They are not yet in `event_contract.dart`, so the contract validator does not check them.
