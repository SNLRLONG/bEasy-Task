# App tracking meta appsflyer

- **ID:** T-0069
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-meta-appsflyer/` · **Branch:** `feat/tracking-meta-appsflyer`
- **Tier:** T2 · extends ad-platform events; no backend contract (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-03
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §5.1, §5.2
- **Depends on:** T-0065 (same checkout/search call sites); T-0031 findings; Meta/AF consent gates unchanged
- **Touches:** `lib/app_tracking/{facebook_helper,appflyer_helper}.dart`, call sites added by T-0064/T-0065 (one line each)

## Goal
Add Meta `Search`, `Schedule`, `Contact`, `AddToCart` and AppsFlyer `af_content_view`, `af_search`, `af_add_to_cart`, `af_initiated_checkout`, `af_add_payment_info`, `af_complete_registration`, `af_login`, `af_schedule`, `af_contact`. Confirm staff exclusion applies (G-14).

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [ ] G4 run it
- [x] G5 code review  - [x] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Events visible in Meta Events Manager test events and AppsFlyer dashboard (dev)
- [ ] ATT gate and staff exclusion verified
- [x] No change to the existing 7 Meta events or to `af_purchase` (code untouched; existing tests green)
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress (3 Nov 2026)

Branch `feat/tracking-meta-appsflyer`. Tests: `test/meta_appsflyer_tracking_test.dart` (6 tests). `fvm flutter analyze --no-pub` clean; `fvm flutter test` 263 pass, only `widget_test.dart` fails (also fails on main).

### Added
Meta (`facebook_helper.dart`) and AppsFlyer (`appflyer_helper.dart`, new generic `AppFlyerHelper.logEvent`). One call per platform next to the existing Firebase call; fire-and-forget, never throw.

| Fired from | Meta | AppsFlyer | Params |
|---|---|---|---|
| `category_building_screen` `_trackViewContent` | (existing ViewContent) | `af_content_view` | `af_content_id`, `af_content`, `af_content_type=service` |
| `home_service_search_screen` `_reportSearch` (service search only) | `fb_mobile_search` | `af_search` | search string lower-cased, 30 chars; `content_type=service` |
| `CheckoutTracker.detailsCompleted` (once per checkout screen) | `fb_mobile_add_to_cart` | `af_add_to_cart` | service id, `value`, `currency=USD` (AF: `af_price`) |
| `confirm_and_pay_checkout_screen` `_trackInitiateCheckout` | (existing) | `af_initiated_checkout` | service id/name, `af_price`, USD |
| same file `_trackAddPaymentInfo` | (existing) | `af_add_payment_info` | same |
| `CheckoutTracker.slotSelected` (first slot per checkout screen) | `Schedule` | `af_schedule` | service id, value, USD |
| `DiscoveryTracking.support` | `Contact` | `af_contact` | `content_type=channel`, `channel` (telegram/phone/chat/email); no order id |
| `auth_controller` account creation (same spot as Meta CompleteRegistration) | (existing) | `af_complete_registration` | `af_registration_method` phone/telegram |
| `auth_controller` OTP success, `fromRegistration == false` | n/a | `af_login` | none |

Staff exclusion: the new events skip staff phones (`PhoneNumbers.instance.isStaff`). They are NOT limited to release builds, so they show in Events Manager / AppsFlyer from dev builds (needed for G4). ATT: no per-event change; Meta SDK stays initialised after the ATT prompt as before, AppsFlyer keeps `timeToWaitForATTUserAuthorization`.

### Deliberately NOT added
- `content_category=instant`: Instant spec section 3 says it is not added.
- Revenue on AF funnel events: `af_revenue` only on `af_purchase` (spec 5.2); funnel values use `af_price`.
- Place/address search to Meta/AF (privacy, location typing); only the service search.
- Top-up, screens, errors, settings (spec 5.1, O-5).
- No change to `CompleteRegistration`, `sign_up`, `af_purchase`, Meta `Purchase` timing or consent gates.

### Differences from spec / findings
- G-14: the EXISTING Meta and AppsFlyer sends (7 Meta events, `af_purchase`) have no staff exclusion and no release-only gate; only the new events do. Left as is (no behaviour change).
- `Schedule`/`af_schedule` is sent once per checkout screen, not on every slot change (spec has no dedupe rule; Firebase `slot_selected` still fires each time).
- Meta `Contact`: spec says `content_type=channel`; I send that literal plus a `channel` param carrying the value.

### Owner decisions needed
1. Apply staff exclusion (and release-only?) to the existing Meta/AF events (G-14)? (Gate for the NEW events: applied: release-only for new events; owner may revert.)
2. `Schedule` once per screen, or on every slot change?
3. `af_login`: fired on every successful OTP login of an existing account (there is no Firebase login event; the backend records logins). Keep, or drop until marketing defines it?
4. `Contact` param shape (literal `channel` vs the channel value in `content_type`).
5. Meta `AddToCart` fires at `booking_details_completed` (address + slot ready), as the spec says; marketing should know it is not a cart add.

### Still open
G4 device / Events Manager test-events / AppsFlyer dashboard check, G5 code review, G7 docs sync (spec 5.1/5.2 status, Meta_App_Events_Setup), G8 PR message, G11 changelog, worktree removal after merge.

## Review round 1 (fresh agent, 3 Nov 2026)

Fixed
- Release-only: new Meta and AF events now need `Helper.kMode && !isStaff`; `debugSendOverride` replaces only the release check (staff check always applies). Existing events stay ungated.
- Search text: `FacebookHelper.searchString` (shared with AF) drops the string when it has `@` or 6+ digits (same rule as `AnalyticsHelper._searchQuery`); event still sent without the param; blank query is still skipped.
- `Contact`/`af_contact` not sent when `source == 'push'`; Firebase `support_contacted` unchanged.
- `af_login` hoisted to the start of the unawaited block in `auth_controller.dart`; await order of existing calls unchanged.
- `AppFlyerHelper.logEvent` logs "not logged: SDK not initialised" when the SDK is null.
- Imports reordered in the five files named.

Accepted: InitiateCheckout before AddToCart (spec mapping, documented); `af_login` only for OTP login; AF registration skips staff while Meta CompleteRegistration does not (existing).

Left: tests cover the helpers; call sites by the device / Events Manager check (G4). Release-only means test events need a release build (or a temporary override).

After-fix results: `fvm flutter analyze --no-pub` clean; `fvm flutter test` 267 pass, only `test/widget_test.dart` fails (pre-existing). `appflyer_helper.dart` still CRLF throughout.
