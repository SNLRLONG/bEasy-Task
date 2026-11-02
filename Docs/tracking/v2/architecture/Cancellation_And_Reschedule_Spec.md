# Cancellation and reschedule — tracking audit, backend contract, portal decisions

Updated: 2026-11-02 · Status: **draft for review** (no code changed) · Extends [App_Tracking_Coverage_Spec.md](App_Tracking_Coverage_Spec.md) §3.4/§4 F (this file wins for cancel and reschedule) · Plan: [`Tasks/_plans/Cross-tracking-coverage.md`](../../../../Tasks/_plans/Cross-tracking-coverage.md)

**Constraint (owner, 2 Nov):** no change to how the app behaves. This spec only changes what is *collected*, what backend *stores*, and what the portal *shows*. Where the app shows something wrong (placeholder fees), that is recorded as a finding and a separate decision (O-10), not folded into the tracking work.

Evidence: `bEasy-app` and `bEasy-portal` at `main`, 2 Nov 2026. Backend statements are *doc-only* (`bEasy-api/` is not cloned): `backend-response/Journey_V2_Portal_Review.md` (14 Oct) and `api/Journey_Backend_API_Spec.md`.

---

## 1. Current state

### 1.1 Cancellation (customer, app)

| Step | Code | Today |
|---|---|---|
| Open cancel | `order_detail_bottom_actions` → `_openCancelSheet` (`order_detail_screen_v2.dart:215`) | **No event** |
| Reason step: 5 reasons (`plan_changed`, `booked_by_mistake`, `want_reschedule`, `no_longer_needed`, `other` + free text) | `order_detail_cancel_sheet.dart:99-110` | No event on selection |
| "I want to reschedule instead" | same, `onWantReschedule` | Opens reschedule; `reschedule_started` fires with no hint that it came from cancel |
| Confirm step shows the fee: $3.30, on hold $36.30, "24-2 hours" | `CancelFeePreview.placeholder` | **Hardcoded**; the real fee is never read |
| Confirm tap | `order_detail_screen_v2.dart:243` | Firebase `cancel_booking_confirm` `{reason, other}` **before** the API call (intent, not result). `other` = the customer's free text (up to 100 chars) goes to Firebase: personal-text risk |
| API | `POST customer/order/v2/{id}/cancel` with body `{}` (`payment_repository.dart:83`) | **The reason is not sent to the backend.** Backend cannot store it |
| Result | `paymentCancelOrder` | No event for success or failure. The sheet's success step is untracked |
| Dismiss | swipe/back | No event (cannot measure "saved" cancellations) |
| Cancel status checks | `cancel/status` (v1 and v2) | untracked |
| Checkout "cancellation and reschedule policy" sheet | `checkout_cancellation_policy_sheet.dart`, `confirm_and_pay…:2151` `read_full_policy` | untracked |

Admin cancel (portal, `routes/order/order.tsx`, `use-cancel-order-mutation.ts`) already sends `reason`, `refundAmount`, `expectedRevision`, `requestId`; for ABA mini-app orders it calls the mini-app API. Backend does not currently expose cancel reason or fee to journey APIs.

### 1.2 Reschedule (customer, app)

| Step | Code | Today |
|---|---|---|
| Open | `_openRescheduleSheet` | Firebase `reschedule_button_click` (legacy) + `reschedule_started` `{order_id, current_slot, reschedule_source}` |
| Pick a new slot, validate with a grace token `POST customer/order/bulk/reschedule-grace` | `onRequestGrace` | `reschedule_slot_selected` `{old_slot,new_slot}`; the grace result (slot unavailable, error) is **not** tracked |
| Fee | `_isFreeWindow => true` (always free), fee `$2.20` is a placeholder; paid path "not wired" | No fee data anywhere |
| Confirm | `reschedule_button_confirm` (before the API) | legacy; no result |
| API | `POST customer/order/bulk/edit-schedule` (+ grace token) | `reschedule_completed` `{old_slot,new_slot,source}` and legacy `order_rescheduled`; `reschedule_failed` `{failure_reason = backend code}` |
| Rules in the client | Free window "24 h before start" lives in the app (`isInsideFreeRescheduleWindow`, currently disabled); `canReschedule` comes from backend | Policy exists in two places |
| Dismiss | back/swipe | no event |

Order detail **v1** (`order_detail_screen.dart`) duplicates `reschedule_button_*`; reachability is checked in T-0066.

### 1.3 What backend and portal have

Backend order records produce `order_cancelled` (counts) but **fee kept, refund amount and reason are not stored**; `rescheduled` is **not tracked at all** (`orders: null`, `unavailableReason: not_tracked`); client `reschedule_*`/`order_rescheduled` are skipped or pending the whitelist (pending request 3.3, 3.4). The portal After-purchase block renders `cancelled` (orders only) and `rescheduled` as "Not tracked" and never as 0.

### 1.4 Findings

| # | Finding | Impact |
|---|---|---|
| CR-1 | Cancel reason never reaches the backend (empty POST body) | No cancellation-reason report is possible, in any tool |
| CR-2 | `cancel_booking_confirm` is sent before the API and carries free text | Counts attempts as cancellations; sends customer text to Firebase |
| CR-3 | No cancel result, no cancel-sheet open/dismiss events | Cannot compute save rate or failure rate |
| CR-4 | Fee panels are hardcoded placeholders in both sheets | The amounts the customer sees are not facts; analytics must **not** treat them as fees |
| CR-5 | No link from cancel to reschedule (`want_reschedule` reason, redirect) | Cannot show reschedule-instead saves |
| CR-6 | Grace-token (slot validation) outcome and reschedule dismissals untracked | Cannot see slots people want but cannot get |
| CR-7 | `order_id` is the `bulkOrderId` in app events; backend after-purchase keys are order ids — join key must be agreed | Client funnel cannot be reconciled with order records |
| CR-8 | Fee window and policy logic exist both in the client and (doc-only) in the backend | Risk of two truths; portal must display backend's rule |
| CR-9 | No `cancelled_by`, channel (app / portal / mini-app / system) on records | Admin cancels are mixed with customer cancels |
| CR-10 | Reschedule count per order, reschedule-then-cancel, partner-assigned state at cancel are unknown | The most useful ops signals are missing |

---

## 2. App data collection (no behaviour change)

Rules: call `AnalyticsHelper.track`; `order_id` = `bulkOrderId` string; pass `service_id` as a plain parameter (not the `serviceId:` argument, which would attach a booking `journey_id`); times are ISO-8601 with `+07:00`; **no placeholder amounts and no free text** are ever sent to Firebase.

Derived values the app can compute from `bookingDetail` without a new call: `hours_to_start` (start − now, rounded to 0.5 h, negative if started), `order_status`, `order_value` + `currency`, `service_id`, `partner_assigned` (0/1) if the detail has partners, `reschedule_count` if present.

### 2.1 Cancellation events

| Event | Trigger | Params (beyond common) |
|---|---|---|
| `cancellation_started` | Cancel sheet appears (`overlay_viewed` also fires from the foundation) | `order_id`, `entry` (`order_detail`/`reschedule_sheet`), `hours_to_start`, `order_status`, `value`, `currency`, `service_id`, `partner_assigned` |
| `cancellation_reason_selected` | A reason row is tapped (last one wins; only emit once per change) | `order_id`, `reason_code` |
| `cancel_to_reschedule_clicked` | "I want to reschedule instead" | `order_id`, `reason_code=want_reschedule` |
| `cancellation_policy_viewed` | "View details" / "Read full policy" | `order_id` (or none from checkout), `source_screen` |
| `cancellation_fee_shown` | Only once the fee comes from the server (O-10) | `order_id`, `fee_amount`, `fee_source=server`, `fee_rule_code` — **not fired while the placeholder is used** |
| `cancellation_submitted` | Confirm tapped, before the call (replaces the intent role of `cancel_booking_confirm`) | `order_id`, `reason_code`, `has_other_text` (0/1), `hours_to_start` |
| `cancellation_result` | The cancel call returns | `order_id`, `result` (`success`/`failed`), `error_code`, `http_status`, `duration_ms` |
| `cancellation_abandoned` | Sheet dismissed without a successful cancel | `order_id`, `step` (`reason`/`confirm`), `reason_code` (if chosen) |

`reason_code` enum (stable, English; labels in the portal): `plan_changed`, `booked_by_mistake`, `want_reschedule`, `no_longer_needed`, `other`. The backend adds `admin_*` and `system_*` values for non-customer cancels (section 3.2).

Legacy `cancel_booking_confirm`: keep the event name for one release (backend does not use it), **remove the `other` free-text parameter**, keep `reason`.

### 2.2 Reschedule events

| Event | Trigger | Params |
|---|---|---|
| `reschedule_started` *(exists)* | Sheet opens | add `entry` (`order_detail`/`cancel_sheet`), `hours_to_start`, `reschedule_count`, `service_id`, `value`, `currency` |
| `reschedule_slot_selected` *(exists)* | Slot tapped | keep `old_slot`, `new_slot`; add `shift_hours` (new − old, signed) |
| `reschedule_grace_requested` | Grace-token call returns | `order_id`, `result` (`ok`/`slot_unavailable`/`error`), `error_code`, `new_slot` |
| `reschedule_fee_shown` | Only when the fee is server-provided | `order_id`, `fee_amount`, `fee_source=server`, `is_free` — not fired today (always free in the UI) |
| `reschedule_submitted` | Confirm tapped (replaces the intent role of `reschedule_button_confirm`) | `order_id`, `old_slot`, `new_slot`, `payment_method` (only when paid) |
| `reschedule_completed` *(exists)* | Call succeeds | keep; add `is_free` (null/omit until the server says), `fee_amount` (omit until server says), `reschedule_count` |
| `reschedule_failed` *(exists)* | Call fails | keep `failure_reason` (backend `code`); add `http_status` |
| `reschedule_abandoned` | Sheet dismissed without success | `order_id`, `step` (`pick`/`confirm`/`choose_pay`/`payment_failed`) |

The legacy `order_rescheduled` client event stays (backend skips it); the **backend** `order_rescheduled` record is the source of truth (section 3).

### 2.3 Platform assignment

| Event | Firebase | Backend ingest | Meta / AppsFlyer |
|---|---|---|---|
| all `cancellation_*`, `cancel_to_reschedule_clicked`, `reschedule_*` above | yes | yes — the funnel events (whitelist), see 3.4 | none (service-quality signals, not ad events) |
| Clarity | tag `screen_name`, custom event `cancellation_result` / `reschedule_failed` for replay lookup | – | – |

### 2.4 Tests (in the app task)
Event builder tests; a test that no cancel/reschedule event contains a key outside the allow-list or a string longer than 100 chars; a regression test that the order-detail flow behaviour is unchanged (existing screen tests, no new navigation).

---

## 3. Backend contract

All items go to `pending-request-backend.md` §11. Backend owns facts; the app owns signals about the screens.

### 3.1 What the app will send (request body and headers)

`POST customer/order/v2/{id}/cancel` today `{}`. Request that it **accepts, optionally and backward-compatibly**:
```json
{ "reasonCode": "plan_changed", "reasonText": "free text, max 200, may be null", "source": "app" }
```
Old app versions send `{}`; backend stores `reasonCode = null` (shown as "Not recorded", never as another reason). Same for `POST customer/order/bulk/edit-schedule`: accept optional `{ "source": "app", "entry": "order_detail|cancel_sheet" }`. Existing headers (`X-App-Instance-Id`, device, version) are used for `app_instance_id`, `platform`, `app_version`.

(Sending the new optional fields is a data-collection change in the request, not a flow change. It is the only way the reason can ever reach the record. It needs product sign-off only because it adds a field to an existing call: O-11.)

### 3.2 Tables

```text
order_cancellation (
  cancellation_id PK, order_id FK, bulk_order_id, customer_id,
  cancelled_at timestamptz, cancelled_by enum(customer|admin|system|partner),
  channel enum(app|portal|mini_app|api), admin_user_id NULL,
  reason_code enum(plan_changed|booked_by_mistake|want_reschedule|no_longer_needed|other
                  |admin_customer_request|admin_no_partner|admin_duplicate|admin_other
                  |system_payment_timeout|system_unfulfilled) NULL,
  reason_text NULL,                          -- stored, never exported to ad platforms
  order_status_before, payment_status_before, partner_assigned bool,
  slot_start timestamptz, lead_minutes int,  -- slot_start - cancelled_at
  order_total numeric(10,2), currency,
  fee_rule_code NULL, fee_amount numeric(10,2) NULL, fee_waived bool NULL,
  fee_waived_by NULL, fee_waived_reason NULL,
  refund_amount numeric(10,2) NULL, refund_destination enum(original_payment|wallet|none) NULL,
  refund_status enum(none|pending|done|failed) NULL, refund_at NULL,
  reschedule_count_before int, app_instance_id NULL, app_version NULL, platform NULL,
  service_id, zone_id NULL, request_id UNIQUE)         -- idempotency

order_reschedule (
  reschedule_id PK, order_id FK, bulk_order_id, customer_id,
  requested_at, status enum(completed|rejected), failure_code NULL,
  initiated_by enum(customer|admin|system|partner), channel enum(app|portal|mini_app|api),
  old_slot_start, new_slot_start, shift_minutes int,
  lead_minutes_before_old_slot int, reschedule_no int,          -- 1st, 2nd, ... for this order
  is_free bool, fee_rule_code NULL, fee_amount numeric(10,2) NULL,
  fee_payment_status enum(none|pending|paid|failed) NULL, fee_payment_method NULL,
  partner_changed bool NULL, grace_token_id NULL,
  app_instance_id NULL, app_version NULL, platform NULL, service_id, zone_id NULL, request_id UNIQUE)

cancellation_fee_rule (rule_code PK, applies_to enum(cancel|reschedule), service_id NULL,
  min_hours_before, max_hours_before NULL, fee_type enum(fixed|percent|none), fee_value,
  vat_included bool, valid_from, valid_to NULL)
ref_cancellation_reason (reason_code PK, label_en, label_km, group enum(customer|admin|system), is_active)
```
`order_reschedule` records rejected attempts too (`status=rejected`, `failure_code`), so "slot unavailable" demand is measurable. `lead_minutes`, `reschedule_no`, `shift_minutes` are stored, not recomputed, so history survives rule changes. Money is USD with two decimals, VAT included (state the flag). Time is stored UTC and returned ISO-8601 `+07:00`.

### 3.3 Event log (backend-owned, architecture §11.1)
Write `order_cancelled` (add `reason_code`, `cancelled_by`, `fee_amount`, `fee_waived`, `refund_amount`, `lead_minutes`), `order_rescheduled` (add `old_slot`, `new_slot`, `fee_amount`, `is_free`, `reschedule_no`, `initiated_by`), `order_refunded` (`amount`, `destination`, `status`) into `event_log` so the Customer page timeline shows them. Admin and system actions are included (`by: backend|team`).

### 3.4 Ingest of client funnel events
Whitelist (currently pending 3.4): `cancellation_started`, `cancellation_reason_selected`, `cancel_to_reschedule_clicked`, `cancellation_policy_viewed`, `cancellation_submitted`, `cancellation_result`, `cancellation_abandoned`, `reschedule_started`, `reschedule_slot_selected`, `reschedule_grace_requested`, `reschedule_submitted`, `reschedule_completed`, `reschedule_failed`, `reschedule_abandoned`. Store in `app_interaction_event` (Coverage spec §6.2) with `order_id` as a column. They never create or change a cancellation or reschedule record.

Reconciliation job: daily count of client `cancellation_result=success` vs `order_cancellation` rows by `order_id`; expose the mismatch on `GET /meta/data-freshness`.

### 3.5 Rules the portal depends on
1. **Null means unknown.** Fee, refund, reason and lead are `null` when not stored; the API never returns 0 for unknown.
2. **One record per action**, keyed by `request_id`; retries do not duplicate.
3. **Counting:** `byCancelDate` (default: records with `cancelled_at` in range) and `cohort` (orders placed in range that were later cancelled); every response says which.
4. **Rate denominators** are explicit `{unit: "orders", count}` like the existing After purchase block.
5. Backend remains the source of the fee and policy; the app must never be the source of a fee for reporting.
6. Exclude test orders and staff phones, as for other tracking.

### 3.6 Endpoints for the portal

Filters on all: `period|from|to`, `service_id`, `zone_id`, `platform`, `app_version`, `source`/`campaign_id`, `customer_type`, `cancelled_by`, `reason_code`, `lead_bucket` (`gt24h`, `24h_2h`, `lt2h`, `after_start`), `partner_assigned`, `channel`, `mode` (`byCancelDate`/`cohort`).

| Method | Path | Returns |
|---|---|---|
| GET | `/after-purchase/cancellations/summary` | `{count, rate{orders,denominator}, feeKept, feeWaived, refundTotal, refundPending, avgLeadHours, byReason[], byLeadBucket[], byCancelledBy[], byChannel[], byService[], byZone[], partnerAssignedShare, rescheduledBeforeCancel, trend[]}` |
| GET | `/after-purchase/cancellations` | paged rows: order, customer (masked), service, zone, cancelled_at, by, reason, lead, fee, waived, refund, refund status; export |
| GET | `/after-purchase/cancel-funnel` | client signal funnel: started → reason selected → submitted → success, plus `savedByReschedule`, `abandoned` (by step), `failed` (by `error_code`) |
| GET | `/after-purchase/reschedules/summary` | `{count, rate, free, paid, feeTotal, avgShiftHours, avgLeadHours, byReschedule_no[], rejected{byFailureCode[]}, rescheduledThenCancelled, byService[], byZone[], byInitiatedBy[], trend[]}` |
| GET | `/after-purchase/reschedules` | paged rows (order, old/new slot, shift, lead, no., free/paid, fee, status, failure) |
| GET | `/orders/{bulkOrderId}/changes` | order timeline: original slot, every reschedule (incl. rejected), cancellation, fee, refund, with actor and channel |
| GET | `/funnels/acquisition` | existing `afterPurchase[]` rows filled: `rescheduled` `{orders, amount, detail{done, upcoming, paidFee}}`, `cancelled` `{orders, amount, detail{withFee, free}}`, with `unavailableReason` only when truly unknown |
| GET | `/meta/cancellation-reasons`, `/meta/fee-rules` | reason labels (EN/KM) and the active fee rules to render policy next to the numbers |

Alerts/people lists (for decisions): `GET /customers?filter=reschedules>=3`, `cancellations>=2` (people-list filters, masked), reusing the existing people-list contract.

---

## 4. Portal: what is shown and which decision it supports

New page **Orders → Changes** (cancellations and reschedules) plus upgrades to existing blocks. RDS v2 components. Tabs: **Cancellations**, **Reschedules**, **Signals** (app funnel).

| View | Metrics and charts | Decision it supports |
|---|---|---|
| Cancellations KPIs | cancelled orders, cancellation rate, fee kept $, fee waived $, refunded $, refund pending, avg lead | Is the fee policy working; cash exposure |
| Reasons | bar by `reason_code`, trend, compare by service/zone/platform | Product and ops: if `booked_by_mistake` is high fix the booking UX; if `no_longer_needed` clusters in a zone check supply |
| Lead-time distribution | histogram by `lead_bucket`, split by fee charged/waived | Is the 24-2 h window set right; are late cancels costly |
| Who and where | `cancelled_by` and `channel`, partner assigned vs not, by service and zone | Admin cancels vs customer cancels; partner-side failure |
| Cancel funnel (signals) | started → reason → submitted → success, abandoned by step, `savedByReschedule`, failures by `error_code` | Save rate; is the cancel flow failing technically |
| Reschedules KPIs | count, rate, free vs paid, fee $, avg shift, avg lead, `reschedule_no` distribution, rescheduled-then-cancelled | Is the free window abused; does rescheduling prevent cancellation |
| Rejected attempts | by `failure_code` and by requested slot | Demand for slots we cannot give: schedule capacity |
| Lists | filterable tables with Export, "Send to CS inbox" for customers with ≥3 reschedules or ≥2 cancellations, open an order | CS follow-up |
| Order and Customer pages | `GET /orders/{id}/changes` timeline in the order detail and Customer Orders tab | Support answers in one place |
| Journey After-purchase block | `cancelled` and `rescheduled` rows filled; "Not tracked" disappears when data arrives | Existing acquisition view |
| Data quality tile | client-signal vs record reconciliation, share of cancels with a recorded reason | Know when to trust the numbers |

Display rules: `null` prints "—" or "Not recorded"; every number says whether it is a **record** (backend) or a **signal** (app); "byCancelDate/cohort" toggle; Khmer and English labels from `ref_cancellation_reason`.

---

## 5. Open questions (recommendation first)

- **O-10** The fee panels in both sheets are placeholders ($3.30, $2.20) and the reschedule UI is always free. Is this intentional until the backend fee API ships? *Recommend:* yes for now; backend ships `GET customer/order/v2/{id}/cancel/preview` and a reschedule fee quote, the app swaps the placeholder in a separate task (it changes what the customer sees, so it is not part of this tracking work). Until then no fee event fires.
- **O-11** Accept adding optional `reasonCode`/`reasonText`/`source` to the existing cancel and edit-schedule calls? *Recommend yes;* without it the reason can never be stored. Old app versions keep working (field absent).
- **O-12** Cancel-by-date as the default counting mode, cohort as toggle? *Recommend yes.*
