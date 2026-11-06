# bEasy App — What we plan to track

Updated: 2026-11-05 · Audience: management, marketing, business and data analysis · Status: tracking is built in the app and is going through testing before release.

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
| **Search** | What do customers search for? Which searches find nothing (unmet demand)? *Ready, but the search screen is not currently reachable, see section 5.* |
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

## 3. Principles

1. **No personal data.** We never record phone numbers, names, address text, chat messages, card details or coupon text. Locations are rounded to about 1 km. Search text is recorded only for the service search, shortened to 30 characters.
2. **One consistent system.** Every screen and button has a fixed name, so reports stay comparable across screens. Khmer and English labels are managed in the portal.
3. **Existing reports keep working.** Old events continue for one release alongside the new ones, so current Firebase and Meta audiences do not break. We retire the old ones only after marketing confirms nothing depends on them.
4. **Staff and test phones are excluded** from the new tracking, so internal test orders do not distort the numbers.
5. **Money and order status come from the backend.** Customer-side events for payment or top-up are only for diagnosis, not for revenue figures.

## 4. Advertising platforms (Meta and AppsFlyer)

| Platform | Already receives | Will also receive |
|---|---|---|
| **Meta** | View service, select service, start checkout, add payment info, view booking, complete registration, purchase | Search, add to cart, schedule, contact |
| **AppsFlyer** | Installs and campaign attribution, purchase | View service, search, add to cart, start checkout, add payment info, schedule, contact, complete registration, login |

For marketing:

- **"Add to cart" means the address and time are ready.** The app has no real cart.
- **Start checkout comes before add to cart** in the data. Keep that order in mind when building funnels.
- **Revenue is sent only with the purchase event.** Funnel events carry a price but no revenue, so revenue is not counted twice.
- **Top-up, errors and settings are not sent** to ad platforms, because they add no ad-optimisation value.

## 5. Not tracked: features that are switched off or unreachable

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

## 6. Open decisions and remaining work

| Item | Detail | Recommendation |
|---|---|---|
| **Crash monitoring** | The app has no crash reporting today. | Add Firebase Crashlytics. |
| **Retire old events** | Old and new events run side by side for one release. | Retire the old ones after marketing confirms no audience depends on them. |
| **Staff exclusion for existing ad events** | The original Meta events and the AppsFlyer purchase still include staff phones. | Exclude staff phones. Needs owner approval. |
| **AppsFlyer login event** | It fires on every login by an existing customer. | Decide whether to keep it. |
| **Chat connection errors and media upload results** | Not tracked yet. | Plan as a separate piece of work. |
| **Guest returns to checkout after signing up** | Not tracked yet. | Add as a follow-up. |
| **Backend** | The backend must store the new events and provide the data for the portal's App Usage pages. | Request has been prepared for the backend team. |
| **Testing** | Compare the new app against the current one before release. | Required before release. |

## 7. Technical detail

For the full event list, rules and backend design, see the tracking specifications in this folder: App Tracking Coverage Spec, Registration and Location Flow Spec, Cancellation and Reschedule Spec, Instant Cleaning Tracking Spec, and the App Tracking Backend Handoff.
