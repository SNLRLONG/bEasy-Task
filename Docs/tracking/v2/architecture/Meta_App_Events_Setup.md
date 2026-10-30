# Meta app events: step-by-step guide

**Updated:** 21 Oct 2026 · Task: [T-0031](../../../../Tasks/2-todo/Cross-meta-app-events-setup.md) · Code: `bEasy-app/lib/app_tracking/facebook_helper.dart`

Follow the steps in order. Each step says where to go, what to click, what you should see, and what to do if you do not. After each step write the result in **Result** and set **Status**: ✅ done · ⏳ not done · ⚠️ problem found.
Meta changes its menus often; if a label differs slightly, look for the closest one. Do not paste secrets (app secret, client token, install referrer key) into this file.

**What the app sends to Meta** (names stay as they are; do not rename):

| Event | Type | When | Parameters |
|---|---|---|---|
| ViewContent | standard | user opens a service page | `service_name` |
| SelectService | custom | user selects a service | `service_name` |
| InitiateCheckout | standard | user reaches checkout | `value`, `currency=USD`, `service_name` |
| AddPaymentInfo | standard | user picks a payment method | `value`, `currency=USD`, `service_name` |
| ViewBooking | custom | user opens Bookings / an order | `service_name`, `order_id` |
| CompleteRegistration | standard | new user registers | method |
| Purchase | standard | booking completed | `value`, `currency=USD`, `order_id`, `service_name` |

---

## Step 1. Open the right Meta app
- **Go to:** developers.facebook.com → **My Apps**.
- **Click:** the app dropdown at top left → pick **bEasy** (not "bEasy Chat").
- **You should see:** App ID `1466778561367463`.
- **If not:** you are in the wrong app; the app uses that ID.
- **Status:** ✅ · **Result:** App ID matches `strings.xml` (Android) and `Info.plist` (iOS).

## Step 2. Check the iOS settings
- **Go to:** left menu → **App settings → Basic**, scroll to the **iOS** card.
- **You should see:** Bundle ID `suntel.beasy.app`, iPhone Store ID and iPad Store ID.
- **Do:** compare the Store ID with your App Store link (`apps.apple.com/.../id<NUMBER>`). The number must be the same.
- **Status:** ✅ · **Result:** Bundle ID matches the app. Store ID `6745190697` confirmed to match the App Store (21 Oct).

## Step 3. Check the Android settings
- **Go to:** same page, **Android** card.
- **You should see:** Package `suntel.beasy.app`, Class name `suntel.beasy.app.MainActivity`, Install Referrer Decryption Key filled, 4 Key hashes.
- **Do:** the install referrer key being filled is what lets Meta credit Android installs. Leave it.
- **Do (key hashes):** open Google Play Console → your app → **Test and release → App integrity** → copy the SHA-1 of the *App signing key*. Convert it to a key hash (or ask whoever set up the hashes) and check one of the 4 is for it. Without it, release builds may fail some Meta features (not event tracking).
- **Status:** ✅ · **Result:** package, class, 4 key hashes and install referrer key are present (21 Oct). Marked done on your confirmation; the hashes were redacted in the screenshot, so I did not compare them with the Play signing key myself.

## Step 4. Know which toggles are on (no action)
- **You should see:** iOS "Log in-app events automatically" on; Android "Log In-App Purchases Automatically" on.
- **Do:** nothing. They only cover App Store / Google Play billing. bEasy has no in-app purchases, so the app's own Purchase event is not duplicated.
- **Status:** ✅ · **Result:** leave as is.

## Step 5. Compare the client token
- **Go to:** **App settings → Advanced** → find **Client token**.
- **Do:** compare with `facebook_client_token` in `bEasy-app/android/app/src/main/res/values/strings.xml` (and `FacebookClientToken` in `ios/Runner/Info.plist`). Do not paste it here.
- **You should see:** the same value.
- **If not:** the SDK cannot send events. Tell me and we fix the app.
- **Status:** ⏳ · **Result:**

## Step 6. Confirm who owns the bEasy app (developer), then hand the ad account to marketing
**Developer part: done (21 Oct)**
- The main **bEasy** app (`1466778561367463`) is owned by the **Suntel Cambodia** business portfolio. The **bEasy** portfolio lists only bEasy Chat. Switch portfolios with the name at the top left of Business settings.
- People on the app: ReportBot (full access), Suntel Cambodia (you, full access), Conversions API System User (partial: view insights and test app).
- The Suntel Cambodia portfolio has one ad account, `2234461800288146`. The app has 0 ad accounts connected today.
- **Status:** ✅ (developer part)

**Marketing part: owner marketing / business admin, status ⏳**
Send marketing this (copy as is):
> The bEasy app (ID 1466778561367463) is owned by the Suntel Cambodia business portfolio, and no ad account is connected to it yet. Please: (1) open Ads Manager and tell me the ID of the ad account that runs the October campaigns. (2) If it is 2234461800288146: Business settings (portfolio Suntel Cambodia) > Accounts > Apps > bEasy > Connect assets > tick that ad account > Add. (3) If it is another ID, tell me which business portfolio it is in; the app must be shared with that portfolio (Apps > bEasy > Assign partner) and the ad account connected there. Reply with the ad account ID and the portfolio.
- **You should see:** bEasy app > **Connected assets** tab lists the ad account.
- **Result:** waiting for marketing.

## Step 7. Find the app in Events Manager
- **Go to:** business.facebook.com → **Events Manager** (grid icon, left) → **Overview**.
- **You should see:** the bEasy app as a dataset with a total-events number for the last 28 days.
- **Found (21 Oct):** two datasets both named **bEasy**: `1466778561367463` (the app) and `1734456480769470`. Each shows **21.3K events in 28 days**, source "Facebook SDK", with the same graph. The ad account selected at the top right is `2234461800288146`.
- **Meaning:** Meta is already receiving events from the app, so the SDK works in production. The total counts all events, including automatic ones (app opens), so it does not yet prove the 7 booking events arrive. The two datasets are one: the app `1466778561367463` is linked to dataset `1734456480769470` (Datasets > bEasy > right panel "Link to dataset").
- **Do:** click **View event activity** on the `1466778561367463` card (needed for Step 8).
- **Status:** ✅ · **Result:** dataset exists and receives events (21.3K / 28 days). The app is linked to dataset `1734456480769470` (same data).

## Step 7b. Read the Diagnostics warning
- **Go to:** Events Manager > Datasets > bEasy > **Diagnostics** tab (it shows a "1").
- **Found (21 Oct):** one active warning, **"Confirm domains that belong to you"**, detected Jul 21, 2026: web domains recently started sending data into this dataset: `staging-landing-alpha.vercel.app`, `staging-landing-kohl.vercel.app`, `beasy.info`.
- **Meaning:** website events reach this dataset. The user says they come from the landing page. In this workspace, `bEasy-landing-page-v2` has Firebase Analytics only (no Meta Pixel, tag manager or Conversions API) and deploys to Firebase Hosting, not Vercel. The warning names `staging-landing-*.vercel.app`, so another landing project on Vercel, or a pixel added outside the code (tag manager, Meta web setup), is likely the sender. Not confirmed; the backend repo is not here.
- **Why it matters:** the two `staging-landing-*` sites are staging. Staging events in the production dataset pollute counts and audiences.
- **Meta's own text (21 Oct):** clicking **Review domain(s)** only opens "How to fix the error". Data from domains **not on the allow list may still show in Events Manager but is not processed, so no action is needed.** So the warning is harmless for now.
- **Do:**
  1. The **Visit Settings page** button does nothing. Go manually: Events Manager > **Datasets** > top **bEasy** (`1734456480769470`) > **Settings** tab > scroll to **Traffic permissions**. If not there, try the other bEasy (`1466778561367463`). Look only; change nothing. Write down whether an allow list exists and which domains are in it.
  2. **Create or extend the allow list with the production domains only:** `beasy.info`, `www.beasy.info` if the site answers on `www`, and any other production landing domain (ask whoever deploys it). Leave the two `staging-landing-*.vercel.app` domains off so staging events stay unprocessed. An allow list that is too short blocks a legitimate sender, so list every production domain first. Then re-check Diagnostics; the warning should clear.
  3. Ask backend and marketing who sends website events to this dataset (Conversions API? tag manager?) and whether staging should use a separate test dataset.
  4. Ask backend whether Purchase is also sent server-side. If both the app and the server send it without a shared event ID, Meta counts it twice (see Step 15).
- **Status:** ✅ · **Result:** allow list added with the production domain(s); the two staging domains (`staging-landing-alpha.vercel.app`, `staging-landing-kohl.vercel.app`) were ignored in Diagnostics (21 Oct). Staging data is not processed. Optional re-check on 24 Oct: if the warning returns, stop the staging sites sending to the production dataset. Still open (not blocking): who sets up the web events (landing page owner / backend).

## Step 8. Parameter check on a device (narrowed)
- **Why (21 Oct):** the production event screen (Overview > event row) shows only the count over time and a **Data Source** filter. It does not show parameters, so `value`, `currency`, `order_id` and `service_name` cannot be read from production data. Only the **Test events** tab shows them, and that needs a device.
- **Scope:** (1) **ViewBooking** and **SelectService**, which do not reach Meta in production: open the Bookings tab once, select a service once, and watch whether each appears in Test events. (2) The events that carry money: **InitiateCheckout**, **AddPaymentInfo**, **Purchase**. First try expanding their row in Overview: the **Parameters** panel may already list the parameters Meta has seen; only use the device if it says none were detected.
- **Do:** run a dev build: `fvm flutter run --flavor dev --target lib/main_dev.dart`. Events Manager > Datasets > top bEasy (`1734456480769470`) > **Test events** tab. Follow the page to add your device. Go to checkout, pick a payment method, complete a test booking. Open each event in the list and read its parameters.
- **You should see:** `value` is a number above 0 (not empty), `currency` = USD, `service_name` filled; Purchase also has `order_id`.
- **If dev events do not show:** the dev package `suntel.beasy.app.dev` is not registered in Meta; add it under App settings > Basic > Android > Package Names. The terminal also prints `[FacebookHelper] ... logged` per event.
- **Status:** ⏳ · **Result:**

## Step 9. ViewContent
- **How to check:** the count and last-received time come from Datasets > bEasy > Overview (the event row). Parameters need Step 8 (Test events). In the event's detail screen, the **Data Source** dropdown may split app from web traffic; use it to see whether any of the count comes from the website.
- **In the app:** open any service page.
- **In Test events you should see:** `ViewContent` with `service_name`.
- **Status:** ⏳ · **Result:** Production data (Events Manager > Datasets > bEasy, Sep 23 to Oct 20, 21 Oct): `ViewContent` 4.9K, last received 26 min ago. Parameter `service_name` not checked yet.

## Step 10. SelectService
- **In the app:** select a service to start booking.
- **You should see:** `SelectService` with `service_name`.
- **Status:** ⚠️ · **Result:** `SelectService` does NOT appear in Events Manager for Sep 23 to Oct 20. Under the Custom events filter the only custom event is `fb_mobile_aem_auto_setup_opt_in` (Meta's own, 117). Code is on `main` since commit `55f4812e` (Sep 23), fired from `category_building_screen.dart` (`_trackSelectService`). Cause unknown; see Step 8.

## Step 11. InitiateCheckout
- **In the app:** go on until the checkout / confirm-and-pay screen.
- **You should see:** `InitiateCheckout` with `value` (a number, not empty), `currency` = USD, `service_name`.
- **Parameters (21 Oct):** Events Manager shows "No event parameters were detected" and the event activity list is empty with "Custom data not being displayed: parameters shown as _removed_". Meta hides custom data here, so parameters cannot be read from production. App tests pass (`fvm flutter test test/facebook_helper_test.dart`, 7/7) and show the helper sends the right parameters. Real values still to confirm in Test events.
- **Status:** ⏳ · **Result:** Production data (Events Manager > Datasets > bEasy, Sep 23 to Oct 20, 21 Oct): `InitiateCheckout` 1.3K, last received 24 min ago. `value` / `currency` not checked yet.

## Step 12. AddPaymentInfo
- **In the app:** choose a payment method on checkout.
- **You should see:** `AddPaymentInfo` with `value` and `currency` = USD.
- **Parameters (21 Oct):** Events Manager shows "No event parameters were detected" and the event activity list is empty with "Custom data not being displayed: parameters shown as _removed_". Meta hides custom data here, so parameters cannot be read from production. App tests pass (`fvm flutter test test/facebook_helper_test.dart`, 7/7) and show the helper sends the right parameters. Real values still to confirm in Test events.
- **Status:** ⏳ · **Result:** Production data (Events Manager > Datasets > bEasy, Sep 23 to Oct 20, 21 Oct): `AddPaymentInfo` 255, last received 54 min ago. `value` / `currency` not checked yet.

## Step 13. ViewBooking
- **In the app:** open the Bookings tab, then an order.
- **You should see:** `ViewBooking`; `order_id` when you opened an order.
- **Status:** ⚠️ · **Result:** `ViewBooking` does NOT appear either (same filter, same window). It fires in `booking_screen.dart` `initState` on every visit to Bookings, so it should be frequent. Same commit as the 5 standard events that do arrive. Cause unknown; see Step 8.

## Step 14. CompleteRegistration
- **In the app:** register a new test account.
- **You should see:** `CompleteRegistration`.
- **Status:** ⏳ · **Result:** Production data (Events Manager > Datasets > bEasy, Sep 23 to Oct 20, 21 Oct): `CompleteRegistration` 270, last received 26 min ago.

## Step 15. Purchase
- **In the app:** complete a test booking.
- **You should see:** `Purchase` with `value`, `currency` = USD, `order_id`.
- **Take screenshots** of steps 9 to 15 and mask phone numbers and names.
- **If an event is missing or `value` is empty:** write it under Findings. Do not change code in this task; it becomes a new task.
- **Parameters (21 Oct):** Events Manager shows "No event parameters were detected" and the event activity list is empty with "Custom data not being displayed: parameters shown as _removed_". Meta hides custom data here, so parameters cannot be read from production. App tests pass (`fvm flutter test test/facebook_helper_test.dart`, 7/7) and show the helper sends the right parameters. Real values still to confirm in Test events.
- **Status:** ⏳ · **Result:** Production data (Events Manager > Datasets > bEasy, Sep 23 to Oct 20, 21 Oct): `Purchase` 218, last received 53 min ago. `value`, `currency`, `order_id` not checked yet. Compare 218 with completed orders in the portal for the same dates.

## Step 16. Consent on iPhone
- **In the app:** delete and reinstall, and when the prompt appears tap **Ask App Not to Track**. Use the app and book.
- **You should see:** either no events (tracking respected) or events still arriving.
- **Do:** write what happened. Ask marketing and product if that is acceptable; if events still arrive and they want to stop it, that becomes a code task.
- **Status:** ⏳ · **Result:**

## Step 17. Consent on Android
- **Do:** decide with marketing if an in-app consent screen is needed. Today there is none. Write the decision here.
- **Status:** ⏳ · **Result:**

## Step 18. AppsFlyer: make Meta installs count
- **Owner:** Marketing + developer
- **Go to:** appsflyer.com dashboard → app bEasy → **Integration → Integrated Partners** → search **Meta** (Facebook Ads).
- **You should see:** the Meta integration enabled with the ad account linked.
- **If not:** open it, enable **Activate Partner**, follow its steps to connect the Meta ad account. This is what fixes "Meta credited zero installs".
- **Ask marketing:** which objective the October campaigns used. Messages or traffic campaigns never report installs.
- **Status:** ⏳ · **Result:**

## Step 19. Test an install is credited
- **Owner:** Marketing
- **Do:** marketing gives you a test Meta ad link (or a OneLink with `pid=Facebook Ads`). Uninstall the app, click the link on the phone, install, open.
- **Go to:** AppsFlyer → **Overview** (filter: today, media source Facebook Ads).
- **You should see:** 1 install attributed to Meta within about an hour.
- **Status:** ⏳ · **Result:**

## Step 20. Mark priority events
- **Owner:** Marketing
- **Go to:** Events Manager → bEasy → **Aggregated Event Measurement** (or **Settings → Event setup**).
- **Do:** set **Purchase** and **InitiateCheckout** as priority events.
- **Status:** ⏳ · **Result:**

## Step 21. Create the three audiences (marketing)
- **Owner:** Marketing
- **Go to:** business.facebook.com → **Audiences** → **Create audience → Custom audience → App activity** → pick bEasy.
- **Create:**
  1. Include ViewContent, last 30 days; exclude Purchase. Name: viewed, not booked.
  2. Include InitiateCheckout, last 30 days; exclude Purchase. Name: checkout, not purchased.
  3. Include Purchase, last 180 days. Name: past buyers.
- **You should see:** each audience in the list with status "Ready" (size may say "too small" until real users arrive).
- **Status:** ⏳ · **Result:**

## Step 22. Check production
- **After the next release,** wait one day.
- **Go to:** Events Manager → bEasy → **Overview**.
- **You should see:** the same events with real counts, not only test events.
- **Status:** ⏳ · **Result:**

---

## Findings and changes
- 21 Oct: Events Manager hides event parameters ("No event parameters were detected"; activity list says "Custom data not being displayed ... _removed_"). Meta says audiences and custom conversions did not use that data. It does mean we cannot confirm `value`/`currency` from production; Test events is the only view left. The helper tests pass 7/7 (`test/facebook_helper_test.dart`: value, USD, service_name, order_id for all 7 events), so the code builds the parameters; what a real booking passes as `amount` is not yet confirmed.
- ⚠️ 21 Oct: `SelectService` and `ViewBooking` (custom events) never reached Meta in Sep 23 to Oct 20, while the 5 standard events from the same commit did. Next: dev build + Test events (Step 8). If they appear there, production builds do not include them yet; if not, the custom events are being dropped and we investigate the SDK call.
- 21 Oct: production counts Sep 23 to Oct 20: ViewContent 4.9K, Activate app 4.7K, InitiateCheckout 1.3K, CompleteRegistration 270, AddPaymentInfo 255, Purchase 218. All five standard booking events arrive; SelectService and ViewBooking (custom) still to read. Diagnostics warning (staging domains) handled in Step 7b.
- 21 Oct: Events Manager shows 21.3K events in 28 days on the bEasy dataset (`1466778561367463`, source Facebook SDK). The second dataset `1734456480769470` is the one the app is linked to: same data.
- ⚠️ 21 Oct: the bEasy app (`1466778561367463`) belongs to the **Suntel Cambodia** portfolio; the **bEasy** portfolio lists only bEasy Chat. The ad account marketing uses may be in a different portfolio than the app (Step 6). A Conversions API System User already exists on the app, which suggests server-side events may already be set up; ask backend before adding any (not checked).
- ⚠️ Dev and staging builds use package names `suntel.beasy.app.dev` / `.staging`, not registered in Meta. Dev events may not arrive (Step 8).
- ➖ The "Currently ineligible for submission" banner (missing app icon and user data deletion URL) blocks App Review only, not event tracking. The app is Published.
- 21 Oct: only 5 of the 7 events are Meta standard events; SelectService and ViewBooking are custom. Audiences use the standard ones.

## Done when
Step 8 (parameters), steps 9 to 15 (counts), 19 and 22 are verified and the audiences from Step 21 exist. Code fixes found here become new tasks.
