# App tracking: A/B regression plan (main vs feat/tracking-integration)

Updated: 3 Nov 2026 · Owner: SNLRLONG · Fits task T-0073 (`Tasks/2-todo/Cross-tracking-qa-signoff.md`) and the G4 gate of every App tracking task.

Event lists are not copied here. Each slice's handler map lives in its task file (links in section 7). Screen coverage: [App_Interaction_Inventory.md](App_Interaction_Inventory.md). Specs: [App_Tracking_Coverage_Spec.md](App_Tracking_Coverage_Spec.md), [Registration_And_Location_Flow_Spec.md](Registration_And_Location_Flow_Spec.md), [Cancellation_And_Reschedule_Spec.md](Cancellation_And_Reschedule_Spec.md), [Instant_Cleaning_Tracking_Spec.md](Instant_Cleaning_Tracking_Spec.md).

## 1. Goal and pass rule

The tracking slices are **collection only**. Build B must behave exactly like build A.

- **A** = `main` (flavor dev). **B** = `feat/tracking-integration` (flavor dev).
- Same scripted cases, same account, same dev backend, same device.
- **Pass rule:** for every case, B equals A in: screens shown, navigation (what opens, back stack, what Back does), request sequence (method + path + status, order), request bodies and amounts, error handling (dialogs, toasts, retry), loading states (spinners, disabled buttons).
- **Any difference is a bug in B**, unless it is a request to Firebase, Meta, AppsFlyer or Clarity (B adds events). Log it, fix on the slice branch, re-run the case on A and B.
- Changed surface: about 99 files, `lib/` plus tests (`git -C bEasy-app-tracking-integration diff --stat main..HEAD`). About 270 lines of `lib/` were removed or rewritten; those are where behaviour can change (wrapped callbacks, route names, auth path).

## 2. Setup

### 2.1 Two builds side by side

| | A (baseline) | B (candidate) |
|---|---|---|
| Folder | `/Users/suntel/work/bEasy/bEasy-app` (branch `main`) | `/Users/suntel/work/bEasy/bEasy-app-tracking-integration` (`feat/tracking-integration`) |
| Run | `fvm flutter run --flavor dev --target lib/main_dev.dart` | same command |

Before the first run: `git -C <folder> status` must be clean on both (no WIP from other tasks), then `fvm flutter pub get` in each.

**Important (verified in `android/app/build.gradle.kts`):** both builds use the same Android id `suntel.beasy.app.dev` (`applicationIdSuffix ".dev"`). Installing B over A replaces A, and app data is shared. Choose one:
1. One device, run A then B in turn; before each run do `adb uninstall suntel.beasy.app.dev` (or clear data) so both start from the same state. Simple, slower.
2. Two devices (or one Android phone and one emulator), A on one, B on the other, cases run in parallel. Preferred for comparing screens.

Build mode: use **debug (default `flutter run`)** for A/B behaviour. Tracking events are not sent in debug (see section 6), but all the tracking code still runs, so it is the right mode to find behaviour changes. Run one final pass in **release** on B only (section 6) to confirm events and that release does not behave differently.

### 2.2 Accounts and data

| Account | Use |
|---|---|
| Guest | fresh install, no sign-in |
| New number | a phone number never registered on dev (use the dev OTP rule agreed with backend; write it in the log) |
| Existing customer with orders | has past, upcoming and Instant orders, at least one coupon, a saved address, wallet balance |
| Staff phone | **excluded from analytics** (`PhoneNumbers.instance.isStaff`, `lib/analytics/analytics_helper.dart:22`). Fine for A/B behaviour cases, but it will show no events in DebugView. Do not use it for section 6 |

Reset between runs of the same case: clear data on the device, reset test order state (cancel/reschedule uses a fresh order each time).

### 2.3 Capturing requests

No Dio log interceptor was found in `lib/data/api/api_client.dart` (not fully verified for every client; the chat module has its own client). Use a proxy, not code changes:

1. Run Charles or mitmproxy on the same Mac; set the device Wi-Fi proxy to it; install and trust its certificate on the device (Android 7+ apps do not trust user certificates by default; if the dev flavor does not trust it, see "Not verified" below, or use an emulator with the system CA installed).
2. Filter to the dev API host (`api-*.beasy.info`). Export the session per case as HAR, or copy the ordered list.
3. **Compare** the ordered list of `method + path + status`, plus key body fields: ids, `amount`/`total`/`discount`, coupon code, schedule date/slot, address id, payment method, `hours`, language, order id. Ignore timestamps, tokens, device ids.
4. **Ignore** hosts: Firebase (`firebaselogging`, `app-measurement`, `firebaseinstallations`, `firebaseremoteconfig`), Meta (`graph.facebook.com`), AppsFlyer, Clarity. B adds events there. B must not add or drop requests to the dev API host. Same applies to MQTT/WebSocket chat frames: compare connect and subscribe only.
5. Diff tip: save each case as `case-ID-A.txt` / `case-ID-B.txt` (one request per line) and `diff` them.

### 2.4 Device matrix

| Device | Builds | Priority |
|---|---|---|
| One Android phone (Android 12 or later preferred; also covers the "Android cannot tell never-asked from denied" permission note) | A and B | P0, run everything |
| One iOS device or simulator | A and B | P0 for sections C, G, L, M (permissions, webview, camera, background); P1 for the rest |

Rotation is out of scope. Language: run each case in Khmer unless the case names another; the language cases (profile) cover en and zh.

### 2.5 Conventions in the tables

- Priority: **P0** money, sign-in, navigation core; **P1** everything else on a main path; **P2** edge or rarely used.
- "Same" in Expected means: identical on A and B and equal to what a user sees on A today.
- Result columns hold OK / DIFF / N/A plus a one-line note; Verdict is PASS only when both columns are OK and there is no unexplained request diff.
- Risk column: what the tracking change touched, so you know what to look at. Names in brackets are task IDs.

## 3. Case tables

### A. Launch and splash (T-0064, T-0063)

| ID | Pri | Steps (screens) | Expected (same on A and B) | Tracking risk to watch | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| A1 | P0 | Cold start, guest, normal network (Splash, Splash ad if shown, Home) | Same screens, same order, same timing; ad skip/close works | `splash_screen`, `splash_ad_screen` edits; first-screen observer must not delay navigation | | | |
| A2 | P1 | Tap the splash ad / its close (Splash ad, Banner detail) | Same destination; Back returns where A returns | ad tap now wrapped in a closure | | | |
| A3 | P0 | Start with an optional update available (force a lower build or mock version) (Update sheet) | Update sheet shows; "Update" opens the store, "Later" dismisses; Home still usable | `custom_button_sheet_update_app.dart` changed; prompt must stay dismissible | | | |
| A4 | P1 | Forced update variant, if the backend can return it | Not dismissible on both builds | same file | | | |
| A5 | P0 | Airplane mode, cold start (No internet screen); retry with network on | Same screen; Retry reloads and continues to Home | `no_internet_screen.dart`, overlay in `my_app.dart` | | | |
| A6 | P1 | Lose network while on Home, pull to refresh | Same error/empty state; recovers on retry | `app_error home_page` wrappers must not swallow the original error | | | |
| A7 | P1 | Backend 5xx at launch (proxy map-local or block the host) (Server error screen) | Same server-error screen; Retry works | `handle_server_error_screen.dart` edited | | | |
| A8 | P1 | Customer profile load fails after sign-in (block profile call) | Same behaviour as A (error handling in `user_info` loads) | `app_error user_info` added around the call | | | |
| A9 | P2 | Holiday alert on Home when a holiday is configured (Holiday alert) | Same alert, same dismiss | `custom_alert_holiday.dart` edited | | | |

### B. Guest home and tab gates (T-0064, T-0074)

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| B1 | P0 | Guest: scroll Home top to bottom (Home) | Same sections, same order, same images | observer / section views must not rebuild or flicker | | | |
| B2 | P0 | Guest: tap Bookings tab, then Profile tab (App shell, Sign in) | Each opens the sign-in screen (locked tab); Back returns to the previous tab on both | `tab_locked_guest` fires before the sign-in push; tab index after Back | | | |
| B3 | P1 | Guest: tap Home and Services tabs repeatedly, then switch quickly | Same tab shown, no extra route pushed | tab observer on `AppScreen` | | | |
| B4 | P0 | Guest: tap the sign-in card on Home (Home, Sign in) | Sign-in opens; Back returns to Home | `entry:` param added to `NewSignINScreen` | | | |
| B5 | P1 | Guest: tap a service (Service detail), tap Book (guest gate) | Same gate (dialog or sign-in) as A | `guest_sign_in_prompt` overlay and Continue wrapper | | | |
| B6 | P1 | Guest: open Notifications tab / inbox if reachable | Same (locked or empty) | | | | |

### C. Registration and login (T-0074)

Screens: Sign in (`new_signin_screen`), OTP (`new_verification_code_screen`), Sign up (`new_signup_screen`), Customer source (`customer_source_screen`), Home.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| C1 | P0 | New number: enter phone, Continue (Sign in to OTP) | OTP screen opens (code first); one send-OTP request | first screen of a new number now reports flow type; `isNewUser` / `openSignupAfterCode` logic must not change | | | |
| C2 | P0 | New number: enter correct code (OTP, then Sign up form) | Sign-up form opens with the code held (no verify request yet, same as A) | held-code path | | | |
| C3 | P0 | New number: fill name, submit (Sign up, Customer source if shown, Home) | Account created; same request order (verify/sign-up); same next screen; token stored; lands on Home signed in | `signup_submitted` / `account_created` unawaited; must not delay or reorder the auth calls | | | |
| C4 | P0 | New number: held code rejected on submit (use expired/wrong code) | Same error message and same screen (back to OTP or inline error) as A | held-code failure path; attempt round counter | | | |
| C5 | P0 | Wrong code on first OTP of an existing number | Same error text, field cleared or kept as A, can retry | `otp_failed` on the failure path | | | |
| C6 | P0 | Existing number: phone, OTP, Home (login) | Signed in, Home shows orders; same requests incl. profile fetch | `trackOtpVerified` was removed/moved in `auth_controller.dart`: check order and that sign-in completes | | | |
| C7 | P1 | Resend code after the timer (OTP) | Same timer, same cooldown, one resend request | resend wrapper | | | |
| C8 | P1 | Resend before timer ends | Same disabled state | | | | |
| C9 | P1 | OTP: Back to Sign in, change number, Continue again | Same screens; no duplicate send or stale code | new deferred screen resets counters; Back must not change stack | | | |
| C10 | P1 | Sign up: Back to OTP, enter code again | Same behaviour as A (code asked again or kept) | `attempt_round` logic | | | |
| C11 | P1 | Sign up: submit with empty/invalid name or missing required field | Same validation message, no request | `form_submitted` wrapper before validation | | | |
| C12 | P1 | Customer source screen: choose a source, Continue; also Skip | Same next screen, same body | legacy event with typed text unchanged (see T-0067 note) | | | |
| C13 | P1 | Top-right customer-support button on Sign in, OTP, Sign up | Same support action opens | new `contact_support` click wrapper | | | |
| C14 | P1 | Telegram login button, if enabled on dev | Same flow | `account_created` for Telegram is unawaited | | | |
| C15 | P0 | Kill the app on the sign-up form, relaunch | Same state as A (back to guest/Home) | registration store must not persist a stuck state | | | |
| C16 | P2 | Back (system and app bar) on Sign in as guest | Same exit behaviour | | | | |

### D. Sign-in entry points (T-0074, T-0064, T-0065)

Each case: guest taps the entry, sign in with an existing number, check the return.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| D1 | P0 | Checkout: address row as guest (Checkout, Sign in) | Sign in opens; after sign-in returns to checkout with the same state | `_runGuestSignIn` and its 3 callers edited | | | |
| D2 | P0 | Checkout: schedule row as guest | Same | same file | | | |
| D3 | P0 | Checkout: confirm button as guest | Guest gate then sign-in; no order request before sign-in | `sign_up_gate` click wrapper | | | |
| D4 | P1 | Home sign-in card | Same as B4 plus return to Home | | | | |
| D5 | P1 | Banner detail CTA as guest (Banner detail, guest dialog, Continue) | Dialog text and Continue behaviour same | `banner_detail_screen.dart`, `banner_detail_screen2.dart` edited | | | |
| D6 | P1 | Locked tab (Bookings / Profile) | Same as B2, then lands on the tapped tab after sign-in on both | `entry:` edit in `app_screen.dart` | | | |
| D7 | P1 | Deep link that needs login (open a booking or promo link as guest) | Same sign-in prompt, then the link target | `deep_link_service.dart` | | | |
| D8 | P1 | Push tap as guest (see section M) | Same | | | | |

### E. Home and discovery (T-0064)

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| E1 | P0 | Tap each banner in the carousel (Home, Banner detail / webview) | Same destination per banner; Back returns to Home with scroll kept | `custom_banner_list.dart` closure; legacy banner event still fires | | | |
| E2 | P0 | Tap each service tile (Home to Service detail) | Same service opens, Back works | named routes `/<WidgetClass>` | | | |
| E3 | P1 | See more / See all on each section (See more screen) | Same list, same item opens | `see_more_screen.dart` | | | |
| E4 | P0 | Search: open, type a service name slowly, then fast, clear, retype same text (Service search) | Same results, same order; no flicker or lost keystrokes; result tap opens same service | 600 ms debounce added; result tap inside debounce | | | |
| E5 | P1 | Search: no results; search with Enter (submit) | Same empty state; submit does not open anything extra | `onSubmitted` added | | | |
| E6 | P1 | Home promo popup / floating promo button as guest and member | Same popup, same close, same CTA target | `promo_popup`, `custom_promo_content_view.dart`; FAB closures | | | |
| E7 | P1 | Announcement / offer popup on launch; close and CTA | Same | `custom_announcement_detail_sheet.dart` | | | |
| E8 | P1 | Category alert sheet from a Home tile (Instant not available etc.) | Same sheet, same buttons | `custom_category_selection_alert_sheet.dart` | | | |
| E9 | P1 | Contact support sheet from Home | Same options; each opens the same app/link | `custom_contact_support_sheet.dart` | | | |
| E10 | P1 | Telegram banner / widget on Home | Same link opens | `custom_telegram_banner.dart`, `telegram_widget.dart` | | | |
| E11 | P1 | "Latest from bEasy" section: open an item; "Turn on notifications" banner (Not now, Turn on) | Same item opens; Turn on shows OS prompt the same way; Not now hides the banner | `home_latest_from_beasy_section.dart`; permission check before the request | | | |
| E12 | P1 | Pull to refresh Home | Same reload and same requests | | | | |
| E13 | P1 | Notification offer banner on Home | Same | `custom_notification_offer_banner.dart` | | | |

### F. Service detail: Standard and Instant (T-0082, T-0065)

Screens: Home, Sub category, Category building (service detail), Option selector, Product selector, Deep Cleaning, Unavailable schedule sheet.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| F1 | P0 | Standard service: open, pick option and duration, change quantity, Continue to checkout | Same prices, same total, same requests (preview/bulk) | `product_option_selector.dart`, `category_product_selector.dart` edits; `setBookingMode` runs after first selection: a duration auto-picked earlier must still be picked | | | |
| F2 | P0 | Instant service: open (hour auto-pick) | The same default hour is selected on A and B; same price | `_instantHours` cleared in `setBookingMode`; auto-pick order (reviewer note) | | | |
| F3 | P0 | Instant: change hours up and down, check price after each | Price equals A at every step | | | | |
| F4 | P0 | Instant: tick and untick rooms | Same hour suggestion; same total | `instant_room_toggled` wrapper | | | |
| F5 | P0 | Instant: pick fewer hours than rooms need (under-pick warning) | Warning appears at the same moment with the same text; Continue behaves as A | `_underpickReported` flag must not alter the warning | | | |
| F6 | P0 | Instant: fully booked day / no availability | Same fully-booked state and message | availability counter helper must not throw | | | |
| F7 | P0 | Instant: "Need more than 2 hours?" to Deep Cleaning and back (round trip) | Same navigation; Instant selections (hours, rooms) restored the same as A after returning | stash/restore of selection; return labelled as refresh | | | |
| F8 | P0 | Instant same-day: choose a start so the time is too short, Continue (Unavailable schedule sheet) | Same error sheet, same text, same button results | `custom_unavailable_schedule_sheet.dart` changed (removed `app_error`); Instant time selector | | | |
| F9 | P1 | Instant: open the time selector, pick time, cancel the picker | Same selected time; cancel keeps previous | picker returns `TimeOfDay`; tracker reads it | | | |
| F10 | P1 | Instant: schedule grace (retry after "too short", if offered) | Same grace behaviour | `schedule_grace_result` | | | |
| F11 | P1 | Service detail: pair-service add / remove / select (Pair service) | Same totals | `pair_service_*` wrappers | | | |
| F12 | P1 | Add-on plus, minus, remove, quantity | Same totals and limits | `addon_*` wrappers | | | |
| F13 | P1 | Info sheets: task info, equipment, cancellation policy | Same sheets, same content | opened with `overlaySettings` or manual overlay | | | |
| F14 | P2 | Open the same service twice quickly, Back | No double route | route-name change | | | |

### G. Checkout: address, schedule, coupon, add-ons, payment (T-0065, T-0077)

Screens: Confirm and pay checkout, Address list, New location, Schedule sheet, Apply coupon list, Cross-sell sheet, Payment methods sheet, Payment result sheet, Payment ABA webview.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| G1 | P0 | Checkout opens from a Standard service (existing customer) | Same summary, price lines, totals, same bulk preview request | `confirm_and_pay_checkout_screen.dart` +77 lines | | | |
| G2 | P0 | Address list: select another saved address | Same selection shown; preview refreshed same way | `address_selected` lines only | | | |
| G3 | P0 | Add a new address from checkout (Address list, map, save, back to checkout) | New address is selected on return (add flow pops `true`) | fix in T-0077 review: the `true` branch | | | |
| G4 | P1 | Edit and delete an address from the list | Same result and confirmation dialogs | | | | |
| G5 | P0 | Schedule sheet: open, choose date and slot, confirm | Same slots, same chosen value shown | `checkout_booking_schedule_sheet.dart` +17 | | | |
| G6 | P1 | Schedule sheet: no slots day, dismiss without choosing | Same empty state; previous schedule kept | `no_slots` event only | | | |
| G7 | P0 | Coupon list: open, list loads; apply a valid coupon | Same list, same discount applied, same totals | `apply_coupon_list_screen.dart` +309 lines (largest rewrite) | | | |
| G8 | P0 | Coupon: apply an invalid / not applicable coupon; type a code manually | Same error text | `app_error coupon_apply` wrapper | | | |
| G9 | P0 | Coupon: remove an applied coupon | Totals go back as A | | | | |
| G10 | P1 | Coupon list load failure (block the call) | Same empty/error state | `getApplyCouponList` now returns a bool: callers must behave as before | | | |
| G11 | P0 | Add-ons in checkout: add, change quantity, remove | Same totals | | | | |
| G12 | P1 | Cross-sell sheet: shown, add an item, dismiss | Same | `show()` helper outside tracker; manual overlay event | | | |
| G13 | P0 | Payment method sheet: open, change among wallet, KHQR/deeplink, card, cash | Same methods list, same selection, same total | `payment_controller.dart` +129 | | | |
| G14 | P0 | Confirm with cash / wallet (whichever dev supports) | Order created; same create-order request and body; result sheet identical; order in Bookings | `order_create` error wrapper, `trackOrderCreated` | | | |
| G15 | P0 | Confirm with ABA payment: webview opens, complete payment on sandbox (Payment ABA webview, Payment result sheet) | Same webview URL behaviour, success result sheet, order shows as paid | webview error handler `_errorTracked` skips -10, -1002, -999 custom-scheme redirects; flow unchanged | | | |
| G16 | P0 | ABA payment: fail / reject in the webview | Same failure sheet, same retry | `custom_payment_result_sheet.dart` | | | |
| G17 | P0 | ABA payment: leave idle for the 3-minute timeout | Same timeout handling and message | timeout wrapper | | | |
| G18 | P0 | ABA payment: Back (app bar) and system Back in the webview | Same: same screen returned to, same order status | `back` event; WillPop behaviour | | | |
| G19 | P1 | ABA payment: ABA app not installed / deeplink launch fails (`openOtherApp`) | Same fallback message | `app_launch_failed` wrapper | | | |
| G20 | P1 | Payment webview loads an invalid URL or no network mid-load | Same error view | load-error handler | | | |
| G21 | P1 | Order creation fails (block the request, or use a rule that fails) | Same error dialog; user stays on checkout | create failure wrapper must not swallow or double-show | | | |
| G22 | P1 | Appointment status screens after booking (v2, v3) | Same status, same next buttons | `appointment_status_*` +5/+3 lines | | | |
| G23 | P2 | Checkout: leave and return (Back, then reopen) | Same state kept or reset as A | route naming | | | |

### H. Top-up (T-0065)

Screens: Sub top-up point, Top-up, Top-up ABA payment, Top-up status, Top-up history.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| H1 | P0 | Open top-up; tap each amount chip | Same amount shown, same bonus text | `top_up_screen.dart` +56 | | | |
| H2 | P0 | Type a custom amount; minimum and maximum; clear it | Same validation and button state; chip selection clears as A | typed amount de-duplication | | | |
| H3 | P0 | Top-up with ABA: payment screen, pay on sandbox, status screen | Same request body (amount), same webview, same status result and balance | `topup_webview` wrappers | | | |
| H4 | P0 | Top-up ABA: fail / cancel / back / 3-minute timeout | Same message and balance unchanged | `top_up_aba_payment_screen.dart` | | | |
| H5 | P1 | Top-up ABA app launch fails | Same fallback | | | | |
| H6 | P1 | Top-up status screen: success and pending states | Same text, Done returns the same screen | `top_up_status_screen.dart` | | | |
| H7 | P1 | Top-up history: open, scroll, refresh, empty state | Same list and paging | `top_up_history_screen.dart` +23 | | | |
| H8 | P1 | Wallet balance shown on Home / Profile after top-up | Same number on A and B | | | | |

### I. Bookings and order detail (T-0066, T-0079)

Screens: Bookings list, Order detail v2, Extend sheet, Reschedule sheet, Cancel sheet, Review form, Instant progress card.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| I1 | P0 | Bookings list: switch each tab/filter; pull to refresh; scroll | Same orders per tab, same paging | T-0066 still being finished: re-run after it merges | | | |
| I2 | P0 | Open an order (upcoming, completed, cancelled) | Same detail layout and buttons per status | `order_detail_screen_v2.dart` +56 | | | |
| I3 | P0 | Instant order: progress card through its stages (searching, assigned, on the way, working, done) | Same stage text and card; stages update on refresh | `order_detail_instant_progress_card.dart` +20 | | | |
| I4 | P0 | Extend sheet (Instant): open, choose extra hours, pay (wallet/ABA) | Same price, same request, same result | `order_detail_extend_sheet.dart`; overlay plus named event both fire | | | |
| I5 | P0 | Reschedule: open sheet from the card, choose day and slot, continue | Same slots, same success message, new time shown | `order_detail_reschedule_sheet.dart` +30; `rescheduleOrder` one result per attempt | | | |
| I6 | P0 | Reschedule: slot unavailable / backend failure (force with a taken slot) | Same error and sheet stays; can retry | failure mapping; grace token request | | | |
| I7 | P1 | Reschedule: dismiss by swipe and by barrier tap; dismiss while the request is running | Same: no state change on dismiss; in-flight request completes as A | in-flight dismiss no longer reports abandon | | | |
| I8 | P1 | Reschedule sheet: Change payment, payment rows, Try again, Back steps | Not reachable for paid path (free window only); the screens that open must match A | `_isFreeWindow => true` | | | |
| I9 | P0 | Cancel: **the cancel row in the reschedule/cancel card is commented out** (`order_detail_reschedule_cancel_card.dart`), so the Cancel sheet cannot be opened from the UI on either build. Check the row is absent on both. Cancel path inside the reschedule sheet: open reschedule and look for a "cancel instead" option | Same on A and B (absent or present as A). If a temporary enable is needed to test the cancel sheet, do it as a local patch on both A and B and say so in the log | `order_detail_cancel_sheet.dart` +33 only wraps; test needs the temp enable (T-0079 note) | | | |
| I10 | P1 | Cancel sheet (only if enabled as above): reason, confirm, success; API failure; dismiss at reason and confirm; cancel then reschedule | Same requests and messages | | | | |
| I11 | P1 | Review: open from a completed order, rate, write text, submit; also skip | Same request body; same thanks state | review form event carries counts only; text must still be sent | | | |
| I12 | P1 | Bottom actions: contact us, pay now, book again, leave review | Same targets | not wrapped for cancel events; legacy events kept | | | |
| I13 | P2 | Order detail from a push or deep link | Same screen and data | | | | |

### J. Notifications inbox and detail (T-0064)

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| J1 | P0 | Open the inbox (Notifications tab); badge count | Same list, same unread badge | `announcement_list_screen.dart` +42 | | | |
| J2 | P1 | Open an item (detail), Back | Same detail; item marked read the same | `announcement_not_details.dart` | | | |
| J3 | P1 | Notification permission banner in the inbox: Not now, Open settings | Same behaviour; OS settings opens | permission read before request | | | |
| J4 | P1 | Mark all read / pull to refresh / empty state | Same | | | | |

### K. Profile, settings, language (T-0067)

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| K1 | P0 | Profile tab: every menu row opens its screen; Back each time | Same screens and order | `profile_screen.dart` +36 | | | |
| K2 | P0 | Language: switch km to en to zh to km (Language screen) | UI language changes immediately on all screens; selection kept after restart; server language sync request as A | `localization_controller.dart`, `language_screen.dart`; silent sync after sign-in must not change language | | | |
| K3 | P1 | Help center: open topics, contact options | Same | `profile_help_center.dart` | | | |
| K4 | P1 | About bEasy: links (Facebook, Telegram, website, privacy) | Same external app opens | `URLLauncher.openApp` indentation fixed | | | |
| K5 | P1 | Tutorial center and tutorial detail (video/list) | Same playback and navigation | `toturial_center.dart`, `toturial_detail.dart` | | | |
| K6 | P0 | Profile info: edit name, phone/email if allowed, gender, birthday; Save | Same validation, same update request and body, same success message | `profile_info_screen.dart` +63 | | | |
| K7 | P1 | Profile info: change photo from gallery and camera; deny permission | Same prompts and result; denied shows the same message | permission status read before `request()` | | | |
| K8 | P1 | My coupons (profile coupon screen): list, copy, refresh; push banner | Same | `profile_coupon_screen.dart` | | | |
| K9 | P1 | Logout dialog: cancel, then confirm | Same; confirm clears session and lands on the same screen | user properties cleared on logout | | | |
| K10 | P1 | Delete account dialog: cancel; confirm only on a throwaway account | Same dialog text and result | | | | |
| K11 | P1 | Address management from Profile (see section L) | Same | | | | |
| K12 | P2 | Wallet / points card in Profile | Same balance | | | | |

### L. Location and address (T-0077)

Screens: Address list (`location_screen`), New location form + map (`new_location_screen`), Search location (`new_search_location`), out-of-range sheet.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| L1 | P0 | Open the map with permission granted (auto current location) | Map centres on the current position; same pin | `permission_result auto_open` check must not show the OS prompt differently | | | |
| L2 | P0 | Permission denied once (fresh install, tap Don't allow) | Same message or fallback map position | permission read-then-request order | | | |
| L3 | P0 | Permission denied forever (Android: deny twice; iOS: deny) then tap "use current location" | Same dialog offering Settings | | | | |
| L4 | P0 | Location services off, tap "use current location" | Same prompt to turn services on | | | | |
| L5 | P0 | Current-location tile with permission granted | Pin moves to the device position; address text filled as A | | | | |
| L6 | P0 | Drag the map pin; tap on the map | Pin and address follow exactly as A; no flicker, no lost drag | drag detection (first idle ignored, 800 ms flag) | | | |
| L7 | P1 | Search a place (type, result tap) (Search location) | Same results, debounce as before; result moves the pin | debounce lives in the controller | | | |
| L8 | P0 | Save a new address: fill form, save (also set default) | Same create request and body; address in list | `reportSave` guard | | | |
| L9 | P0 | Edit an address and save; set default | Same | | | | |
| L10 | P0 | Pin outside the service area, try to save (out-of-range sheet) | Same sheet, same buttons; save blocked as A | coverage check | | | |
| L11 | P1 | Delete an address (confirm, cancel) | Same dialog; list updates | | | | |
| L12 | P1 | Address API failure on save/delete (block call) | Same error | | | | |
| L13 | P1 | Address selected for checkout by all routes: pick existing, add new, from Profile list | Same selection arrives in checkout | `_tracker.addressSelected` lines | | | |

### M. Chat (T-0067)

Screens: Chat list (message view), Chat room (message details), Gallery sheet, Media preview, Create story, no-chat screen. Chat has its own client and MQTT.

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| M1 | P0 | Open chat list as a customer with orders; open a room | Same rooms, messages load, MQTT connects and subscribes the same | `message_view.dart`, `message_details.dart` +36 | | | |
| M2 | P0 | Send a text message; receive a reply (use the portal/staff side) | Message appears and delivers the same; no duplicate send | `send_message` wrapper | | | |
| M3 | P1 | Send an image from gallery (gallery sheet, media preview: retake, continue) | Same upload and message | `media_send`, `media_retake` | | | |
| M4 | P1 | Camera icon in chat: grant, deny, deny forever | Same prompts and fallback; camera opens | status read before request | | | |
| M5 | P1 | Send a video | Same | | | | |
| M6 | P1 | Create story: shutter, switch camera, gallery, close | Same behaviour; permission prompts as A | `create_story_screen.dart` +28; `_loadLastImage` permission read | | | |
| M7 | P1 | Chat back button, no-chat-found screen (new customer) | Same | | | | |
| M8 | P1 | Image / video viewers and attachments open | Same | not wrapped | | | |

### N. Push, deep links, lifecycle (T-0063, T-0064)

| ID | Pri | Steps | Expected | Tracking risk | Res A | Res B | Verdict |
|---|---|---|---|---|---|---|---|
| N1 | P0 | Push tap, app killed, logged in (send from the portal or Firebase console to a dev token) | Same screen opens | `firebase_messaging_service.dart`, `local_notifications_service.dart` edits | | | |
| N2 | P0 | Push tap, app in background, logged in | Same screen | | | | |
| N3 | P0 | Push tap as guest | Same prompt or screen | | | | |
| N4 | P1 | Push tap while on checkout | Same (no state loss) | | | | |
| N5 | P1 | Foreground push (local notification shown); tap | Same | | | | |
| N6 | P0 | Deep link (service, order, banner/promo, coupon) with app killed, background, foreground | Same target in all three states | `deep_link_service.dart`, `helper.dart` edit | | | |
| N7 | P0 | Home button, return after 10 s and after several minutes (resume) | Same screen kept; no relaunch, no duplicate splash | `my_app.dart` lifecycle pause/resume hook | | | |
| N8 | P1 | Background mid-payment webview, return | Same webview state | | | | |
| N9 | P1 | Background during OTP timer | Timer continues as A | | | | |
| N10 | P1 | Kill and relaunch signed in | Same Home, still signed in | | | | |
| N11 | P2 | Switch app language in OS settings while app runs | Same | | | | |

Rotation is not tested.

## 4. Highest risk first (run these before everything else)

All reviewer-flagged. If any fails, stop and fix before running the rest.

| # | Item | Cases | Why |
|---|---|---|---|
| 1 | Wrapped callbacks turned into closures (tiles, banners, FAB, cards) | E1, E2, E6, E8, E10 | a closure that drops the original return value or `async` breaks the action |
| 2 | Sheets and dialogs now opened as named routes via `overlaySettings` | F7, F13, G5, G12, G13, I4, I5 | route name change; Back / dismiss and result values must be unchanged |
| 3 | Auth path edited (`auth_controller.dart`, OTP, sign-up) | C1 to C6, D1 to D3 | sign-in is the gate for every purchase |
| 4 | ABA payment and top-up webview error handling | G15 to G20, H3 to H5 | money; error handler must not hide or double a failure; custom-scheme redirects ignored |
| 5 | Payment controller and coupon list rewrite | G7 to G10, G13, G14, G21 | +129 and +309 lines; amounts |
| 6 | Instant: auto hour pick, under-pick warning, Deep Cleaning round trip, same-day error | F2 to F8 | selection state cleared in `setBookingMode` |
| 7 | Permission prompts (location, photos, camera, push) | L1 to L4, K7, M4, M6, J3, E11 | status is now read before `request()`; prompt must still appear the same way |
| 8 | Language switch | K2 | localization controller changed |
| 9 | Search debounce | E4, E5, L7 | new 600 ms timer and `onSubmitted` |
| 10 | Guest sign-in gates and return | B2, B4, B5, D1 to D8 | `entry:` params and tab hook in `app_screen.dart` |
| 11 | Hidden cancel row | I9, I10 | cancel sheet not reachable from the UI on either build; do not report its absence as a diff |
| 12 | Lifecycle hook and push/deep link | N1 to N7 | `my_app.dart` pause/resume |

## 5. Run log and sign-off

### 5.1 Result log template (one row per case run)

| Date | Case ID | Device / OS | Account | Build A result | Build B result | Request diff (file names) | Verdict | Bug / note |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

Keep proxy exports as `case-ID-A` / `case-ID-B` next to the log. Do not paste tokens or phone numbers into docs or tasks.

### 5.2 Sign-off checklist

- [ ] Section 4 items all PASS on Android and iOS
- [ ] All P0 cases PASS on A and B (both columns filled, no blanks)
- [ ] All P1 cases run; any DIFF has a bug ID and is fixed or accepted by the owner
- [ ] Every request diff on the dev API host is explained (none expected)
- [ ] No new crash or ANR on B in `adb logcat` / Xcode console vs A (search `FATAL`, `ANR`, `Unhandled Exception`, `INVALID TRACKING EVENT`)
- [ ] B: `fvm flutter analyze` clean
- [ ] B: `fvm flutter test` passes; the only failure is the pre-existing `test/widget_test.dart` (it also fails on `main`)
- [ ] T-0066 (orders) and T-0069 (Meta/AppsFlyer) merged into `feat/tracking-integration`, cases I and the checkout/search events re-run after that merge
- [ ] Optional: DebugView smoke events (section 6) seen
- [ ] Result log attached to T-0073; then the PR to `main` can be prepared (commit and push only on the owner's instruction)

## 6. Optional: Firebase DebugView

Not part of the A/B behaviour check. Events are only sent when `Helper.kMode && !PhoneNumbers.instance.isStaff` (`lib/analytics/analytics_helper.dart:22`; `kMode = kReleaseMode`, `lib/helper/helper.dart:55`). So:

- Use a **release build** of the dev flavor: `fvm flutter run --release --flavor dev --target lib/main_dev.dart` (Android) or the same on iOS.
- Sign in with a **non-staff** phone (a staff phone sends nothing, by design).
- Debug builds log `INVALID TRACKING EVENT "..."` in the console on a contract breach; a quick way to catch bad params without a release build.

Enable DebugView:
- Android: `adb shell setprop debug.firebase.analytics.app suntel.beasy.app.dev` (package from `applicationId` + `.dev` in `android/app/build.gradle.kts`; turn off with `.none.`). Open Firebase console, Analytics, DebugView.
- iOS: add `-FIRDebugEnabled` to the Run scheme arguments (Xcode), run from Xcode.
- The app pauses collection between events (`_pauseCollection`); each event re-enables it, so events may arrive a few seconds late.

Handler maps and expected events per slice (do not copy them here):

| Area | Task file |
|---|---|
| Foundation, registry, screen_viewed/exited | `Tasks/3-in-progress/App-tracking-foundation.md` |
| Registration | `Tasks/3-in-progress/App-registration-tracking.md` |
| Location | `Tasks/3-in-progress/App-location-tracking.md` |
| Instant | `Tasks/3-in-progress/App-instant-cleaning-tracking.md` |
| Cancel / reschedule | `Tasks/3-in-progress/App-cancel-reschedule-tracking.md` |
| Home / discovery | `Tasks/3-in-progress/App-tracking-home-discovery.md` |
| Account / profile / chat | `Tasks/3-in-progress/App-tracking-account-profile-chat.md` |
| Booking / checkout | `Tasks/3-in-progress/App-tracking-booking-checkout.md` |
| Orders | `Tasks/3-in-progress/App-tracking-orders.md` |
| Meta / AppsFlyer | `Tasks/3-in-progress/App-tracking-meta-appsflyer.md` |

Smoke events to check first (names as in each task's handler map; if one is missing there, check the task file before raising a bug):

1. `screen_viewed` on Home, then on each tab switch (with `screen_group`, `entry_type`)
2. `screen_exited` after more than 1 s on a screen, and after background/foreground
3. `element_clicked` on a Home banner tile
4. `overlay_viewed` when a named sheet opens (payment method sheet)
5. `search_performed` once after typing in search (no raw long digits or emails)
6. `registration_started`, `otp_submitted`, `otp_code_entered`, `signup_submitted`, `account_created` in order for a new number
7. `permission_result` for location on first map open
8. `address_selected` when an address reaches checkout
9. `order_created` / checkout events and `app_error` on a forced payment failure
10. `setting_changed` on a language switch (and no event on the silent server sync after sign-in)

Then compare one staging batch with the registry as T-0073 requires.

## 7. Facts not verified when this plan was written

- Whether the dev API certificate and network security config allow proxy inspection on a release build (use debug builds or an emulator with a system CA).
- The dev OTP rule and which dev accounts have orders/coupons/wallet balance (owner to fill in section 2.2).
- Whether cancel is reachable from the reschedule sheet (code review says the cancel row is commented out; confirm on device).
- Whether the update prompt, forced-update and holiday alert can be triggered on dev without backend config.
- T-0066 and T-0069 were still being finished: their cases (I1 to I3, I11, E4 search events) must be re-run after they merge.
- Case counts and the "about 270 removed lines" figure come from `git diff main..HEAD` at the time of writing.
