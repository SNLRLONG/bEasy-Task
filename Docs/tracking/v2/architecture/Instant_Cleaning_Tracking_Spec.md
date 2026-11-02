# Instant Cleaning — tracking audit and specification

Updated: 2026-11-02 · Status: **draft for review** (no code changed; **no app-flow change**, owner rule) · Extends [App_Tracking_Coverage_Spec.md](App_Tracking_Coverage_Spec.md) §4 D/E/F (this file wins for Instant) · Plan: [`Tasks/_plans/Cross-tracking-coverage.md`](../../../../Tasks/_plans/Cross-tracking-coverage.md)

Evidence: `bEasy-app` `main`, 2 Nov 2026. Backend statements are doc-only (`bEasy-api/` not cloned). Instant Cleaning is the service whose detail response has `bookingMode == 'INSTANT'` (`category_building_screen.dart:1210`); it reuses `CategoryBuildingScreen` and the shared checkout.

## 1. Instant flow and what is tracked

| # | Step | Code | Tracked today |
|---|---|---|---|
| 1 | Home tile (with `badge`/Beta) → optional **selection alert sheet** (once per category) → service | home grid, `custom_category_selection_alert_sheet.dart`, `docs/category-selection-alert-sheet-trigger.md` | `service_click`; alert shown/closed/CTA: **none** |
| 2 | Service detail loads; **availability check** over all dates (`_checkInstantAvailability`); if every date unavailable → **fully-booked sheet** | `category_building_screen.dart:345` | `service_viewed`, `booking_started`; fully-booked: **none** (demand lost) |
| 3 | "How long" duration cards (1h, 2h, "Suggested" badge); first option **auto-selected** | `product_option_selector.dart`, `:301` | `service_option_selected` + `duration_selected` (via `optionSelected`, `isInstant`), but auto-selection also fires them as if the customer chose |
| 4 | **Room/task ticks** (living room, kitchen…) → total minutes → band → auto-suggested hours; under-pick warning | `custom_instant_rooms_section.dart`, `_onInstantRoomToggle` `:1348` | **none** (room choice, suggested hours, warning) |
| 5 | "Need more than 2 hours?" link and "See Deep Cleaning" button → pushes Deep Cleaning | `_onExploreOtherOptionsTap` `:1012` | legacy `instant_cleaning_explore_other_options` (same event for both entry points; no `source`, no target) |
| 6 | Add-on groups, quantity | `custom_service_addon_group.dart` | `addon_*` events (service_detail_tracker) |
| 7 | Info sections: before we arrive, excludes, how it works, why choose, estimated time, task info, equipment | `custom_*_section.dart` | `task_info_viewed`, `equipment_viewed` only; the Instant sections (excludes, how it works, why choose, before we arrive) **none** |
| 8 | Time slot: `CustomInstantTimeSelector` (choose time slot → time picker), or checkout schedule sheet | `custom_instant_time_selector.dart`, `checkout_booking_schedule_sheet.dart` | `date_selected`/`time_selected`/`slot_selected` from checkout; the selector widget itself and sheet open/close/confirm: **none** |
| 9 | **Checkout grace token** for a same-day start (`requestCheckoutGrace`); errors `INSTANT_NOTICE_TOO_SHORT`, `BOOKING_SLOT_UNAVAILABLE`, `BOOKING_LEAD_TIME_REQUIRED`, `BOOKING_GRACE_INVALID` clear the slot and refresh | `booking_grace_logic.dart`, `payment_controller.dart:2055` | **none** (only generic snackbar). These are the Instant drop-off reasons |
| 10 | Checkout, promo, payment | shared | `checkout_started`, `payment_started`, etc. (no `booking_mode` to separate Instant) |
| 11 | Order detail: **instant progress card** (waiting → in progress → wrapping up → completed countdown) | `order_detail_instant_progress_card.dart` | **none** |
| 12 | **Extend by 1 hour** offer (card section currently commented out; sheet and pay path exist) | `order_detail_extend_sheet.dart` | `extend_hour_pay` fires with no payment call behind it (UI-only, `onPay` returns true) |
| 13 | Instant order cancel/reschedule | shared sheets | covered by [Cancellation_And_Reschedule_Spec.md](Cancellation_And_Reschedule_Spec.md); Instant needs `booking_mode` on those events |

### Findings
- **I-1** No event separates Instant from other services: `booking_mode` is missing on every journey, checkout and order event, so the portal cannot report Instant funnels.
- **I-2** Room ticks, suggested hours and the under-pick warning (the Instant decision logic) are invisible.
- **I-3** Auto-selected hour option is tracked as a customer choice (step 3); `selection_method` is missing.
- **I-4** Availability outcome and the fully-booked sheet are invisible, so unmet demand (the key Instant ops number) is unknown.
- **I-5** Same-day grace/notice errors are the main Instant failure and are untracked; `INSTANT_NOTICE_TOO_SHORT` and slot-unavailable need a count by hour of day.
- **I-6** "Need more than 2 hours?" and "See Deep Cleaning" share one legacy event; upsell to Deep Cleaning cannot be measured end to end (Instant → Deep Cleaning booked).
- **I-7** Instant info sections (excludes, how it works, why choose) cannot be linked to conversion.
- **I-8** Order-detail progress card and Extend are untracked; `extend_hour_pay` is a UI event that does not mean a payment (the extend API is not wired).
- **I-9** `booking_mode` is also needed on backend order records for Instant order reporting (doc-only: not in the journey APIs).

## 2. Common parameter

Add `booking_mode` (`instant` | `scheduled`) to every event that already carries `service_id`, `journey_id` or `order_id`, set by the wrapper from the service detail (cached per `service_id` in `JourneyStore` or a small map in `AnalyticsHelper`). This covers `service_viewed` through `payment_started`, add-on events, and cancel/reschedule events. `instant_hours` (selected hours) is added to `checkout_started` and `payment_started` for Instant only. No existing param changes meaning.

## 3. New events (no flow change)

| Event | Trigger | Params (beyond common) | Platforms |
|---|---|---|---|
| `category_alert_viewed` | Selection alert sheet shown | `service_id`, `alert_key`, `badge` | Firebase; backend ingest |
| `category_alert_action` | Alert closed (X) or CTA | `service_id`, `action` (`close`/`cta`) | Firebase |
| `instant_availability_checked` | After `_checkInstantAvailability` and the quiet refresh | `service_id`, `available_dates`, `total_dates`, `fully_booked` (0/1), `trigger` (`load`/`refresh`/`return`) | Firebase; backend ingest |
| `instant_fully_booked_viewed` | Fully-booked sheet shown | `service_id`, `total_dates` | Firebase; backend (demand) |
| `instant_duration_selected` | Hour card tapped **by the user** | `hours`, `selection_method` (`user`/`auto_default`/`auto_rooms`/`restored`), `was_suggested` (0/1) | Firebase; backend |
| `instant_room_toggled` | Room tick on/off | `room_key` (stable service-item id or slug, not the label), `selected` (0/1), `selected_count`, `total_minutes`, `band` (`none`/`one_hour`/`two_hours`/`bigger_job`), `suggested_hours` | Firebase; backend |
| `instant_underpick_warning_viewed` | Warning appears | `selected_hours`, `suggested_hours`, `total_minutes` | Firebase |
| `instant_upsell_clicked` | "Need more than 2 hours?" or "See Deep Cleaning" | `source` (`text_link`/`rooms_section`), `target_service_id`, `rooms_selected` | Firebase; backend |
| `instant_upsell_result` | Return from Deep Cleaning | `target_service_id`, `converted` (0/1: a Deep Cleaning `checkout_started` happened), `returned` (0/1) | Firebase |
| `element_clicked` / `overlay_viewed` | Per the foundation: info sections expand/links (`excludes`, `how_it_works`, `why_choose`, `before_we_arrive`, `estimated_time`), schedule sheet open/close/confirm, time selector tap | `screen_name=service_detail`, `element_id`, `section` | Firebase; backend |
| `instant_slot_selected` | Time chosen in selector or schedule sheet | `slot_start`, `same_day` (0/1), `lead_minutes`, `slot_source` (`selector`/`schedule_sheet`), `days_ahead` | Firebase; backend |
| `schedule_grace_result` | `requestCheckoutGrace` returns | `result` (`ok`/`error`), `error_code` (`INSTANT_NOTICE_TOO_SHORT`, `BOOKING_SLOT_UNAVAILABLE`, `BOOKING_LEAD_TIME_REQUIRED`, `BOOKING_GRACE_INVALID`, `BOOKING_POLICY_INVALID`, `INSUFFICIENT_BALANCE`, `other`), `lead_minutes`, `same_day`, `action` (`clear_refresh`/`refresh_config`/`top_up`/`message`) | Firebase; backend |
| `instant_progress_viewed` | Order detail progress card first shown per order/stage | `order_id`, `stage` (`waiting`/`in_progress`/`wrapping_up`/`completed`) | Firebase |
| `extend_started` / `extend_payment_selected` / `extend_closed` | Extend sheet (when the offer is re-enabled) | `order_id`, `stage`, `extra_hours`, `quote_amount` **only if server-provided**, `payment_method` | Firebase |

Changes to existing: `service_option_selected`/`duration_selected` gain `selection_method` (auto-selection no longer looks like a choice; keep firing so counts stay continuous, flagged); legacy `instant_cleaning_explore_other_options` keeps firing one release with a new `source`; `extend_hour_pay` is renamed in meaning: documented as UI-only, no revenue (the backend payment record is the truth). Meta: `ViewContent`/`InitiateCheckout`/`Purchase` already fire for Instant; add `content_category=instant` where the SDK supports it. No Meta/AppsFlyer event for rooms or availability.

Privacy/limits: `room_key` is a catalogue id, never free text; no address; ≤12 params per event.

## 4. Backend

(Added to `pending-request-backend.md` §12.)
1. Accept `booking_mode`, `instant_hours`, `selection_method` and the §3 events; store `booking_mode` on `journey` and on `order` (orders: `INSTANT` vs scheduled, derived from the service/booking mode the app already sends) so Instant is a filter on every funnel, After-purchase and order report.
2. Tables (spec 3NF, additions to coverage spec §6.2): `instant_availability_check (event_id, occurred_at, customer_id NULL, app_instance_id, service_id, available_dates, total_dates, fully_booked, trigger)`, `instant_room_selection (journey_id, room_key, selected, total_minutes, band, suggested_hours, occurred_at)` (last state per journey kept as read model), `schedule_grace_error (event_id, occurred_at, service_id, booking_mode, error_code, lead_minutes, same_day, hour_of_day, app_version, platform)`. Server-side, also store every rejected grace/schedule request with the code (the authoritative source; the app event is the signal).
3. Endpoints (filters as Journey plus `booking_mode`): `GET /instant/overview` (views → availability → duration → rooms → slot → checkout → paid, fully-booked rate, same-day share, avg lead, upsell to Deep Cleaning rate); `GET /instant/rooms` (room popularity, band distribution, suggested vs chosen hours, under-pick warnings, conversion by band); `GET /instant/availability` (fully-booked by day and hour, unmet demand count, views while fully booked); `GET /instant/schedule-errors` (by `error_code`, hour of day, lead bucket); `GET /instant/upsell` (clicks, Deep Cleaning conversions); `GET /instant/orders/progress` (orders by stage right now, completed on time, extended). Existing `GET /funnels/booking` accepts `booking_mode`.
4. Confirm the booking-mode value set (`INSTANT` and others), the error-code list, and whether the extend payment API/record is planned (until then extend revenue is zero and not reported).

## 5. Portal

New tab **Instant Cleaning** under Marketing → App Usage (and a `Booking mode` filter chip on Customer Journey, Orders and After-purchase):

| View | Content | Decision |
|---|---|---|
| Funnel | service viewed → availability ok → duration → rooms ticked → slot → checkout → paid; drop reasons from schedule errors | Where Instant loses people |
| Unmet demand | fully-booked views by day/hour, views while no slot, trend | Ops: add partners/slots, open hours |
| Rooms | popularity bars, band distribution, suggested vs chosen hours, under-pick warning rate, conversion by band | Pricing/duration design; are estimates right (placeholder minutes in the app) |
| Schedule errors | by `error_code`, by hour of day, lead bucket, same-day share | Is the minimum notice too strict |
| Upsell | "Need more than 2 hours?" clicks → Deep Cleaning conversions | Product |
| Live orders | stage counts (waiting/in progress/wrapping up/completed), extends | Ops monitoring |

`null` is "—"; signals vs records labelled as in the cancel spec.

## 6. Decisions and open questions
- D-10 Instant = `bookingMode == 'INSTANT'`; `booking_mode` is a common param, not a separate event family.
- D-11 No flow change: auto-selection, restore after Deep Cleaning and warnings behave as today; they are only labelled.
- **O-13** Are `instantRoomPlaceholderMinutes` (25/30/15) the real estimates? *Recommend: no, ops confirms; analytics stores `total_minutes` as computed so the portal flags when the API supplies `durationMinutes`.* 
- **O-14** Extend offer is commented out and `onPay` is UI-only: keep tracking limited to views until the backend extend API exists? *Recommend yes.*
