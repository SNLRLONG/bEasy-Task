# Plan: app tracking coverage, backend data and portal "App Usage"

**Updated:** 2 Nov 2026 · Owner: SNLRLONG (Seavlong) · Status: **planning only, awaiting review** · Spec: `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` · Inventory: `Docs/tracking/v2/architecture/App_Interaction_Inventory.md`

## Goal
0. (Added 2 Nov) Fix the Registration funnel so the portal matches the app's current order, show where each registration came from, and track every location action. Spec: `Docs/tracking/v2/architecture/Registration_And_Location_Flow_Spec.md`.
1. Every live screen, sub-screen, sheet and tappable element in `bEasy-app` reports to the right platform (Firebase, Meta, AppsFlyer, Crashlytics) without duplicating existing events.
2. Backend receives, stores and maps those events and serves aggregate APIs.
3. The admin portal shows usage, element performance, flows, search demand and app health, with filters and segmentation.

## Flow (state: exists / change / new)
| Step | State |
|---|---|
| User action → `AnalyticsHelper.track` adds `event_id`, `seq`, `schema_version`, ids | **exists** |
| Funnel events (booking, promo, checkout, registration, OTP, push, banner, reschedule) | **exists**; gaps in orders (cancellation), top-up |
| Screen views via navigator observer (page routes only) | **change**: add tabs, sheets, dialogs, web views, `screen_exited`, `entry_type` |
| Taps via ~35 ad-hoc events with no screen/element context | **change**: generic `element_clicked` + registry, legacy dual-fire |
| Errors / crashes | **new**: `app_error`, Crashlytics |
| Search, settings, permissions, support contact, chat, address management | **new** events |
| Meta: 7 events; AppsFlyer: `af_purchase` | **change**: add `Search`, `Schedule`, `Contact`, `AddToCart`; AF standard events |
| Firebase → BigQuery → backend importer; backend skips `screen_viewed` and legacy clicks today | **change**: ingest new events into separate tables |
| Backend read models + `/app-usage/*` | **new** |
| Portal Marketing → App Usage (Overview, Screens, Elements, Flows, Search, Health); Customer page "Show taps" | **new** |

## Screens and states
App screens covered, with seed element ids per screen: spec §4 (A to K). Portal: spec §7. Empty/loading/error states for the portal pages follow the RDS data-table and stat-tile patterns; "no data yet (backend not live)" state is part of the portal tasks.

## Data and API
Event model: spec §3. Backend tables, metric definitions and endpoints: spec §6. Open asks: `backend-request/pending-request-backend.md` §9. The backend repo is not cloned here, so nothing about backend behaviour was verified against code; the QA task checks the first staging batch.

## Tracking events
New: `screen_exited`, `overlay_viewed`, `element_clicked`, `form_submitted`, `search_performed`, `search_result_clicked`, `setting_changed`, `permission_result`, `support_contacted`, `app_error`, plus domain events (cancellation, extend, top-up, coupons, address, chat, share, logout). Extended: `screen_viewed`. Full parameter tables: spec §3.3 and §3.4.

## Gaps and decisions
Gaps G-1 to G-18: spec §1.3. Decisions D-1 to D-5: spec §10 (Firebase → BigQuery transport, generic events with registries, separate tables, backend owns money, no instrumenting dead screens).

Sequencing decisions:
- All new `AnalyticsHelper` methods, the registry and the observer change land in **one foundation task** so slices never edit `analytics_helper.dart` (no parallel conflicts).
- Slices split by screen group, each owning disjoint files. Shared widgets (`view/custom/*`) are assigned to exactly one slice (spec §4 K).
- Tracking-contract tasks are **T3** (backend depends on names and params); slice tasks that only call already-agreed generic events are **T2**.

## Open questions (recommendation first; none block planning)
| # | Question | Recommendation |
|---|---|---|
| O-1 | Adopt Firebase Crashlytics now | Yes, small native change, no crash visibility today |
| O-2 | Adopt Firebase Performance | Later (backlog) |
| O-3 | Retire legacy ad-hoc events | Dual-fire one release, retire after marketing confirms no GA/Meta audience depends on them |
| O-4 | Retention raw 90 d, errors 180 d, aggregates 2 y | Yes |
| O-5 | Send top-up to Meta/AppsFlyer | No until marketing defines value |
| O-6 | Who owns Khmer labels in `ref_app_*` | Marketing via portal later; seed from spec |

## Task breakdown
| ID | Task | Repo | Depends on | Tier | Parallel group |
|---|---|---|---|---|---|
| T-0062 | `Cross-tracking-coverage-spec` (this audit + spec; approve it) | cross | none | T0 | start |
| T-0063 | `App-tracking-foundation` (registry, wrapper helpers, observer, contract, tests) | bEasy-app | T-0062 | T3 | A |
| T-0068 | `App-monitoring-errors` (Crashlytics, `app_error`, no-internet/force-update) | bEasy-app | T-0062 (O-1) | T3 | A (touches `main.dart`, `pubspec.yaml`, `api_client.dart`; not `analytics_helper.dart` except one method owned by T-0063) |
| T-0070 | `Cross-tracking-backend-ingest` (send request, agree contract, staging date) | cross | T-0062 | T3 | A |
| T-0064 | `App-tracking-home-discovery` (launch, home, search, banners, notifications) | bEasy-app | T-0063 | T2 | B |
| T-0065 | `App-tracking-booking-checkout` (service, schedule, checkout, coupon, payment, top-up) | bEasy-app | T-0063 | T3 | B |
| T-0066 | `App-tracking-orders` (bookings list, order detail, cancel, reschedule, extend, review) | bEasy-app | T-0063; backend 3.3/3.4 for `cancellation_*` | T3 | B |
| T-0067 | `App-tracking-account-profile-chat` (auth gaps, profile, address, language, chat, help) | bEasy-app | T-0063 | T2 | B |
| T-0069 | `App-tracking-meta-appsflyer` (Meta + AF event additions) | bEasy-app | T-0065; T-0031 findings | T2 | C |
| T-0071 | `Portal-app-usage-analytics` (Overview, Screens, Elements, Flows, Search, Customer "Show taps") | bEasy-portal | T-0070 contract (mock data first); `Portal-marketing-foundations-ui`; RDS pages stable | T2 | D |
| T-0072 | `Portal-app-health-monitor` (Health tab) | bEasy-portal | T-0071 shell; T-0068 | T2 | E |
| T-0074 | `App-registration-tracking` (entry at every call site, `otp_mode`, `otp_code_entered`, `signup_submitted`, sign_up timing) | bEasy-app | T-0063; O-7 | T3 | B0 (before the other B slices: it touches `home_screen`, `app_screen`, `banner_detail`, `helper`, checkout gate lines) |
| T-0075 | `Cross-registration-funnel-stages` (new stage order, entry fields, backend request §10, doc fixes) | cross | T-0062 | T3 | A |
| T-0076 | `Portal-registration-funnel-fix` (stage order, labels, Entry card/filter/column, customer timeline) | bEasy-portal | T-0075 backend reply; coordinate with `Portal-rds-marketing` | T2 | D |
| T-0077 | `App-location-tracking` (list, map, search, coverage check, permissions, address CRUD) | bEasy-app | T-0063 | T3 | B0 (owns the address region of `confirm_and_pay_checkout_screen.dart` before T-0065) |
| T-0078 | `Portal-location-insights` (Locations tab: out-of-range demand map, methods, permission denied) | bEasy-portal | T-0071 shell; T-0077; backend `/app-usage/locations` | T2 | E |
| T-0079 | `App-cancel-reschedule-tracking` (events, reason in request, remove free text from Firebase, tests; no flow change) | bEasy-app | T-0063; T-0080 agreed fields | T3 | B0 (owns `order_detail/*` cancel and reschedule code and the cancel/reschedule lines of `payment_controller.dart` before T-0066) |
| T-0080 | `Cross-cancel-reschedule-contract` (backend request §11, API spec, expectation doc, reply) | cross | T-0062 | T3 | A |
| T-0081 | `Portal-order-changes-analytics` (Orders → Changes: cancellations, reschedules, signals; order timeline; After-purchase block) | bEasy-portal | T-0080 reply (mock first); coordinate with `Portal-rds-orders` | T2 | D |
| T-0082 | `App-instant-cleaning-tracking` (booking_mode param, rooms, availability, duration source, upsell, grace errors, progress/extend views; no flow change) | bEasy-app | T-0063 | T3 | B0 (owns `custom_instant_*`, `product_option_selector`, instant parts of `category_building_screen.dart` before T-0065) |
| T-0083 | `Cross-instant-cleaning-contract` (backend request §12, API spec, reply) | cross | T-0062 | T3 | A |
| T-0084 | `Portal-instant-cleaning-insights` (Instant tab, Booking mode filter) | bEasy-portal | T-0083 reply (mock first); T-0071 shell | T2 | E |
| T-0073 | `Cross-tracking-qa-signoff` (DebugView matrix, staging batch check, docs sync, legacy retirement decision) | cross | T-0064..T-0067, T-0069, T-0070 | T2 | F |

Parallel: group A together (now incl. T-0075); **B0 (T-0074, T-0077, T-0079, T-0082) merges before B** because they touch lines in files that T-0064/T-0065/T-0067 also edit, and B rebases on them; group B four tasks together (disjoint files, see each `Touches`); C after T-0065; D can start on mock data once T-0070 agrees the contract; E after D; F last.
Overlaps found: `bEasy-portal` marketing routes are being converted by `Portal-rds-marketing` (3-in-progress) and the sidebar/routes by `Portal-marketing-foundations-ui` (2-todo); T-0071 starts only after both land or coordinates through `Tasks/_inbox/`. T-0066 no longer owns cancel and reschedule (moved to T-0079); T-0067 no longer owns auth or location (moved to T-0074 / T-0077); T-0065 skips the checkout address region owned by T-0077/T-0074. `T-0009 App-journey-event-qa` and `T-0031 Cross-meta-app-events-setup` verify existing events and may edit `analytics_helper.dart`/`facebook_helper.dart` only if a bug is proven; T-0063 and T-0069 re-read those files first. `git worktree list` shows no other bEasy worktree live.

## Registration and location addendum (2 Nov)
Findings R-1..R-8, L-1..L-7 and decisions D-6..D-8, O-7..O-9: `Registration_And_Location_Flow_Spec.md`. Headline: for a new number the app verifies the OTP **after** the sign-up form (changed 27 Oct, commit `1bfecde4`) but the portal funnel puts OTP verified before Sign-up started; every registration reports `entry=gate` because no caller sets `entry`; the address coverage check and every address action are untracked.

## Cancellation and reschedule addendum (2 Nov)
Spec: `Cancellation_And_Reschedule_Spec.md`. Headline: the app never sends the cancel reason to the backend (empty POST body) and sends the customer's free text to Firebase; both fee panels are hardcoded placeholders; there are no result, dismiss or save events; the backend stores no reason, fee, refund or reschedule record. Owner decisions (2 Nov): **no app-flow change** (O-7, O-8 answered), group `checkout_*` entries (O-9 yes). New open questions: O-10 placeholder fees (recommend: backend ships a fee preview, app swap is a separate task), O-11 optional `reasonCode`/`reasonText`/`source` on existing calls (recommend yes), O-12 count by cancel date with cohort toggle (recommend yes).

## Instant Cleaning addendum (2 Nov)
Spec: `Instant_Cleaning_Tracking_Spec.md` (findings I-1..I-9, D-10/D-11, O-13/O-14). Headline: nothing distinguishes Instant from other services, room ticks, availability/fully-booked, same-day grace errors and upsell to Deep Cleaning are untracked, and the auto-selected duration is logged as a customer choice. T-0065 leaves the Instant code to T-0082.

## Out of scope
- Implementing anything in this phase.
- Replacing the journey funnel, `event_log`, or backend-owned money events.
- Session replay tooling beyond Clarity tags; Firebase Performance (backlog); label-editing UI for the registry; TikTok SDK; landing page and mini-app tracking (separate repos, `EVENTS.md` governs landing).
- Changing existing Meta event definitions (T-0031 owns verification).
- Any change to what the app shows or does: placeholder fees stay until the backend fee API ships (separate task, O-10); registration order and ad-platform event timing stay (D-9).
