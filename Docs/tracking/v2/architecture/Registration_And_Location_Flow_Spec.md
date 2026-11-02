# Registration flow, registration entry and location flow — audit and specification

Updated: 2026-11-02 · Status: **draft for review** (no code changed) · Extends [App_Tracking_Coverage_Spec.md](App_Tracking_Coverage_Spec.md) · Plan: [`Tasks/_plans/Cross-tracking-coverage.md`](../../../../Tasks/_plans/Cross-tracking-coverage.md)

Evidence: `bEasy-app` `main`, portal `main`, 2 Nov 2026. Backend claims are *doc-only* (the API repo is not cloned).

---

## Part A. Registration flow: portal does not match the app

### A.1 What the app does today (new phone number)

The order changed on **27 Oct 2026** (commit `1bfecde4`, "OTP first, signup form after"). The docs and the portal were written on 7–8 Oct for the earlier order.

| # | App step | Code | Event(s) fired | Server fact |
|---|---|---|---|---|
| 1 | Phone entry (`NewSignINScreen`) | `new_signin_screen.dart:94` | `registration_started` (`entry`), legacy `sign_in_screen` | – |
| 2 | Tap continue → `checkPhoneNumber` says number is new | `auth_controller.dart:checkAccountUser` | – | `OTP_SENT` (backend, from send-OTP) |
| 3 | OTP screen opens, user types the code | `new_verification_code_screen.dart:93,362` | `otp_viewed`. **No `otp_submitted`**: the screen hands the code to the signup flow and returns before the tracking line (`openSignupAfterCode` branch) | Code is **held in the app, not verified** |
| 4 | Sign-up form (`NewSignUPScreen`) | `new_signup_screen.dart:53` | `signup_viewed`, legacy `sign_up_screen` | – |
| 5 | Tap "create account" | `new_signup_screen.dart:_onCreateAccount` | legacy `create_account_button` | Account created (pending) |
| 6 | App logs in with the held code | `auth_controller.dart:690-720` `loginWithOTP` | success → **`otp_verified`**, `registration_completed` (`auth_controller.dart:898-902`); also at step 5: Firebase `sign_up` and Meta `CompleteRegistration` | OTP verified, account active |
| 6b | Held code rejected (wrong or expired — the user may have spent minutes on the form) | same | none for the rejection; app sends a **second OTP** and opens the OTP screen again: `otp_viewed` #2, `otp_submitted`, `otp_failed`/`otp_verified` | second `OTP_SENT` |
| 7 | Customer-source screen, then home | `loginWithOTP` end | – | – |

Existing phone number: phone entry → send OTP → OTP screen → `otp_submitted` → `otp_verified` (login; no `registration_completed`).

Telegram login has no OTP: account create completes the registration (`registration_completed` method `telegram`).

The older `signin_screen` / `signup_screen` / `verification_code_screen` are only reachable from `WelcomeScreen`, which no code constructs, so the chain is dead and is not instrumented. Because `NewSignINScreen` is never opened with its default `showGuestMode: true`, there is also **no live onboarding entry**: every live caller passes `showGuestMode: false`, so `registration_started.entry` is always `gate` today.

### A.2 What the portal and docs say

Stages in `bEasy-portal/app/lib/journey/stages.ts:REGISTRATION_STAGES` and UI flow v2.1 §3.2: **Registration page viewed → OTP requested → OTP verified → Sign-up started → Registration completed**. Backend mapping (`Journey_Backend_API_Spec.md:400`): `registration_viewed` = phone entered, `signup_started` = app reported the sign-up form. UC-07 in the Flow Breakdown is titled "Registration: OTP then sign-up".

### A.3 Mismatches

| # | Mismatch | Effect on the portal |
|---|---|---|
| R-1 | **`otp_verified` fires after `signup_viewed`**, not before. The portal puts "OTP verified" third and "Sign-up started" fourth | Stage 3 count is lower than stage 4 in real data, or the backend infers it; either way the funnel shows impossible "negative" drops or hides the form step. The insight text "verified the code but never started sign-up" is false: that person never verified at all |
| R-2 | The code is typed **before** it is verified, so "wrong code" and "expired" are discovered only after the form. `otp_failed` therefore cannot happen at the OTP screen for new numbers | "Why OTP fails" (wrong code, blocked) undercounts for new numbers; "expired or left" is inflated |
| R-3 | No event for "code typed" (step 3). `otp_submitted` is skipped in the handoff | Cannot tell "left at the OTP screen" from "typed code, left at the form" |
| R-4 | No event for "create account tapped" or its result (step 5); only a legacy click | Cannot split form abandonment from submit failure |
| R-5 | `registration_started` fires on the same screen for **login and register**; the app cannot say which until the phone check | Funnel top is inflated by existing customers; backend currently excludes logins by identity (doc-only) |
| R-6 | Firebase `sign_up` and Meta `CompleteRegistration` fire at **account creation (step 5)**, before the OTP is verified; `registration_completed` fires at step 6 | Meta registrations include people who never verified; differs from the portal's Registration completed. **Not changed** (D-9); documented for marketing so Meta and portal counts are not compared directly. The portal uses `registration_completed` |
| R-7 | `signup_viewed` fires every time the form opens, including after a failed attempt; two `otp_viewed` per flow in 6b | Event counts exceed people; backend must count distinct `reg_flow_id` per stage (it should already) |
| R-8 | `entry` is always `gate` in practice (all live callers pass `showGuestMode: false`; none passes the optional `entry` argument) | The three real sources the business asks about (Home, Checkout, Select address) are not distinguishable. See Part B |

### A.4 Target flow and stage definitions

Decision (owner, 2 Nov: *the app flow does not change; only data collection must be correct*): **the portal follows the app.** The funnel must describe what ships.

New-number funnel, five stages (same count, two swap places):

| # | Stage code (unchanged names) | Label EN | Definition | Source |
|---|---|---|---|---|
| 1 | `registration_viewed` | Registration page viewed | `registration_started` | App |
| 2 | `otp_requested` | OTP requested | send-OTP call for a **number with no account** | Backend `OTP_SENT` |
| 3 | `signup_started` | Sign-up form opened | `signup_viewed` (a code was typed on the OTP screen) | App |
| 4 | `otp_verified` | OTP verified | code accepted after "create account" (`otp_verified`) | App + backend |
| 5 | `registration_completed` | Registration completed | `registration_completed` | App + backend |

Stage 4 sits after the form because verification happens at submit. Reading it: drop 2→3 is OTP friction (SMS not delivered, user gave up at the code); drop 3→4 is the **form** (abandoned, or code wrong/expired at submit); drop 4→5 should be ~0 and is a data-quality tile.

Because the code is typed before the form, add one **diagnostic** (not a stage): `otp_code_entered`.

Existing-number logins stay out of this funnel (unchanged backend rule). For visibility, `otp_viewed` and `otp_submitted` gain `flow_type`.

### A.5 Events: changes (app)

| Event | Change | Params | Trigger |
|---|---|---|---|
| `registration_started` | extend | `entry` (taxonomy in Part B), `entry_detail` (optional), `source_screen` | Phone entry opens. Keep `reg_flow_id` creation here |
| `otp_viewed` | extend | `flow_type` (`register`/`login`), `otp_mode` (`deferred` = code first, verified after the form; `immediate`), `attempt_round` (1, 2 for the second-chance OTP) | OTP screen opens |
| `otp_code_entered` | **new**, diagnostic | `flow_type`, `otp_mode` | Last digit typed on the OTP screen when verification is deferred |
| `otp_submitted` | keep, fix | `attempt_no`, `otp_mode` | A real verification request is made (immediate mode, or the held code sent at account creation). Fire for the deferred path too, so attempt counts are honest |
| `otp_failed` | keep, extend | `error_code` (`wrong_code`, `blocked`, `expired`, `exception`), `otp_mode` | Verification rejected. Also fires for a held code rejected at submit (today silent) |
| `signup_viewed` | keep | `reg_flow_id`; add `attempt_round` | Form opens |
| `signup_submitted` | **new** (replaces legacy `create_account_button`) | `result` (`success`/`error`/`invalid`), `error_code`, `has_email` (0/1), `has_referral` (0/1) | "Create account" tapped and the create call returns. Never names, phone or email values |
| `otp_verified` | keep | `otp_mode` | As today |
| `registration_completed` | keep, extend | `method`, `entry` (copied from the attempt, so lists need no join) | After identity bound |
| Firebase `sign_up`, Meta `CompleteRegistration` | **unchanged** (owner decision 2 Nov: no behaviour change). Add a new Firebase event `account_created` at the same moment (`method`, `entry`) so the portal can tell *account created* from *verified* | `method`, `entry` | See D-9 |

`entry` also becomes a **user property** `registration_entry` set once at completion (cohort analysis for orders by entry).

### A.6 Backend (request §10)

- Registration stage order as in A.4. Count each stage as distinct `reg_flow_id` (or device while a guest); allow repeated `otp_viewed`/`signup_viewed` per flow.
- Store `entry`, `entry_detail`, `source_screen`, `flow_type`, `otp_mode` on a per-attempt row; expose them in the funnel and people endpoints:

```text
registration_attempt (
  reg_flow_id PK uuid, app_instance_id, customer_id NULL,
  entry, entry_detail NULL, source_screen NULL,
  started_at, otp_requested_at NULL, code_entered_at NULL, signup_viewed_at NULL,
  signup_submitted_at NULL, otp_verified_at NULL, completed_at NULL,
  otp_mode, otp_rounds int, last_error_code NULL, method NULL, campaign_code NULL)
```
- `GET /funnels/registration`: new query param `entry` (filter) and `group_by=entry`; response `entryBreakdown[]` `{entry, started, completed, conversion}`. `GET /funnels/registration/stages/{stage}/people` rows gain `entry`. `GET /customers/{id}/activity` registration group gains `entry`.
- Order-by-entry link: `GET /attribution/registration-entries`: entry → registered → first purchase within 7 / 30 days.
- Confirm that stage 4 `otp_verified` is read from the event, not inferred from `registration_completed`.

### A.7 Portal changes

| Area | Change |
|---|---|
| `lib/journey/stages.ts` `REGISTRATION_STAGES`, `types/journey.ts` `RegistrationStage` | Reorder: `registration_viewed, otp_requested, signup_started, otp_verified, registration_completed` |
| `locales/journey/{en,km}/funnel.json` `insightReason` | `signup_started`: "opened the sign-up form but never finished creating the account"; `otp_verified`: "verified the code but never completed registration"; adjust the labels in `shared.json`, `customer.json` ("Shorten the sign-up form" tip moves to stage 3) |
| `components/journey/funnel/{registration-funnel-tab,stage-icon}.tsx` | Icon and order; new **Registration entry** card under the funnel: bars per entry with conversion; clicking a bar filters the funnel and opens the people list for that entry |
| `components/journey/list/*` | New column and filter chip **Entry** (EN/KM labels from the taxonomy) |
| `components/journey/customer/{registration-summary-card,journey-tab,event-log-group-card}.tsx`, `lib/journey/customer.ts` (event order list lines ~95-120) | Show entry, `otp_mode`, rounds, code-entered row; keep event order by timestamp, not by a fixed list |
| "Why OTP fails" card | Add reason **Wrong or expired code at submit** (from `otp_failed` with `otp_mode=deferred`); relabel "Code expired or left" |
| Filters | `entry` in the shared journey URL state |

---

## Part B. Where did the registration come from

### B.1 Every route into sign-in today

All live callers open `NewSignINScreen(showGuestMode: false)` and none sets `entry`, so every registration reports `gate`.

| # | Source | Code | Customer sees | Proposed `entry` / `entry_detail` |
|---|---|---|---|---|
| 1 | Onboarding welcome | `welcome_screen` → legacy `SignINScreen`; `WelcomeScreen` is never constructed | Not reachable today | `onboarding` (kept in the set for a future onboarding; no work now) |
| 2 | **Home** "Get started / register" card for guests | `home_screen.dart:931` | Home | `home` / `get_started_card` |
| 3 | Bottom tab for guests (Bookings, Chat, Profile) | `app_screen.dart:418` | Tab bar | `tab_gate` / `bookings` · `chat` · `profile` |
| 4 | **Checkout: select / add address** | `confirm_and_pay_checkout_screen.dart:165-176` `_onAddressActionTap` → `_runGuestSignIn` | Address row on checkout | `checkout_address` |
| 5 | **Checkout: choose date** | same file `:1227` → `_runGuestSignIn` | Schedule row | `checkout_schedule` |
| 6 | Checkout: sign-up prompt | same file `:1596` (`sign_up`) | Summary banner | `checkout_summary` |
| 7 | Banner detail register button | `banner_detail_screen.dart:918` | Promo banner | `banner` / banner id |
| 8 | Push tap by a guest | `firebase_messaging_service.dart:136` | Push | `push` / push `type` |
| 9 | Deep link `REGISTER-SCREEN` | `helper.dart:321` | Link / campaign | `deeplink` |
| 10 | "Register now" dialog | `custom_show_dialog.dart:124` | Dialog (callers currently commented out) | `dialog` |

The three the business named map to #2 (Home), #4–6 (Checkout: three gates, grouped as Checkout) and #4 (Select address). **Decision:** report `entry` at the level the business thinks in — `home`, `checkout_address`, `checkout_schedule`, `checkout_summary` — and let the portal group the three `checkout_*` values under "Checkout" with a drill-down. This answers "which of the checkout gates converts".

### B.2 Taxonomy (low cardinality, closed set)

`entry`: `onboarding | home | tab_gate | checkout_address | checkout_schedule | checkout_summary | banner | push | deeplink | dialog | other`. `entry_detail`: free short code (≤40) such as tab name, banner id, push type; not used for grouping, only for drill-down. `source_screen`: `screen_name` from the registry. Old values `gate` and `checkout_gate` remain readable (backend maps `gate` → `other`, `checkout_gate` → `checkout_summary`).

### B.3 App changes
- Each call site passes `entry:` (and `entry_detail`) into `NewSignINScreen`; the constructor already has the argument. Default (no entry given) becomes `other` plus a debug log so no source is silent.
- Registration attempt remembers `entry` in `RegistrationStore` so later events (`registration_completed`) reuse it without parameters passing through four screens.
- After completion the app returns to where the guest was (checkout resumes at `_completeGuestAuth`). Add `registration_resumed` (`entry`, `result` `back_to_checkout`/`home`) so the portal can show whether people who register at checkout return to it. (New, small.)
- Portal metric: for each entry, registered → first purchase within 7 / 30 days (A.6).

---

## Part C. Location flow

### C.1 Flow and current tracking

| # | Step | Code | Tracked today |
|---|---|---|---|
| 1 | Open address list from Profile "My location" | `profile_screen.dart:274` → `LocationScreen` / `NewLocationScreen` | legacy `location_screen` only |
| 2 | Open address list from checkout (select address) | `confirm_and_pay_checkout_screen.dart:366-383` | `address_selected` via `CheckoutTracker` (lines 234, 407, 422, 980) |
| 3 | List actions: use current location, add new, select a saved address, edit, delete, set default | `location_screen.dart:331,341,370,477,509,536` | none |
| 4 | Select address with `returnSelectedLocation` → server **coverage check** `customer/address/check/distance` | `location_screen.dart:384` | none. Out of range shows `showBottomSheetUnavailableAddress()` silently |
| 5 | New / edit address: map, drag pin, tap map, current location, search | `new_location_screen.dart` (16 handlers), `new_search_location.dart` | none |
| 6 | Place search (Google Places) and pick a result | `location_controller.dart:359,395` | none |
| 7 | Permission prompts and failures when reading GPS | `getCurrentLocation*` (errors swallowed, toast commented out) | none |
| 8 | Save / update address (create, update APIs) and, from checkout, coverage check on the new address | `location_controller.dart:532-590,612-640` | none |
| 9 | Delete address | `location_controller.dart:643` | none |
| 10 | Return to checkout with the chosen address | checkout `:407-422` | `address_selected` |

Findings: **L-1** no event for any list/edit/delete/default action; **L-2** coverage check result and the out-of-range sheet are invisible (demand outside the service area is lost); **L-3** permission outcomes and GPS failures are swallowed; **L-4** place-search usage and pick rank unknown; **L-5** save/update/delete API failures are only `Get.log`; **L-6** `address_selected` only fires inside checkout, so addresses picked from the profile list or switched in the list before returning are not seen unless checkout re-fires (lines 407/422 do on return — verify with QA); **L-7** dead path `location_screen` vs `new_location_screen` duplicates (`isFromConfirm` vs `returnSelectedLocation`): confirm which are live.

### C.2 Events (extends App_Tracking_Coverage_Spec §3.4, which listed `address_*`)

Common params for all: `source` (`profile` | `checkout`), `screen_name`, optional `address_id`. **Never** street text, floor, room, note or exact coordinates; location is expressed as `area_cell` (rounded to 2 decimals, ~1 km, existing rule) and `zone_id` when known.

| Event | Trigger | Extra params |
|---|---|---|
| `screen_viewed` (`locations`, `location_edit`, `location_search`) | Foundation task | – |
| `element_clicked` | `add_address`, `use_current_location`, `address_card`, `edit_address`, `delete_address`, `set_default`, `map_tap`, `search_bar`, `save_address`, `back` | `element_type`, `item_id`=address id |
| `permission_result` (`permission=location`) | After the OS prompt or when already denied | `result` (`granted`/`denied`/`denied_forever`/`services_disabled`), `context` (`use_current_location`/`auto_open`) |
| `search_performed` | Place search, debounced | `search_scope=place`, `result_count`; `query` **omitted** for place search (addresses are personal) |
| `search_result_clicked` | A place suggestion is chosen | `search_scope=place`, `rank` |
| `location_pin_moved` | Map camera idle after a user drag or tap (once per idle) | `method` (`drag`/`tap`/`current_location`/`search`), `area_cell` |
| `service_area_checked` | After `checkAddressV2` returns | `in_range` (0/1), `area_cell`, `address_id`, `result` (`ok`/`timeout`/`error`), `duration_ms` |
| `address_added` / `address_edited` | Create/update returns | `method` (`current_location`/`search`/`map_pin`), `has_floor`, `has_room`, `has_note`, `in_range`, `result`, `error_code` |
| `address_deleted` / `address_default_set` | Delete/set-default returns | `result`, `was_default` |
| `form_submitted` (`form_id=address`) | Save tapped | `result`, `field_count` |
| `address_selected` | exists | add `source` and `method`=`saved_list` |
| `app_error` | Address API failure, Places API failure, coverage timeout | `error_type=api`, `endpoint_group=address`/`places`/`coverage` |

Out-of-range is `service_area_checked` with `in_range=0`; the sheet `showBottomSheetUnavailableAddress` also logs `overlay_viewed`. Platforms: Firebase for all; Meta/AppsFlyer none (privacy); backend ingest for `service_area_checked`, `address_*`, `permission_result`.

### C.3 Backend and portal
- Table `app_service_area_check (event_id PK, occurred_at, customer_id NULL, app_instance_id, source, area_cell, zone_id NULL, in_range bool, result, duration_ms)`; address events go to `app_interaction_event` (Coverage spec §6.2). Endpoint `GET /app-usage/locations`: `{summary: {checks, outOfRange, outOfRangeRate, permissionDenied}, outOfRangeAreas: [{areaCell, lat, lng (cell centre), checks, uniqueUsers}], methodBreakdown: [{method, count}], errors}`.
- Portal tab **Locations** under App Usage (T-0078): KPI tiles (address adds, out-of-range rate, permission denied rate), table and map of out-of-range cells (demand outside coverage), method breakdown, funnel from `address_added` started → success. Filters as in the App Usage spec.

---

## Open questions and decisions

Decisions (from docs and project rules):
- D-6 Portal follows the app's current registration order (A.4). No app-flow change (owner, 2 Nov).
- D-7 `entry` stays a small closed set; detail goes in `entry_detail`.
- D-8 No coordinates, street text, floor, room or notes leave the device in analytics (existing rule, restated).

Open (recommendation first):
- ~~O-7~~ **Answered 2 Nov:** no app-flow change; ad-platform event timing is unchanged (D-9). 
- ~~O-8~~ **Answered 2 Nov:** no app-flow change.
- ~~O-9~~ **Answered 2 Nov:** yes, group `checkout_*` as one "Checkout" bar with drill-down.

Additional decision: **D-9** `sign_up`, `CompleteRegistration` and any AppsFlyer registration event keep their current timing (account creation). The new Firebase `account_created` and the existing `registration_completed` give the portal the exact stage. Marketing is told that Meta registrations include unverified accounts.
