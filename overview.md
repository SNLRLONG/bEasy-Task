# bEasy: Product and Tracking Overview

Written for pasting into an outside AI chat. It is self-contained. It was generated from the codebase and docs on 2026-10-21. No secrets (API keys, tokens, dev keys) are included on purpose.

Items marked **[verify]** are things I could not confirm from code and that you should check before relying on them.

---

## 1. What bEasy is

bEasy is a home-services booking product (cleaning, AC cleaning, upholstery, washing machine, post-renovation, office cleaning, instant cleaning, etc.) for Cambodia, built by Suntel Technology Cambodia. Customers book a cleaner through a mobile app, a mini-app inside ABA Mobile (a bank super-app), or are acquired through a marketing website. Staff run operations through an admin portal.

All four front-ends talk to one backend (`https://api.beasy.info/api/...`, dev at `api-dev.beasy.info`, plus a temp/staging host). Customer surface is `/api/customer`.

Languages: Khmer (default) and English only. Currency reported to ad platforms: USD.

## 2. The four products (separate git repos under `bEasy/`)

| Product | Directory | Stack | Hosting / release |
|---|---|---|---|
| Customer mobile app (iOS/Android) | `bEasy-app/` | Flutter 3.38.7 (FVM), GetX, Dio | App Store / Google Play; OTA patches through Shorebird |
| Admin portal | `bEasy-portal/` | React 19, React Router 7 (SPA), TanStack Query, Zustand, Tailwind 4, shadcn/ui | Firebase Hosting (dev and pro) |
| ABA Mobile mini-app | `beasy-mini-app/` | React 19, Vite, react-router, TanStack Query, Zustand, SSR prerender | Firebase Hosting (dev) |
| Marketing / SEO site | `bEasy-landing-page-v2/` | Next.js 16 App Router, Tailwind 4 | Firebase Hosting, site `beasy-page` |

Other folders in the workspace: `Docs/` (tracking and RBAC docs), `Tasks/` (task board: backlog, todo, in-progress, testing, done, deploy), `lsdb-bEasy/` (older sheet-DB admin/mini-app plus a BigQuery setup guide), several portal git worktrees (`bEasy-portal-*`), `CHANGELOG.md`.

### 2.1 Customer app (`bEasy-app`)

- Identity: bundle/application id `suntel.beasy.app` (dev flavor adds `.dev`, staging adds `.staging`). App Store id `6745190697`. Version at time of writing: `2.5.30+139` (iOS version comes from `pubspec.yaml`; Android name/number are passed on the build command).
- Flavors: `lib/main_dev.dart` and `lib/main_prod.dart` call `mainCommon()` with a base URL and MQTT URL. A staging entry exists as `lib/main_stagging.dart` (misspelled).
- Architecture: `get_di.dart` registers everything with `Get.lazyPut`: `DioClient` → repositories (`lib/data/repository/`) → controllers (`lib/controller/`) → screens (`lib/view/`). Navigation is mostly imperative `Get.to`.
- Chat: self-contained `lib/chat_feature/` using MQTT (`wss://api.beasy.info/mqtt`) and sockets.
- Payments: ABA PayWay (sandbox URL in constants) for checkout.
- OTP: phone OTP login/registration (Twilio Verify URL is in constants); Telegram login also exists.
- Key packages: `firebase_core`, `firebase_analytics`, `firebase_messaging` (push), `firebase_in_app_messaging`, `firebase_app_installations`, `facebook_app_events`, `appsflyer_sdk`, `app_tracking_transparency`, `android_play_install_referrer`, `app_links`, `google_maps_flutter`, `geolocator`, `mqtt_client`, `shorebird_code_push`.
- Deep links: `beasy://` scheme and AppsFlyer OneLink `https://beasy.onelink.me/n7UZ/...` (Android App Link `pathPrefix /n7UZ`). Default share link `https://beasy.onelink.me/n7UZ/6tsirqzp`.

### 2.2 Admin portal (`bEasy-portal`)

Modules include orders, calendar, cleaners/partners, partner payout and tracking, finance, categories/products/options/add-ons, bundles, coupons/vouchers/promotions, referral program, banners/popups, push notifications, chat, blocked schedule, activity log, and RBAC (roles, users). Routes are permission-gated per `module` and `action`. Axios client unwraps `response.data.data`, injects Bearer token, refreshes on 401.

The newest work is the **Customer Journey** area: funnels, acquisition, per-customer journey canvas and event log, audiences, recovery push, CS cases (CS parked for now). It reads `funnels/{funnel}`, `meta/filter-options`, `customers/{id}/journey-canvas`, `customers/{id}/activity`, `audiences`, `integrations`, `actions/recovery-push`, and a GA4/Meta-backed `dashboard/campaign-report` endpoint.

### 2.3 Mini-app (`beasy-mini-app`)

Runs inside the ABA Mobile webview. Uses `web-bridge-gateway` (`callHandler`/`registerHandler`) to fetch the ABA profile and location, then posts them to the backend to auto-login (token kept in the `token` cookie). Routes: `/`, `/order`, `/v5/detail/:bulkOrderId`, `/banner/:id`, `/service/:id`, `/checkout/:productId/:serviceId[/location|/add-location]`. vConsole is injected in dev/uat; production strips `console.*`. **No analytics/ad SDK code was found in this repo.**

### 2.4 Landing site (`bEasy-landing-page-v2`)

`middleware.ts` redirects every path to `/{km|en}/...`. Pages: home, services (slugs), media/guides (SEO articles), about-us, contact, faq, install, privacy-policy, term-of-condition. Service slugs: 2 deep-cleaning, 3 office-cleaning, 4 upholstery-cleaning, 6 washing-machine-cleaning, 7 post-renovation-cleaning, 8 ac-cleaning, 9 instant-cleaning. General Cleaning (1) and Pest Control (5) are hidden by removing their slug. Each service's "Book Now" button is an AppsFlyer OneLink so web visitors are attributed and deep-linked into the app (or the store). Old numeric URLs 301 to slugs.

---

## 3. Tracking and attribution: the big picture

```
Ad platforms (Meta, TikTok, Google) ──click──▶ AppsFlyer OneLink / Play referrer
                                                   │
Landing site (web-v2 events, UTM) ─ OneLink ─▶     ▼
                                          bEasy mobile app
          ┌────────────────────┬──────────────────┬───────────────────────┐
          ▼                    ▼                  ▼                       ▼
   Firebase Analytics     Meta (FB SDK)     AppsFlyer SDK        Backend API headers
   (GA4 → BigQuery)       app events        (install, purchase)  (journey/campaign ids)
          │                                                              │
          └──────────────▶ Backend (BigQuery import, order lifecycle) ◀──┘
                                       │
                                       ▼
                     Admin portal: Customer Journey, Acquisition, Campaign report
```

Four systems receive data from the app, each with a different job:

| System | Role | Gated how |
|---|---|---|
| Firebase Analytics (GA4) | Custom journey events, funnel, BigQuery export that the backend imports | Only when `Helper.kMode` (release) and the phone is not a staff number (`_sendEnabled`) |
| Meta (Facebook App Events) | Ad optimisation and conversion reporting for Meta campaigns | Not gated by `_sendEnabled`; iOS ATT prompt runs first |
| AppsFlyer | Install attribution, deferred deep links, `af_purchase` revenue | Not gated by `_sendEnabled` |
| Backend API | Order/lifecycle truth, identity linking | Optional tracking headers; failures never block orders |

### 3.1 App start sequence (`lib/main.dart`)

1. Paint a branded splash immediately.
2. `_initTracking()`: request iOS App Tracking Transparency (60 s timeout), then `FacebookHelper.initialize()`.
3. In parallel: dependency injection and `_initFirebase()` (Firebase init, In-App Messaging enabled but messages suppressed, GA4 `app_open` logged, Firebase installation id fetched).
4. `_initAttribution()`: `AppFlyerHelper.init()`, `scheduleAppOpened()`, Android install-referrer fallback.
5. `_listenForAppLinks()`: for the initial link and every incoming link, store Facebook ad attribution from the URI and hand it to `AppFlyerHelper.handleUri`.

---

## 4. Firebase / GA4

**Project wiring:** `firebase_core` plus FlutterFire; Android has Firebase Messaging default channel/icon meta-data and in-app messaging. Web landing page uses `firebase/analytics` with `NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID` (dynamic import, buffers events until ready).

**Collection toggling:** the helper calls `setAnalyticsCollectionEnabled(true)` before each log and `false` after. `trackInstallOnce` logs `install_amount` once per install in release.

### 4.1 Common parameters on every custom event (`AnalyticsHelper.track`)

`event_id` (UUIDv4), `schema_version` (currently 1), `seq` (increments per launch). With a service: `service_id` and `journey_id` (UUIDv4, reused until 24 h idle). With registration: `reg_flow_id`. Values are sanitised: snake_case keys ≤40 chars, strings ≤100 chars, `value`/`duration_hours`/`final_value` forced to double, booleans as 0/1, nulls dropped.

`EventContract` validates required params per event in development and logs `INVALID TRACKING EVENT`. The backend quarantines events that break the contract.

### 4.2 Identity and user properties

- `user_id`: the backend customer id only (never phone or email). Cleared on logout.
- `auth_state`: `registered` or `guest`.
- `utm_campaign`, `utm_source`: set from AppsFlyer / deep-link data.
- `first_campaign`: set once per install.
- Last-touch campaign stored locally for **7 days** (`AttributionStore`), and sent to the API as the `X-Campaign-Code` header at account creation.

### 4.3 Journey events (backend contract)

App/session: `app_opened` (with `entry_type`: `organic`, `push`, `deeplink`; plus campaign fields), `first_open`, `homepage_viewed`, `service_list_viewed`, `screen_viewed`, `app_backgrounded`, `app_foregrounded`, `banner_viewed`, `banner_clicked`.

Service and booking: `service_viewed`, `booking_started`, `address_selected`, `service_option_selected`, `duration_selected`, `date_selected`, `time_selected`, `booking_details_completed`, `slot_selected`, `booking_summary_viewed`, `promo_code_entered`, `promo_applied`, `promo_rejected`, `checkout_started`, `payment_started`, `payment_failed`, `payment_returned_success`, `order_created`, `order_confirmation_viewed`.

Add-ons and extras: `addon_group_viewed`, `addon_selected`, `addon_quantity_changed`, `addon_removed`, `task_info_viewed`, `equipment_viewed`, `pair_service_viewed/selected/added/removed`.

Post-purchase: `reschedule_started`, `reschedule_slot_selected`, `reschedule_completed`, `reschedule_failed`, `order_rescheduled`, `review_submitted`.

Registration: `registration_started`, `otp_viewed`, `otp_submitted`, `otp_failed`, `otp_resend_clicked`, `otp_verified`, `signup_viewed`, `registration_completed`.

Push: `push_received`, `push_opened`, `notification_inbox_viewed`, `notification_detail_viewed`.

### 4.4 Legacy / GA4 standard events

`view_content`, `select_service`, `begin_checkout`, `add_payment_info`, `order_detail_screen`, `splash_cta_button`, `splash_skip_button`, `splash_ad_screen`, `create_account_button`, and standard GA4 `sign_up` (method `phone` or `telegram`), `app_open`, `purchase` (via `logPurchase`, so GA4 revenue metrics populate; currency USD, `transaction_id` = order id), `campaign_details`.

Campaign attribution events: `referrer_install` (Play referrer), `skan_install`, `skan_open`, and an event named after the AppsFlyer `media_source` on a deep link.

The purchase also closes the local journey, so the next view of that service starts a new `journey_id`.

### 4.5 Backend ingestion

GA4 exports to BigQuery; a backend scheduler imports recent events, deduplicates by `event_id`, and reconciles order lifecycle (paid, cancelled, completed) from order data using deterministic event ids. Tracking is dispatched after DB commit and never blocks login or order creation. **[verify]** the 4 Oct 2026 review says BigQuery credentials, dataset, scheduler, and a staging end-to-end test still needed confirmation; check `Docs/tracking/v2/backend-response/` for the newest status.

---

## 5. Meta (Facebook) tracking

**SDK:** `facebook_app_events`. Android manifest sets the Facebook App ID/Client Token (from string resources), `AutoLogAppEventsEnabled=false`, `AdvertiserIDCollectionEnabled=true`. iOS Info.plist: Facebook App ID `1466778561367463` (URL scheme `fb1466778561367463`), auto-log off, advertiser id collection on, ATT usage string, and SKAdNetwork identifiers.

**Why auto-log is off:** ATT must run first. After ATT, `FacebookHelper.initialize()` enables auto-log and advertiser id, then explicitly calls `activateApp()` (this is the Meta "first app open" `fb_mobile_activate_app`) and `flush()`.

**Events sent** (all flushed immediately; currency forced to USD):

| Trigger in the app | Meta event | Params |
|---|---|---|
| Service page opened | `fb_mobile_content_view` (ViewContent) | `service_name` |
| Service selected, booking started | `SelectService` (custom) | `service_name` |
| Checkout summary screen | `fb_mobile_initiated_checkout` | `value`, `currency`, `service_name` |
| Payment method chosen | `fb_mobile_add_payment_info` | `value`, `currency`, `service_name` |
| Booking list/detail opened | `ViewBooking` (custom) | `service_name`, `order_id` |
| New account created | `fb_mobile_complete_registration` | method: `phone` or `telegram` |
| Payment confirmed | `fb_mobile_purchase` | `value`, `currency`, `order_id`, `service_name`, `booking_status`, `payment_status`, `booking_source=app`, `booking_id`, `coupon`, plus Facebook attribution fields if recent |

**Advanced Matching:** after login/profile load, `setUserData` is called with email (lower-cased), phone, first/last name, and external id. The SDK hashes them.

**Facebook ad attribution store:** any incoming link with `utm_source`, `source`, `utm_campaign`, `campaign`, or `fbclid` is stored in local prefs for **7 days**. While fresh, purchase events carry `fb_attribution_source`, `fb_campaign`, `fb_clicked_at`, `fb_attribution_present`.

Design notes live in `bEasy-app/docs/facebook_attribution_*.md` and `facebook_ad_view_vs_click_attribution.md`.

---

## 6. AppsFlyer

- SDK `appsflyer_sdk`; iOS app id `6745190697`; ATT wait up to 30 s. **The dev key is hard-coded in `appflyer_helper.dart` (not reproduced here); consider moving it to config.**
- Listeners registered before `initSdk`: install conversion data (GCD), app-open attribution, and Unified Deep Linking (UDL).
- **Campaign resolution** (`CampaignTouch`): campaign = `utm_campaign` → `campaign` → `c`; source = `utm_source` → `media_source` → `pid`; medium = `utm_medium` → `af_channel`; service = `category_id`, else numeric `deep_link_value`.
- On a touch: save last-touch (7-day TTL), set `first_campaign` once, forward UTM data to Firebase as `campaign_details` and user properties, and fire `app_opened` with `entry_type=deeplink` (deduplicated within 5 s, because UDL, app-open attribution and handleUri all fire for one click).
- **Organic open:** if no link resolves within 4 s of launch, `app_opened` fires as `organic` (or `push` if cold-started from a push notification).
- **Install handling:** only a first-launch, non-organic install counts as a touch, so replayed conversion data does not refresh the 7-day window.
- **Deferred deep linking** routes to a service category using `category_id`, or `deep_link_value` mapped via a shortcode table (e.g. `deep_cleaning`→2, `ac_cleaning`→15, `upholstery`→9, `washing_machine`→17, `post_renovation`→16, `office_cleaning`→7, `pest_control`→13, `general_cleaning`→1, `commercial_cleaning`→14, and OneLink short ids). Special value for claiming a voucher (`claim_voucher`). **[verify]** the mobile category ids (e.g. 15 for AC) differ from the landing-site service ids (e.g. 8 for AC); confirm the mapping is intentional.
- **Android Play Install Referrer fallback:** runs once per install (guarded by a persisted flag). Parses `utm_*`, `campaign`, `c`, `category_id`, `deep_link_value`, etc., logs `referrer_install`, records the touch, and may deep-link. Needed because AppsFlyer returns 404 on the dev flavor and Play can deliver deferred attribution.
- **Revenue:** `af_purchase` with `af_revenue`, `af_currency`, `af_order_id`, `af_coupon_code`, sent on payment success, so the campaign that drove the first open also gets revenue.
- Custom-event fan-out on purchase (one booking triggers all three): Meta `fb_mobile_purchase`, AppsFlyer `af_purchase`, GA4 `purchase`. A SharedPreferences flag per order id prevents double-counting.

---

## 7. TikTok

What exists in the code today:

- **No TikTok SDK or TikTok event code was found in the app** (no package in `pubspec.yaml`, no TikTok event calls). The only technical trace is `TikTokAppID` in `ios/Runner/Info.plist`.
- TikTok is a **social/profile link** (`https://www.tiktok.com/@beasy_kh`) in the app's About screen and footer of the landing site (`footer_media_click`).
- TikTok is an answer option on the in-app "how did you hear about us" screen (`customer_source_screen.dart`).
- **Attribution for TikTok ads is therefore expected to flow through AppsFlyer** (TikTok as a self-reporting/ad network, with `media_source` and campaign carried by OneLink), then GA4 user properties and `app_opened` campaign fields. **[verify]** the AppsFlyer dashboard integration with TikTok For Business and whether TikTok postbacks (install, purchase) are enabled; this is not visible from code.
- Backend/portal side: the Acquisition API has a `channel` filter (`all`, `meta`, `tiktok`, `other`). The channel table anticipates `tiktok_ads`. Audiences (Push, Meta Custom Audience, TikTok, CSV) are planned and hidden in the portal.
- **TikTok ad clicks / spend are not imported yet.** See section 9.

---

## 8. Web tracking

**Landing site (`bEasy-landing-page-v2`):** Firebase Analytics via `lib/analytics.ts`; `trackEvent()` prefixes `web-v2:`. Events: `page_view`, `section_view`, `telegram_click/close`, `language_change`, `footer_media_click` (Telegram, Facebook, TikTok, Instagram), `download_click`, `contact_click`, `contact_navigate(_fail)`, `service_card_click`, `guide_cta_click`, `video_play/close`. Auto params: stored first-visit `utm_source/medium/campaign`, `referrer`, `is_direct`. Catalogue file: `EVENTS.md` (must be updated when adding events). No Meta Pixel or TikTok Pixel was found in the code. **[verify]** whether pixels are injected elsewhere (e.g. a tag manager in hosting config).

**Mini-app and portal:** no marketing-analytics SDKs found. The portal only *reads* campaign and journey data from the backend.

---

## 9. Portal: Customer Journey, Acquisition, and ad data status

Source of truth: `Docs/tracking/v2/README.md` and sub-folders (`requirements`, `architecture`, `api`, `backend-request`, `backend-response`, `archive`).

- Metric/funnel stages on Acquisition: App opened → … → Purchased, plus **Ad clicked**, drop-offs, people lists, after-purchase block (reschedule/cancel/refund).
- **Works today:** stages from App opened to Purchased, drop-offs, people lists (data starts the day guest mode went live).
- **Blocked on ad platforms:** the *Ad clicked* stage, `ad_clicks` and `click_to_buy` KPIs, campaign table `adClicks`/`clickToBuy`, and ad-set filter. The API returns `null` with `unavailableReason: meta_not_connected`; ad-set filter returns 422.
- **Who must act (from `Acquisition_Ad_Data_Readiness.md`):**
  1. Marketing/business owner: grant backend read access to the Meta ad account (and TikTok), name the Business Manager owner and system-user token issuer.
  2. Backend: daily import of clicks, impressions, spend per campaign/ad set, with backfill; flag `integrations.meta.connected`.
  3. Backend: use the stored attribution touch when `app_opened` lacks campaign fields.
  4. Mobile: send real campaign names on app opens, keep `X-Campaign-Code` at account creation, set Firebase `user_id` on login; decide the deep-link provider (Firebase Dynamic Links shut down in 2025, AppsFlyer OneLink is current).
  5. Portal: wired to the contract; polish empty states.
- **Open decisions:** which click metric (all clicks vs link clicks), join key `meta:<campaign name>` (names must match exactly), history/backfill depth, how filters that cannot split ad totals behave (`filter_not_applicable`), refresh cadence, whether to import spend now.
- Per-campaign funnels currently show mostly "Organic" until the app sends campaign names consistently.

---

## 10. Operating rules (from `AGENTS.md`, useful context)

- Never `git commit` or `git push` without explicit instruction in that session.
- One task = one worktree = one branch. Task files live in `Tasks/<stage>/`, tier T0-T3 picked by the agent; gates depend on tier. Tracking contract changes with the backend are T3.
- The backend repo (`bEasy-api`) is read-only evidence.
- Journey docs: one fact in one place (open asks → `backend-request/pending-request-backend.md`, replies → `backend-response/`, endpoint shapes → `api/Journey_Backend_API_Spec.md`, mobile events → `architecture/`).
- Merged work is logged in the root `CHANGELOG.md` with prefixes `[Portal] [App] [Mini-App] [Landing-Page]`.

## 11. Known gaps and risks (observed)

1. TikTok has no direct measurement in code; all of it depends on AppsFlyer configuration **[verify]**.
2. Meta and AppsFlyer events are not gated by the staff-number / release check that gates Firebase, so test or staff bookings could count toward ad platforms **[verify]**.
3. AppsFlyer dev key is hard-coded in source.
4. Service id mapping differs between the app, landing site, and OneLink shortcodes.
5. Meta/TikTok ad data (clicks, spend) is not yet imported, so Ad clicked, CAC and ROAS are empty in the portal.
6. Landing site has Firebase Analytics only; no pixel found for retargeting audiences.
7. `CLAUDE.md` says `main_staging.dart` is missing, but `main_stagging.dart` exists (typo).
8. Guests appear as devices (via `app_instance_id`), not people, in audiences and exports.

## 12. Key files to point an AI at

| Topic | File |
|---|---|
| Startup and SDK order | `bEasy-app/lib/main.dart` |
| GA4 events and params | `bEasy-app/lib/analytics/analytics_helper.dart`, `event_contract.dart`, `tracking_ids.dart` |
| Campaign last-touch | `bEasy-app/lib/analytics/attribution_store.dart` |
| AppsFlyer | `bEasy-app/lib/app_tracking/appflyer_helper.dart` |
| Meta | `bEasy-app/lib/app_tracking/facebook_helper.dart` |
| Purchase fan-out | `bEasy-app/lib/controller/payment_controller.dart` |
| Registration events | `bEasy-app/lib/controller/auth_controller.dart` |
| Platform config | `bEasy-app/ios/Runner/Info.plist`, `android/app/src/main/AndroidManifest.xml` |
| Tracking architecture | `Docs/tracking/v2/architecture/bEasy-tracking-architecture-guide.md` |
| Backend contract | `Docs/tracking/v2/architecture/tracking-backend.md`, `api/Journey_Backend_API_Spec.md` |
| Ad data readiness | `Docs/tracking/v2/architecture/Acquisition_Ad_Data_Readiness.md` |
| Landing events | `bEasy-landing-page-v2/EVENTS.md`, `lib/analytics.ts` |
