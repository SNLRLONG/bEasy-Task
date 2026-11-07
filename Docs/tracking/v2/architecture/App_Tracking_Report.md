# bEasy App — What we plan to track

Updated: 2026-11-06 · Audience: management, marketing, business and data analysis · Status: tracking is built in the app and is going through testing before release.

This report explains, in plain terms, which customer behaviour the mobile app will record, what each record helps us decide, and what we have chosen not to track.

---

## 1. Summary

Today the app records the booking and sign-up journey well, and almost nothing else. We cannot see how customers use chat, notifications, Profile, top-up, addresses or search, and we cannot see when something fails.

The plan keeps the existing booking and sign-up tracking and adds coverage for every screen customers can actually reach. It records:

- which screens are opened and how long customers stay;
- which buttons and cards are tapped;
- which pop-ups appear;
- which searches are made;
- which errors customers hit.

This only adds measurement. No screen, step or price in the app has changed.

## 2. What we will be able to answer

| Area | Questions the data answers |
|---|---|
| **App start and system** | Do customers skip the splash ad? How many hit no internet, a server error or a forced update? |
| **Home and discovery** | Which services, banners, promotions and bottom tabs are used most? How often do guests tap a tab that needs sign-in? |
| **Search** | What do customers search for? Which searches find nothing (unmet demand)? *Ready, but the search screen is not currently reachable, see section 6.* |
| **Notifications** | Do customers open the inbox? Which announcements get tapped? |
| **Registration** | Where do sign-ups start (onboarding, home, checkout, banner, push)? At which step do customers drop, including the OTP code step? |
| **Location and address** | Do customers get stuck on the map, location permission or the service-area check? How many addresses are added, edited or deleted? |
| **Service and booking** | Where do customers leave between the service page, schedule and checkout? Which options and add-ons do they look at? |
| **Instant Cleaning** | How often is it fully booked (lost demand)? Which hours and rooms are chosen? Do customers switch to Deep Cleaning? Is a same-day slot picked? |
| **Checkout and payment** | Which payment method and coupon are used? Which error stops an order? |
| **Top-up (wallet)** | How many customers start a top-up, choose an amount and reach payment? |
| **Orders** | Which tab and action do customers use on an order (pay now, book again, review)? How many leave a review? |
| **Extend hours** | Do customers extend a job in progress? |
| **Reschedule** | How many customers reschedule, to which times, and why does it fail? |
| **Profile and settings** | What do customers use in Profile? Which language do they choose? |
| **Support** | How often do customers contact us, and by which channel (Telegram, phone, chat, email) and from which screen? |
| **Chat** | Is chat used, and how often are messages sent? Message content is never recorded. |
| **Errors and health** | What fails, on which screen, how often, and for how many customers? |

## 3. What is tracked today, screen by screen

This is the full list of what the app records as of 6 Nov 2026. Names in sections 3.1 to 3.12 are plain-language; the exact event and parameter names that are stored are in section 9. Not yet verified on a device; the comparison against the current app is still to do.

### 3.1 On every screen (automatic)

| What | Detail |
|---|---|
| Screen opened | Which screen, which screen the customer came from, and how they got there (tap, tab, push, deep link, back). |
| Screen left | How long they stayed. Visits under 1 second are ignored, visits over 30 minutes are capped, and time in the background is not counted. |
| App session | App opened, sent to background, brought back, with session length and time away. |
| Push notification | Received, and opened (with the screen it led to). |
| Who | The customer id is attached after login. Staff and test phones are excluded from the new events. |

### 3.2 Launch and system

| Screen | Tracked |
|---|---|
| Splash ad | Ad shown, Skip tapped, ad button tapped. |
| No internet | Error recorded at start, Retry tapped. |
| Server error | Error recorded (profile load failed), Retry tapped. |
| Update prompt (optional or forced) | Prompt shown (forced or optional), Update now, Later. |

### 3.3 Sign-in and registration

| Screen | Tracked |
|---|---|
| Welcome | Viewed. |
| Sign in | Where the sign-up started (onboarding, home, tab gate, checkout step, banner, push, link), guest mode tapped, contact support tapped. |
| Sign up | Viewed, Create account tapped, form submitted (with result), contact support tapped. |
| OTP code | Viewed, code typed, code submitted (attempt number), failed (error code only), resend, verified, contact support. |
| Customer source ("how did you hear") | Option chosen, Continue, Skip, contact support, form result. |
| Notification prompt | Viewed, Continue, Not now. |
| Completion | Account created, registration completed (method), Meta and AppsFlyer sign-up event. Returning customers send an AppsFlyer login event. |

### 3.4 Home, discovery and search

| Screen | Tracked |
|---|---|
| Bottom tabs | Each tab tapped (home, bookings, notifications, chat, profile). A guest tapping a tab that needs sign-in is recorded separately. |
| Home | Viewed (home page and service list), service card tapped, See all, retry services, refresh, language picker, sign-in card (guests), promo floating button, home load error. |
| Home pop-ups | Promo pop-up (shown, tapped, closed), Lucky Draw pop-up, "coming soon" dialog. |
| Home banners | Banner seen (with placement), banner tapped. |
| Latest from bEasy | Turn on notifications, Join Telegram, promo card tapped. |
| Telegram floating button | Tapped (counts as a support contact), closed. |
| See more services | Viewed, service tapped, coming-soon dialog, contact support, promo code copied. |
| Service search | Search made (result count, text shortened to 30 characters), result tapped (rank). Also sent to Meta and AppsFlyer. *Screen is not currently reachable, see section 6.* |
| Banner detail | Viewed, image zoom, button or link tapped, coupon picked, terms ticked, payment method chosen, receipt upload (take photo, choose album, cancel, submit), guest sign-in prompt and Continue. |

### 3.5 Service detail and booking

| Screen or step | Tracked |
|---|---|
| Service detail | Service viewed (also Meta and AppsFlyer View Content), service selected, booking started, Schedule appointment tapped, Back, contact support, retry, "show all options", task info and equipment cards (viewed), service options failed to load. |
| Options chosen | Address, option, duration, date, time, "booking details complete". |
| Add-ons | Add-on group viewed, add-on chosen, quantity changed, removed, Done. |
| Pair services (cross-sell) | Viewed, chosen, added, removed; sheet Confirm or Cancel. |
| Category alert | Alert shown, then its button or close. |
| Sub-category | Sub-category chosen, detail sheet closed. |
| Unavailable-schedule sheet | Closed. |
| Instant Cleaning | Availability checked, fully booked shown, duration, rooms toggled, "too few rooms" warning, upsell to Deep Cleaning (tapped and result), time slot picked (same-day flag), grace-period result. |

### 3.6 Checkout and payment

| Screen or step | Tracked |
|---|---|
| Checkout | Summary viewed, checkout started, Back, Refresh price, Change schedule, Read full policy, contact support, "payment method" chosen. Meta, AppsFlyer and Firebase receive start checkout and add payment info; Meta and AppsFlyer also receive add to cart and schedule. |
| Schedule sheet | Confirm, Close. |
| Coupons | Coupon list viewed (count), coupon tapped, code searched, code entered, applied or rejected (reason), Apply coupon, Use other promo. |
| Placing the order | Payment started, order created, confirmation viewed, payment failed (error code only), payment returned success, purchase (the only event that carries revenue). |
| Payment result sheet | Success: Close, Back home. Failed: Close, Try again. |
| Payment web view and app hand-off | Errors only: timeout, page load failure, invalid link, bank app could not open. |
| Appointment status screen | Back, Start booking, Back home. |
| Low balance sheet | Top up from balance tapped, closed. |
| Errors | Price preview, coupon, order create, no schedule slots, payment methods and status failures, with an error code. |

### 3.7 Top-up (wallet)

| Screen | Tracked |
|---|---|
| Top-up | Viewed, top-up started, amount chosen (preset with position, or typed), Confirm, payment method chosen, payment started, Back. |
| ABA payment and status | Status viewed, Done tapped, Back; timeout and web view errors. |
| Top-up history | Viewed, New top-up tapped, item opened, detail closed, Back. |
| Errors | Amounts failed to load, top-up could not be created. |

### 3.8 Orders

| Screen | Tracked |
|---|---|
| Bookings list | Viewed (also Meta View Booking), Active or History tab, Refresh, Explore services, order card tapped. |
| Order detail | Viewed, Refresh, Back, contact support and Contact us, Pay now, Book again, Leave review, Swipe to complete, Extend hours, Service add-on. |
| "Cleaners not finished" dialog | Not yet, Done. |
| Order add-ons | Add, increase, decrease, Checkout, Close. |
| Extend hours | Started, payment method chosen, Confirm, Done, closed. |
| Instant job in progress | Progress stage viewed. |
| Reschedule | Started, slot chosen, grace period requested, submitted, completed, failed (reason), abandoned (step), "order rescheduled". |
| Cancel | Policy viewed from checkout and order detail. The cancel option itself is switched off in the app, so the cancel events (started, reason, submitted, result, abandoned, switched to reschedule) are built but record nothing today. |
| Leave review | Star chosen, tags chosen, Submit, review submitted (stars only), errors. Review done: Close. |

### 3.9 Notifications

| Screen | Tracked |
|---|---|
| Inbox | Inbox viewed (unread count), Notifications or Announcements tab, Refresh, Mark all read (confirm, cancel), announcement tapped. |
| Notification detail | Viewed (opened from inbox or push), sheet button, sheet closed, contact via Telegram. |
| Permission banner | Not now, Open settings, push permission granted. |

### 3.10 Profile and settings

| Screen | Tracked |
|---|---|
| Profile | Viewed, every menu row (profile info, my location, my coupons, language, share app, help centre, contact support, about, logout), logout confirm, push permission banner. |
| Profile info | Viewed, Save (with result), change avatar (take photo, choose album, save), gender, date of birth, delete account and confirm, camera and photo permission results. |
| Language | Option tapped, language changed (from, to, and which screen), language saved as a customer property. |
| My locations | Viewed, Add address, use current location, address card, Edit, Delete (with result), location permission result, service-area check, coverage timeout. |
| Add or edit address | Save (address saved), Back, search bar, use current location, map pin moved (tap, drag, search, current location; area rounded to about 1 km), place search and result tapped. |
| Help centre | FAQ, Terms, Privacy, Email, Telegram, Phone (each counts as a support contact where it is one). |
| Tutorials | Category, video card, play or pause, full screen, related video. |
| About bEasy | Website, Facebook, Instagram, TikTok, Telegram, RedNote. |
| My coupons | Refresh, push permission banner. |
| Contact support sheet | Telegram, phone, email. |

### 3.11 Chat

| Screen | Tracked |
|---|---|
| Chat list | Chat opened (order chat or support chat), Refresh, Place order. |
| Chat room | Chat opened, message sent (text, image or video only; never the content), camera attach, camera permission result, Back. |
| Story camera and media | Shutter, switch camera, gallery, Close, photo permission result; preview Retake, Send, Back; gallery pick; media opened. |

### 3.12 By use case (the order events are recorded in)

| Use case | Steps recorded |
|---|---|
| Sign up | Registration started (entry) → sign-up viewed → OTP viewed → code typed → submitted → verified or failed → account created → registration completed. |
| Book a service | Home or banner → service viewed → booking started → address → options and add-ons → duration → date → time → details complete → schedule → add to cart → checkout started. |
| Instant Cleaning | Availability checked → fully booked or slot picked → duration and rooms → warnings and upsell → checkout. |
| Pay | Payment method → payment started → order created → confirmation viewed, or payment failed → purchase. |
| Use a coupon | Coupon list viewed → coupon picked or code entered → applied or rejected. |
| Top up | Top-up started → amount → payment method → payment started → status screen. |
| Manage an order | Bookings list → order detail → pay now, extend, add-on, reschedule, review or book again. |
| Contact support | Support contacted, with the channel (Telegram, phone, chat, email) and the screen it came from. |
| Notifications | Push received → push opened → notification or announcement viewed. |
| Address | Add address → pin moved → service-area check → address saved, edited or deleted. |
| Errors | Error type, screen and error code for network, server, payment, web view and update failures. |

## 4. Principles

1. **No personal data.** We never record phone numbers, names, address text, chat messages, card details or what a customer types in free-text fields (for example the cancel "other" reason). Locations are rounded to about 1 km. Search text is recorded only for the service search, lower-cased and shortened to 30 characters, and dropped if it looks like an email or phone number. Promo codes are recorded (they are needed to measure coupons).
2. **One consistent system.** Every screen and button has a fixed name, so reports stay comparable across screens. Khmer and English labels are managed in the portal.
3. **Existing reports keep working.** Old events continue for one release alongside the new ones, so current Firebase and Meta audiences do not break. We retire the old ones only after marketing confirms nothing depends on them.
4. **Staff and test phones are excluded** from the new tracking, so internal test orders do not distort the numbers.
5. **Money and order status come from the backend.** Customer-side events for payment or top-up are only for diagnosis, not for revenue figures.

## 5. Advertising platforms (Meta and AppsFlyer)

| Platform | Already receives | Will also receive |
|---|---|---|
| **Meta** | View service, select service, start checkout, add payment info, view booking, complete registration, purchase | Search, add to cart, schedule, contact |
| **AppsFlyer** | Installs and campaign attribution, purchase | View service, search, add to cart, start checkout, add payment info, schedule, contact, complete registration, login |

For marketing:

- **"Add to cart" means the address and time are ready.** The app has no real cart.
- **Start checkout comes before add to cart** in the data. Keep that order in mind when building funnels.
- **Revenue is sent only with the purchase event.** Funnel events carry a price but no revenue, so revenue is not counted twice.
- **Top-up, errors and settings are not sent** to ad platforms, because they add no ad-optimisation value.

## 6. Not tracked: features that are switched off or unreachable

These are commented out or cannot be opened in the current app, so there is nothing to measure. If one returns, we add its tracking then.

| Feature | Reason |
|---|---|
| **Cancel booking** | The cancel option is switched off in order detail, so customers cannot cancel in the app today. The tracking is prepared but records nothing until the option returns. Cancellation numbers come from the backend. |
| Home search bar and the search screen | The search bar is gone and no button opens the search screen. Search tracking is ready for when it returns. |
| Home: notification bell, greeting avatar, top-up and history cards | Removed or switched off on Home. |
| Banner share and PDF open | No share button exists; the PDF view is switched off. |
| Profile: payment methods, notifications, chat rows | Switched off or not present. |
| Change password, change avatar and payment methods screens | Not reachable. |
| Address "set as default" | Switched off. |
| Order detail (old version), call, chat or view cleaner buttons | Not in the current order screen. |
| Notification delete and per-item "mark as read" | No such control; tapping a card already marks it read. |
| Checkout: add note, remove coupon, old payment and schedule pickers | Not in use or add little value. |
| Top-up keypad, note and receipt upload | Not reachable. |
| Lucky draw screen | Seasonal campaign; only the pop-up that opens it is tracked. |
| Instant Cleaning info sections | Static text, nothing to tap. |
| Coupon copied or used (My coupons) | Not planned for now. |

## 7. Open decisions and remaining work

| Item | Detail | Recommendation |
|---|---|---|
| **Crash monitoring** | The app has no crash reporting today. | Add Firebase Crashlytics. |
| **Retire old events** | Old and new events run side by side for one release. | Retire the old ones after marketing confirms no audience depends on them. |
| **Staff exclusion for existing ad events** | The original Meta events and the AppsFlyer purchase still include staff phones. | Exclude staff phones. Needs owner approval. |
| **AppsFlyer login event** | It fires on every login by an existing customer. | Decide whether to keep it. |
| **Chat connection errors and media upload results** | Not tracked yet. | Plan as a separate piece of work. |
| **Guest returns to checkout after signing up** | Not tracked yet. | Add as a follow-up. |
| **Customer properties not filled yet** | `location_permission` and `has_ordered` are defined but the app does not set them yet (`has_ordered` is only reset to 0 on logout). | Decide whether to fill them or drop them before audiences are built on them. |
| **Backend** | The backend must store the new events and provide the data for the portal's App Usage pages. | Request has been prepared for the backend team. |
| **Testing** | Compare the new app against the current one before release. | Required before release. |

## 8. Technical detail

For the full event list, rules and backend design, see the tracking specifications in this folder: App Tracking Coverage Spec, Registration and Location Flow Spec, Cancellation and Reschedule Spec, Instant Cleaning Tracking Spec, and the App Tracking Backend Handoff.

---

## 9. Raw events and parameters

Exact names as stored in Firebase. Use this section to build reports, audiences and funnels. "Common" parameters (9.2) are on every custom event and are not repeated in the tables.

### 9.1 How events are sent

- Custom events go to **Firebase Analytics**. Firebase standard events are also used: `purchase`, `sign_up`, `app_open` and campaign details (9.6).
- Meta and AppsFlyer receive a separate, smaller set (9.7).
- Events are sent from release builds only. Staff and test phones are excluded from Firebase events and from the new Meta and AppsFlyer events.
- Parameter names are `snake_case`, 40 characters at most. Text values are trimmed to 100 characters. True or false is sent as `1` or `0`. `value`, `duration_hours` and `final_value` are always decimal numbers. Empty values are not sent. `currency` is always `USD`.

### 9.2 Common parameters

| Parameter | Type | Meaning |
|---|---|---|
| `event_id` | UUID v4 | Unique per event, used to de-duplicate. |
| `schema_version` | int | `1`. Raised when parameters change shape. |
| `seq` | int | Counter within one app launch, orders events with equal timestamps. |
| `service_id` | int | On service and booking events. |
| `booking_mode` | text | `instant` or `scheduled`, with `service_id`. |
| `journey_id` | UUID v4 | One booking attempt for one service. Reused until 24 hours without activity, closed by the purchase. On service and booking events. |
| `reg_flow_id` | UUID v4 | One registration attempt, reused until 24 hours without activity. On registration events. |
| `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_id` | text | Last-touch campaign (valid 7 days). Added to `app_opened`, `homepage_viewed`, `service_list_viewed`, `service_viewed`, `booking_started`, `checkout_started` and every registration event. |

### 9.3 Customer identity and properties

| Name | Values | Set when |
|---|---|---|
| `user_id` | backend customer id | After login or registration; cleared on logout. Never a phone number or email. Guests have none. |
| `auth_state` | `registered`, `guest` | Every launch and login or logout. |
| `app_language` | `km`, `en`, `vi`, `zh-cn`, `zh-tw` | App start and every language change. |
| `push_enabled` | `granted`, `denied` | When the push permission result is known. |
| `registration_entry` | closed list (9.5, Registration) | Registration completed. |
| `first_campaign` | campaign code | Once per install. |
| `utm_campaign`, `utm_source` | text | When a campaign link is resolved. |
| `location_permission`, `has_ordered` | | Defined, not filled yet (section 7). |

### 9.4 What the backend receives and requires

Headers on API calls: `X-App-Instance-Id` (Firebase app instance id, all calls), `X-App-Version`, `X-App-Platform`, `X-Journey-Id` (booking preview, coupon, calculation, create-order, checkout and add-on calls), `X-Reg-Flow-Id` (send OTP, login, create account, phone status check), `X-Campaign-Code` (create account and login).

The backend rejects (quarantines) a contract event that is missing a required parameter, or whose ids are not valid UUID v4. These parameters are required in addition to `event_id`, `schema_version` and `seq`:

| Event | Required |
|---|---|
| `service_viewed`, `booking_started`, `order_confirmation_viewed` | `journey_id`, `service_id` |
| `address_selected` | `journey_id`, `service_id`, and `zone_id` or `area_cell` |
| `service_option_selected` | `journey_id`, `service_id`, `option_group_id`, `option_id` |
| `duration_selected` | `journey_id`, `service_id`, `duration_hours` |
| `date_selected` / `time_selected` | `journey_id`, `service_id`, `slot_date` / `slot_time` |
| `booking_details_completed`, `booking_summary_viewed`, `checkout_started` | `journey_id`, `service_id`, `value`, `currency` |
| `slot_selected` | `journey_id`, `service_id`, `slot_start` (ISO-8601 with timezone offset), `value`, `currency` |
| `promo_applied` | `journey_id`, `service_id`, `promo_code`, `promo_result` |
| `payment_started` | `journey_id`, `service_id`, `payment_method`, `value`, `currency` |
| `topup_amount_selected` | `amount`, `source` |
| `topup_payment_started` | `amount`, `payment_method`, `value`, `currency` |
| `registration_started` | `reg_flow_id`, `entry` |
| `otp_viewed` | `reg_flow_id`, `flow_type`, `otp_mode`, `attempt_round` |
| `otp_code_entered` | `reg_flow_id`, `flow_type`, `otp_mode` |
| `otp_submitted` | `reg_flow_id`, `attempt_no`, `otp_mode` |
| `otp_failed` | `reg_flow_id`, `error_code`, `otp_mode` |
| `otp_verified` | `reg_flow_id`, `otp_mode` |
| `otp_resend_clicked` | `reg_flow_id` |
| `signup_viewed` | `reg_flow_id`, `attempt_round` |
| `signup_submitted` | `reg_flow_id`, `result` |
| `account_created`, `registration_completed` | `reg_flow_id`, `method`, `entry` |
| `service_area_checked` | `source`, `result` |
| `address_added`, `address_edited` | `source`, `result`, `method` |
| `address_deleted` | `source`, `result` |
| `location_pin_moved` | `method` |
| `screen_viewed` / `screen_exited` | `screen_name` + `entry_type` / `duration_seconds` |
| `overlay_viewed` | `overlay_name`, `overlay_type` |
| `element_clicked` | `screen_name`, `element_id`, `element_type` |
| `form_submitted` | `form_id`, `result` |
| `search_performed` / `search_result_clicked` | `result_count` / `rank` |
| `setting_changed` | `setting_id`, `setting_value` |
| `permission_result` | `permission`, `result` |
| `support_contacted` | `channel`, `source_screen` |
| `chat_opened` / `chat_message_sent` | `chat_kind`, `entry` / `message_type` |
| `app_error` | `error_type`, `screen_name`, `fatal` |
| Cancel and reschedule events, Instant events | the parameters listed in 9.5 |

Ids such as `screen_name`, `element_id`, `overlay_name`, `form_id` and `previous_screen` must be `snake_case`.

### 9.5 Custom events and their parameters

**App, campaign and notifications**

| Event | Fires when | Parameters |
|---|---|---|
| `app_opened` | App launch with attribution | `entry_type` (`organic`, `deeplink`, `push`), `campaign_code`, `utm_*` |
| `campaign_attributed` | A campaign link is resolved | `utm_campaign`, `utm_source`, `utm_medium`, `utm_content`, `utm_id` |
| `app_backgrounded` | App goes to background | `session_seconds` |
| `app_foregrounded` | App returns | `background_seconds` |
| `homepage_viewed` | Home shown | `entry_type` (`organic`, `deeplink`, `push`, `banner`), `campaign_code`, `utm_campaign`, `utm_source`, `utm_medium` |
| `service_list_viewed` | Service list shown | `campaign_code`, `utm_campaign`, `utm_source`, `utm_medium` |
| `banner_viewed` | Banner on screen | `banner_id`, `placement` (`home`, `coupon_list`, `other`), `campaign_id` |
| `banner_clicked` | Banner or pop-up button tapped | `banner_id`, `placement`, `campaign_id`, `action`, `target`, `category_id` |
| `push_received` | Push arrives while app is open | `notification_id`, `campaign_id`, `segment_id` |
| `push_opened` | Push tapped | `campaign_id`, `segment_id`, `destination`, `push_type`, `target_id`, `notification_id`, `send_id`, `action_id` |
| `notification_inbox_viewed` | Inbox tab opened | `unread_count` |
| `notification_detail_viewed` | Announcement opened | `announcement_id`, `notification_id`, `source` (`push`, `inbox`, `popup`) |

**Screens, taps and general**

| Event | Fires when | Parameters |
|---|---|---|
| `screen_viewed` | A screen opens | `screen_name`, `screen_group`, `entry_type` (`nav`, `tab`, `push`, `deeplink`, `back`), `previous_screen` |
| `screen_exited` | A visit of 1 second or more ends | `screen_name`, `duration_seconds` (max 1800) |
| `overlay_viewed` | Sheet, dialog, pop-up or toast appears | `overlay_name`, `overlay_type` (`sheet`, `dialog`, `popup`, `toast`), `trigger` (`user`, `auto`, `push`), `screen_name` |
| `element_clicked` | A tap | `screen_name`, `element_id`, `element_type`, `section`, `position` (1-based), `item_id` (entity id only), `target` |
| `form_submitted` | A form is sent | `form_id` (`profile_info`, `customer_source`, `review`), `screen_name`, `result` (`success`, `error`, `invalid`), `error_code`, `field_count`, `tags_count`, `has_comment` |
| `search_performed` | Search made (after 600 ms of no typing) | `search_scope` (`service`, `place`), `result_count`, `screen_name`, `query` (service search only) |
| `search_result_clicked` | A search result tapped | `search_scope`, `rank`, `query`, `service_id` |
| `setting_changed` | Language changed | `setting_id` (`language`), `setting_value`, `source` (`home_sheet`, `profile`, `default_fallback`, `other`), `previous_value` |
| `permission_result` | Permission answer | `permission` (`push`, `location`, `photos`, `camera`), `result` (`granted`, `denied`, `denied_forever`, `limited`, `services_disabled`), `context` (`pre_prompt`, `system`, `use_current_location`, `auto_open`) |
| `support_contacted` | A support channel is used | `channel` (`telegram`, `phone`, `chat`, `email`), `source_screen`, `element_id`, `order_id` |
| `chat_opened` | Chat room opened | `chat_kind` (`order`, `support`), `entry` (`chat_list`, `order_detail`, `other`), `screen_name`, `order_id` |
| `chat_message_sent` | Message sent | `message_type` (`text`, `image`, `video`, `story`), `screen_name` |
| `app_error` | A failure the customer can see | `error_type` (`api`, `payment`, `webview`, `network`, `update`, `exception`), `screen_name`, `fatal` (0/1), `endpoint_group`, `http_status`, `error_code` |

**Registration** (all carry `reg_flow_id`)

| Event | Fires when | Parameters |
|---|---|---|
| `registration_started` | Customer enters sign-in or sign-up | `entry` (`onboarding`, `home`, `tab_gate`, `checkout_address`, `checkout_schedule`, `checkout_summary`, `banner`, `push`, `deeplink`, `dialog`, `other`), `entry_detail`, `source_screen`, `campaign_code` |
| `otp_viewed` | OTP screen shown | `flow_type` (`register`, `login`), `otp_mode` (`deferred`, `immediate`), `attempt_round` |
| `otp_code_entered` | Last digit typed (deferred mode) | `flow_type`, `otp_mode` |
| `otp_submitted` | Code submitted | `attempt_no`, `otp_mode` |
| `otp_failed` | Code rejected | `error_code`, `otp_mode` |
| `otp_resend_clicked` | Resend tapped | none |
| `otp_verified` | Code accepted | `otp_mode` |
| `signup_viewed` | Sign-up form shown | `attempt_round` |
| `signup_submitted` | Create account returned | `result`, `error_code`, `has_email`, `has_referral` |
| `account_created` | Account exists on server | `method`, `entry` |
| `registration_completed` | Registration finished | `method`, `entry` |

**Location and address**

| Event | Fires when | Parameters |
|---|---|---|
| `service_area_checked` | Coverage check returned | `source` (`profile`, `checkout`), `result` (`ok`, `error`), `in_range`, `area_cell`, `zone_id`, `address_id`, `duration_ms` |
| `address_added`, `address_edited` | Address saved | `source`, `result` (`success`, `error`), `method` (`current_location`, `search`, `map_pin`), `has_floor`, `has_room`, `has_note`, `area_cell`, `error_code` |
| `address_deleted` | Address deleted | `source`, `result` |
| `location_pin_moved` | Map pin moved | `method` (`drag`, `tap`, `current_location`, `search`), `area_cell` |

**Service, booking and checkout** (all carry `service_id` and `journey_id`)

| Event | Fires when | Parameters |
|---|---|---|
| `service_viewed` | Service page opened | `entry_point` (`home`, `search`, `banner`, `push`, `deeplink`) |
| `booking_started` | Booking flow entered | none |
| `address_selected` | Address chosen | `source` (`checkout`), `method` (`default_auto`, `saved_list`, `new_address`, `existing`), `zone_id`, `area_cell` |
| `service_option_selected` | Option chosen | `option_group_id`, `option_id`, `selection_method` (`user`, `auto_default`, `auto_rooms`, `restored`) |
| `duration_selected` | Duration chosen | `duration_hours`, `selection_method` |
| `date_selected` | Date chosen | `slot_date` (`YYYY-MM-DD`) |
| `time_selected` | Time chosen | `slot_time` (`HH:mm`) |
| `slot_selected` | Date and time both chosen | `slot_start` (ISO-8601 with offset), `value`, `currency` |
| `booking_details_completed` | Details complete | `value`, `currency` |
| `booking_summary_viewed` | Checkout summary shown | `value`, `currency` |
| `checkout_started` | Payment step opens | `value`, `currency`, `instant_hours`, `promo_code`, `promo_source`, `campaign_id` |
| `payment_started` | Payment method chosen | `payment_method`, `value`, `currency`, `instant_hours` |
| `order_created` | Backend created the order | `order_id`, `order_source` (`app`), `value`, `currency`, `payment_method` |
| `order_confirmation_viewed` | Confirmation shown | `order_id` |
| `payment_failed` | Payment failed | `payment_method`, `error_code` |
| `payment_returned_success` | Back from bank app and paid | `transaction_ref`, `payment_method`, `value`, `currency` (diagnostic, not revenue) |
| `coupon_list_viewed` | Coupon list opened | `coupon_count` |
| `promo_code_entered` | Code typed and submitted | `promo_code` |
| `promo_applied` | Code accepted | `promo_code`, `promo_result` (`valid`), `promo_source` (`manual`), `source` (`manual`, `list`, `deeplink`, `notification`), `final_value` |
| `promo_rejected` | Code refused | `promo_code`, `rejection_reason` |
| `addon_group_viewed` | Add-on group shown | `addon_group_id`, `is_required`, `item_count` |
| `addon_selected` | Add-on chosen | `addon_group_id`, `addon_id`, `quantity`, `value`, `currency` |
| `addon_quantity_changed` | Quantity changed | `addon_group_id`, `addon_id`, `quantity` |
| `addon_removed` | Add-on removed | `addon_group_id`, `addon_id` |
| `task_info_viewed`, `equipment_viewed` | Info sheet opened | none |
| `pair_service_viewed` | "Often paired with" shown | `item_count` |
| `pair_service_selected` | Pair card tapped | `pair_service_id`, `position` |
| `pair_service_added` | Pair option added | `pair_service_id`, `option_id`, `quantity`, `value`, `currency` |
| `pair_service_removed` | Pair option removed | `option_id` |

**Instant Cleaning** (carry `service_id`, `booking_mode=instant`)

| Event | Fires when | Parameters |
|---|---|---|
| `instant_availability_checked` | Availability loaded | `available_dates`, `total_dates`, `fully_booked`, `trigger` (`load`, `refresh`, `return`) |
| `instant_fully_booked_viewed` | Fully booked shown | `total_dates` |
| `instant_duration_selected` | Hours chosen | `hours`, `selection_method`, `was_suggested` |
| `instant_room_toggled` | Room ticked or unticked | `room_key`, `selected`, `selected_count`, `total_minutes`, `band` (`none`, `one_hour`, `two_hours`, `bigger_job`), `suggested_hours` |
| `instant_underpick_warning_viewed` | "Too few rooms" warning | `selected_hours`, `suggested_hours`, `total_minutes` |
| `instant_upsell_clicked` | Deep Cleaning link tapped | `source` (`text_link`, `rooms_section`), `target_service_id`, `rooms_selected` |
| `instant_upsell_result` | Back from Deep Cleaning | `target_service_id`, `returned` |
| `instant_slot_selected` | Time chosen | `slot_start`, `same_day`, `lead_minutes`, `days_ahead`, `slot_source` (`selector`, `schedule_sheet`) |
| `schedule_grace_result` | Grace-period check returned | `result` (`ok`, `error`), `error_code` (`INSTANT_NOTICE_TOO_SHORT`, `BOOKING_SLOT_UNAVAILABLE`, `BOOKING_LEAD_TIME_REQUIRED`, `BOOKING_GRACE_INVALID`, `BOOKING_POLICY_INVALID`, `INSUFFICIENT_BALANCE`, `other`), `lead_minutes`, `same_day`, `action` |
| `category_alert_viewed` / `category_alert_action` | Selection alert shown / answered | `alert_key`, `badge` / `action` (`close`, `cta`) |
| `instant_progress_viewed` | Job progress stage shown | `stage` (`waiting`, `in_progress`, `wrapping_up`, `completed`), `order_id` |

**Top-up**

| Event | Fires when | Parameters |
|---|---|---|
| `topup_started` | Top-up screen opens | none |
| `topup_amount_selected` | Amount chosen | `amount`, `currency`, `source` (`preset`, `typed`), `position` |
| `topup_payment_started` | Payment method chosen | `amount`, `payment_method`, `value`, `currency` |

**Orders, reschedule, cancel, extend, review**

| Event | Fires when | Parameters |
|---|---|---|
| `order_detail_screen` | Order detail opened | `order_detail_screen`=1, `service_name`, `order_id`, `service_id` |
| `reschedule_started` | Reschedule sheet opens | `order_id`, `current_slot`, `reschedule_source` (`customer`, `admin`), `entry` (`order_detail`, `cancel_sheet`), plus order context |
| `reschedule_slot_selected` | New time picked | `order_id`, `old_slot`, `new_slot`, `shift_hours` |
| `reschedule_grace_requested` | Grace check returned | `order_id`, `result` (`ok`, `slot_unavailable`, `error`), `error_code`, `new_slot` |
| `reschedule_submitted` | Confirm tapped | `order_id`, `old_slot`, `new_slot`, `payment_method` |
| `reschedule_completed` | Reschedule succeeded | `order_id`, `old_slot`, `new_slot`, `reschedule_source` |
| `reschedule_failed` | Reschedule failed | `order_id`, `failure_reason`, `http_status` |
| `reschedule_abandoned` | Sheet closed without success | `order_id`, `step` (`pick`, `confirm`, `choose_pay`, `payment_failed`) |
| `order_rescheduled` | Order moved | `order_id` |
| `cancellation_started` | Cancel sheet opens | `order_id`, `entry` (`order_detail`, `reschedule_sheet`), plus order context |
| `cancellation_reason_selected` | Reason chosen | `order_id`, `reason_code` (`plan_changed`, `booked_by_mistake`, `want_reschedule`, `no_longer_needed`, `other`) |
| `cancel_to_reschedule_clicked` | "Reschedule instead" tapped | `order_id`, `reason_code`=`want_reschedule` |
| `cancellation_policy_viewed` | Policy opened | `order_id`, `source_screen` (`cancel_sheet`, `reschedule_sheet`, `order_detail`, `checkout`) |
| `cancellation_submitted` | Confirm tapped | `order_id`, `reason_code`, `has_other_text`, `hours_to_start` |
| `cancellation_result` | Cancel call returned | `order_id`, `result` (`success`, `failed`), `error_code`, `http_status`, `duration_ms` |
| `cancellation_abandoned` | Sheet closed without cancelling | `order_id`, `step` (`reason`, `confirm`), `reason_code` |
| `extend_started`, `extend_payment_selected`, `extend_closed` | Extend hours steps | `stage`, `extra_hours`, `payment_method` |
| `review_submitted` | Review sent | `order_id`, `stars` |

Order context (cancel and reschedule start events): `hours_to_start` (rounded to 0.5), `order_status`, `value`, `currency`, `partner_assigned` (0/1). The cancel events are built but record nothing while the cancel option is switched off.

### 9.6 Firebase standard and ecommerce events

| Event | Fires when | Parameters |
|---|---|---|
| `purchase` | Payment confirmed (the only event with revenue) | `value`, `currency`, `transaction_id`, `coupon`, `service_name`, `order_id`, `booking_source` (`app`), `booking_status`, `payment_status`, `booking_id`, `utm_*`, common parameters |
| `sign_up` | Account created | `sign_up_method` |
| `app_open` | App launch | none |
| campaign details | Campaign link resolved | `source`, `medium`, `campaign`; also user properties `utm_campaign`, `utm_source` |
| `view_content` | Service page opened | `service_name`, `service_id` |
| `select_service` | Service selected | `service_name`, `service_id` |
| `begin_checkout` | Checkout summary opened | `service_name`, `value`, `currency`, `service_id` |
| `add_payment_info` | Payment method chosen | `service_name`, `value`, `currency`, `service_id` |

### 9.7 Meta and AppsFlyer

| Meta event | AppsFlyer event | Fires when | Parameters |
|---|---|---|---|
| ViewContent | `af_content_view` | Service page opened | service name, content type `service`; AppsFlyer: `af_content_id`, `af_content`, `af_content_type` |
| SelectService | none | Service selected | service name |
| Searched | `af_search` | Service search | Meta: content type `service`; AppsFlyer: `af_search_string`, `af_content_type` |
| AddedToCart | `af_add_to_cart` | Details complete (address and time ready) | `value`, `currency`; AppsFlyer: `af_price`, `af_currency`, `af_content_id`, `af_content_type` |
| Schedule | `af_schedule` | Date and time chosen | `value`, `currency` (AppsFlyer: `af_price`) |
| InitiatedCheckout | `af_initiated_checkout` | Checkout opened | service, `value`, `currency` |
| AddedPaymentInfo | `af_add_payment_info` | Payment method chosen | service, `value`, `currency` |
| ViewBooking | none | Bookings list opened | `order_id` when known |
| CompletedRegistration | `af_complete_registration` | Account created | registration method (`af_registration_method`) |
| none | `af_login` | Returning customer logs in | none |
| Contact | `af_contact` | Support channel tapped (not a push tap) | `channel`, content type `channel` |
| Purchase | `af_purchase` | Payment confirmed | Meta: `value`, `currency`, `order_id`, service, attribution; AppsFlyer: `af_revenue`, `af_currency`, `af_order_id`, `af_coupon_code` |
| App activate | install attribution | First open | handled by the SDKs |

Funnel events carry a price but never revenue; only `af_purchase` and Meta Purchase carry revenue.

### 9.8 Screen and element names

**Screens (`screen_name` → `screen_group`).** launch: `splash`, `splash_ad`. system: `no_internet`, `server_error`, `force_update`, `webview`. auth: `welcome`, `sign_in`, `sign_up`, `otp`, `set_password`, `customer_source`, `notification_prompt`. home: `home`, `home_search`, `see_more`, `banner_detail`. booking: `service_detail`, `service_addons`. checkout: `checkout`, `coupon_list`, `schedule_sheet`, `payment_sheet`, `payment_result`, `appointment_status`. orders: `bookings`, `order_detail`, `order_cancel`, `order_reschedule`, `order_extend`, `leave_review`, `review_done`. notifications: `notifications`, `notification_detail`. chat: `chat_list`, `chat_room`, `chat_story_create`. profile: `profile`, `profile_info`, `change_password`, `change_avatar`, `locations`, `location_edit`, `location_search`, `language`, `payment_methods`, `my_coupons`, `help_center`, `tutorial_center`, `tutorial_detail`, `about`. topup: `topup`, `topup_history`, `topup_status`. A screen with no assigned name is sent as its own name in `snake_case`, group `other`.

**Bottom tabs** are screens `home`, `bookings`, `notifications`, `chat_list`, `profile` with `entry_type=tab`.

**`element_type`** is one of `button`, `link`, `icon`, `tab`, `card`, `banner`, `chip`, `menu_row`, `toggle`, `input`, `sheet_action`, `back`.

**`element_id` by area**

| Area | element_id |
|---|---|
| Launch, system | `splash_ad_cta`, `splash_ad_skip`, `update_now`, `update_later`, `retry_connection`, `retry_server` |
| Tabs | `tab_home`, `tab_bookings`, `tab_notifications`, `tab_chat`, `tab_profile`, `tab_locked_guest` |
| Home | `language_picker`, `promo_fab`, `promo_popup`, `promo_close`, `bcombo_card`, `sign_in_card`, `refresh_home`, `service_card`, `see_all_services`, `retry_services`, `banner_card`, `latest_promo_card`, `turn_on_notifications`, `join_telegram`, `telegram_float_button`, `close_telegram_float`, `contact_support`, `copy_promo_code` |
| Banner detail | `banner_cta`, `banner_image_zoom`, `banner_coupon_select`, `banner_terms_check`, `banner_payment_method`, `banner_sheet_close`, `receipt_upload`, `receipt_submit`, `receipt_take_photo`, `receipt_choose_album`, `receipt_cancel`, `guest_sign_in_continue` |
| Notifications | `notification_card`, `announcement_card`, `mark_all_read`, `mark_all_read_confirm`, `mark_all_read_cancel`, `tab_notifications_list`, `tab_announcements`, `refresh_notifications`, `refresh_announcements`, `notif_banner_not_now`, `notif_banner_open_settings`, `announcement_close`, `announcement_cta` |
| Service detail, checkout | `back`, `contact_support`, `retry`, `schedule_appointment`, `add_to_booking`, `showcase_task_info`, `equipment_included`, `addon_done`, `sheet_close`, `sub_category_selected`, `show_all_options`, `option_info`, `start_booking`, `refresh_price`, `change_schedule`, `read_full_policy`, `apply_coupon`, `use_other_promo`, `confirm_pay`, `sign_up_gate`, `cross_sell_next`, `select_payment_method`, `pair_sheet_cancel`, `pair_sheet_confirm`, `schedule_confirm`, `schedule_close`, `coupon_card`, `search_coupon`, `back_home`, `try_again`, `result_close` |
| Top-up | `topup_amount`, `topup_confirm`, `topup_history_item`, `topup_status_done`, `aba_transfer_submit`, `aba_transfer_close`, `topup_from_balance`, `topup_new` |
| Orders | `tab_active`, `tab_history`, `order_card`, `explore_services`, `refresh_bookings`, `contact_us`, `pay_now`, `book_again`, `leave_review`, `swipe_complete`, `cleaners_not_yet`, `cleaners_done`, `service_add_on`, `refresh_order`, `gift_apply`, `gift_cancel`, `extend_hours`, `extend_close`, `extend_confirm`, `extend_done`, `addon_service_item`, `addon_add`, `addon_increase`, `addon_decrease`, `addon_checkout`, `addon_close`, `review_star`, `review_tag`, `review_submit`, `review_done_close_icon`, `review_done_close` |
| Profile | `menu_profile_info`, `menu_my_location`, `menu_my_coupons`, `menu_language`, `menu_share_app`, `menu_help_center`, `menu_contact_support`, `menu_about`, `menu_logout`, `logout_confirm`, `language_option`, `help_faq`, `help_terms`, `help_privacy`, `help_email`, `contact_telegram`, `contact_phone`, `contact_email`, `change_avatar`, `avatar_take_photo`, `avatar_choose_album`, `avatar_save_image`, `gender_field`, `gender_option`, `dob_field`, `dob_done`, `dob_cancel`, `save_profile`, `delete_account`, `delete_account_confirm`, `refresh_coupons`, `about_website`, `about_facebook`, `about_instagram`, `about_tiktok`, `about_telegram`, `about_rednote` |
| Tutorials | `tutorial_category`, `tutorial_card`, `tutorial_play`, `tutorial_toggle_play`, `tutorial_fullscreen`, `tutorial_related`, `tutorial_helpful_yes`, `tutorial_helpful_no` |
| Sign-up helpers | `skip_customer_source`, `source_option`, `customer_source_continue`, `contact_support` |
| Address | `use_current_location`, `add_address`, `address_card`, `edit_address`, `delete_address`, `save_address`, `search_bar`, `back` |
| Chat | `chat_tile`, `refresh_chats`, `place_order`, `chat_back`, `send_message`, `attach_camera`, `story_shutter`, `story_switch_camera`, `story_gallery`, `story_close`, `media_retake`, `media_send`, `media_back`, `media_open`, `gallery_pick` |

**`overlay_name`:** `language_picker`, `promo_popup`, `lucky_draw`, `coming_soon`, `guest_sign_in_prompt`, `banner_payment_methods`, `banner_aba_transfer`, `receipt_source`, `mark_all_read`, `announcement_detail`, `force_update`, `optional_update`, `schedule_sheet`, `payment_sheet`, `payment_result_success`, `payment_result_failed`, `cross_sell_sheet`, `pair_service_sheet`, `task_info`, `equipment_included`, `service_addons`, `unavailable_schedule`, `topup_payment_sheet`, `topup_aba_transfer`, `topup_detail`, `topup_low_balance`, `sub_category_detail`, `order_payment_sheet`, `cleaners_not_finished`, `addon_services`, `order_addon`, `gift_coupon_confirm`, `extend_hours`, `contact_support`, `logout_confirm`, `avatar_source`, `gender_picker`, `dob_picker`, `delete_account_confirm`, `story_gallery`.

**`endpoint_group` (in `app_error`):** `home_page`, `user_info`, `no_internet`, `places`, `address`, `coverage`, `order_preview`, `coupon_apply`, `order_create`, `order_schedule`, `payment_methods`, `service_options`, `payment_status`, `payment_webview`, `topup_create`, `topup_amounts`, `topup_webview`, `topup_status`, `booking_list`, `order_detail`, `order_complete`, `review_options`, `app_version`.

### 9.9 Older events that still run

These existed before the new tracking and still fire next to the new ones, so current reports keep working. Each carries its own name as a parameter set to `1` unless another parameter is shown. They are retired after marketing confirms nothing depends on them (section 7).

| Event | Extra parameters |
|---|---|
| `splash_ad_screen`, `splash_skip_button` | none |
| `splash_cta_button` | `action`, `target` |
| `sign_up_screen`, `sign_in_screen`, `sign_in_continue`, `guest_mode_button`, `create_account_button`, `register_button_on_home` | none |
| `customer_source_screen_tap`, `notification_prompt_continue`, `notification_prompt_not_now`, `homepage_notification_screen_view` | none |
| `customer_source_continue` | one parameter named after the answer chosen, set to `1` |
| `home_screen`, `see_more_screen`, `banner_detail_screen`, `booking_screen`, `profile_screen`, `profile_info_screen`, `location_screen`, `topup_screen`, `top_up_history`, `top_up_status` | none |
| `service_click` | `service_name` |
| `bcombos_click`, `language_selection`, `click_promote`, `fully_book_button`, `cancel_button` | none |
| `tracking_pages` | `page_name` |
| `contact_us_telegram` | none |
| `join_telegram_channel`, `turn_on_notifications_click`, `notification_banner_not_now` | `source` |
| `increase_quantity`, `decrease_quantity` | the quantity under the same name |
| `one_item_at_a_time`, `tasK_information` | none |
| `instant_cleaning_explore_other_options` | `source` |
| `rating_selected` | `rating_start` (1 to 5) |
| `leave_review` | none |
| `leave_review_submitted` | `rating`, `feedback_categories`, `feedback_count` |
| `pay_now_order_details` | none |
| `reschedule_button_click` | `order_detail_screen` |
| `reschedule_button_confirm` | `order_detail_screen`, `payment_method` |
| `cancel_booking_confirm` | `order_detail_screen`, `reason` |
| `extend_hour_pay` | `order_detail_screen`, `payment_method` |
