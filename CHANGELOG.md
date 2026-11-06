# Changelog

Merges to `main` across all projects, newest first. Prefix: `[Portal]` `[App]` `[Mini-App]` `[Landing-Page]` `[Backend]` `[Cross]`. Format and rules: `Tasks/README.md`.

## 2026-11-05 · [Portal] feat(order): order list, order details and Direct Sale on v2 components, permission guards
- Owner: SNLRLONG (Seavlong)
- Task: Tasks/4-testing/Portal-kpi-order-management-revamp.md (T-0095, T3) and Tasks/4-testing/Portal-kpi-direct-sale-v2.md (T-0096, T2), same branch
- Branch/PR: feat/order-management-revamp · PR to be added
- What changed:
  - **Order list:** v2 look (search, Filters panel, active chips, flat paginated rows). Status filter now sends ALL CAPS (title case returned nothing). Payment filter is available for every order type. Filters live in the URL. Old `?status=Pending` links still work.
  - **Order details (app and Direct Sale):** header, customer card (booking date, address, View map), services card with a Staff box per line (Fully staffed / Short / Not assigned), Payment card, Operations card, Sales agent, Notes and attachments. Manage and Replace staff drawers with day-off, busy and no-skill callouts. Staff loading state. "Leader" is now "Coordinator". Main's Telegram dispatch panel stays under Operations.
  - **Direct Sale create and edit:** card order follows Figma (Customer and address, Services, Date and time, Notes and attachments). Customer search, New customer and Add address dialogs, deposit 30/50/100 segmented control, six-field service row, amber "Waiting for deposit" banner (hidden hint for 100%).
  - **Dialogs:** Add service on balance, Service Add-On for app orders (never called Add service), cancel summary (Payment received, Refund, Balance, Keep order), reschedule dialog already matched Figma.
  - **Permissions:** actions are now hidden without the right permission: ORDER:UPDATE for edit, accept, complete, mark paid, cancel, refund, remark, receipts and balance link; ORDER:ASSIGN for staff changes; EXCHANGE_RATE:ADD and MARKETING_CUSTOMER:VIEW kept. Status and payment rules in `direct-sale-payment.ts` are unchanged.
  - **Fixes from review:** stale remark draft when switching orders, payment filter cleared by the type chip, "short of staff" shown when staff failed to load.
  - **Docs:** `docs/order-business-logic.md` and `docs/direct-sale-payment-flow.md` in bEasy-portal (flow charts of the order, details and Direct Sale rules).
- Not in this release, blocked by backend. Full spec for backend: `Docs/tracking/v2/backend-request/Order_Revamp_Backend_Request.md` (pending request item 19; reschedule rules stay in item 15):
  - **Reschedule reason, max 2 changes, "Changes used X of 2", 30-day limit** (item 15). Dialog works without them.
  - **Address name ("Save as": Home, Condo, Office)** in the Add address dialog, and the name in the address dropdown (spec item 2).
  - **Attachment on managed (payment link) Direct Sale create**: the create call answers `Unexpected field` (spec item 3). Legacy orders still upload a receipt.
  - **Cross-sell price on the order line:** the Aircon cross-sell line shows $3.50 but should be $8.50 because the API puts the $5 welcome coupon into that line's `discount` (order 0000233329, spec item 4). The portal math is right; the order total is right.
  - **Scope of work / area of work** on the service line and **Photos from customer**: no field in the order data (spec items 1 and 8).
  - **Deposit link "expired" state and Generate new link**: no Expired status and no endpoint; only the balance link can be created. "Link on {date}" wording waits for what `balanceAvailableAt` means.
  - **Price edited badge, App price, Reset price, separate Internal note, riel estimate and automatic exchange rate** on create: not in the data or payload (the editable Exchange Rate field stays).
  - **N previous orders, service-line code (SL-…), Timeline, open-issue and Telegram counts, customer-vs-internal note split, attachment file size** on details.
  - **Staff:** Telegram send and receipts, Reserve and send, Retry auto-assign, Coordinator change and Make coordinator, reason for a hand pick, last-change log, chat per person, travel buffer, equipment and transport checks.
  - **List:** status groups and row flags, Staff filter, Needs-action filters; `IN_PROGRESS` and `CANCELLED` status filters (returned nothing on dev); payment filter is ignored by the API for app orders (not filed).
  - **Assignment pending** state and its sentence (not in the data).
- Also in this release (closed from the earlier open list):
  - **ORDER:ADD** is now required for New Direct Sale (route and both "+" buttons); view-only users no longer reach the create page.
  - **Early time re-check:** on a new Direct Sale, if services change after a time is picked, the slot is re-checked at once; if it no longer fits, the date is cleared with "This time is no longer available". The server check on Create stays.
  - **Order details and list:** the Direct Sale "Add service" link and its hint are removed (changes still go through Edit); the line above the Telegram Operations card is gone; the order list has its back button again, top left.
  - **Direct Sale address field:** the "Select a customer first" hint under Address is removed.
  - **New customer dialog:** Phone number, Name and Referral source all show the red required marker, and Create is blocked in the form until a referral source is picked (before, only the API rejected it).
  - **Create page to Figma:** payment summary rows are now Services, Discount, Fees, VAT, Total. The red required markers stay (kept on purpose, Figma shows none).
- Needs:
  - Whether the backend rejects write calls without ORDER:UPDATE / ORDER:ASSIGN (not verified; no test with a view-only role in the browser).
  - Backend answer on when `balanceReady` turns true, and whether Complete is blocked with an unpaid balance (not confirmed).
  - Native check of the Khmer text; new list, filter and `directSaleLayout.*` strings are English only (km falls back).
  - Known, left for the owner: ABA payment check panels sit behind Payment details; Manage staff drawer reads page 1 of the cleaner list only; date filter can build from > to and drops silently.
  - G3 tests and G6 security for the permission guards were still open when this was written.

## 2026-11-02 · [Landing-Page] feat(i18n): Simplified Chinese (/zh) back, related guide cards, Khmer service pages synced
- Owner: SNLRLONG (Seavlong)
- Task: Tasks/6-deploy/Landing-Page-enable-chinese.md
- Branch/PR: main (committed directly, no PR)
- What changed: `/zh` is live again with the same services, claims and 12 guides as English and Khmer (no General Cleaning or Pest Control). Khmer `serviceDetail` entries 2, 3, 4, 6, 7, 9 now match the audit fixes. Washing Machine shows Top Load $25 and Front Load $35. Service pages show related guides as article cards aligned with the FAQ column.
- Needs: resubmit the sitemap in Search Console (zh URLs were redirected to km since 3 Oct). Chinese text and the Khmer service pages were reviewed by the owner.

## 2026-11-02 · [Landing-Page] fix(seo): 30 Oct audit fixes, retired services removed, related links, robots
- Owner: SNLRLONG (Seavlong)
- Task: Tasks/5-done/Landing-Page-seo-audit-fixes.md
- Branch/PR: main · b3fd4b6 (committed directly, no PR)
- What changed: General Cleaning and Pest Control are gone from pages, nav, footer, FAQ and the sitemap in Khmer and English, with 308 redirects for old URLs (`service-detail/1,2,5,7`, `services/general-cleaning`, `services/pest-control` to home, the General vs Deep article to Instant vs Deep). Office, Washing Machine, Upholstery, AC, Deep, Post-Renovation and Instant pages rewritten to the real scope. Service pages and guides now link to each other; `robots.txt` blocks `/purchase-coupon`, `/direct-sale-payment` and `/api/`. Also repaired a corrupted English `serviceDetail.json` that broke `yarn build`.
- Needs: Firebase deploy (`yarn deploy`), then resubmit the sitemap in Search Console and spot-check the redirects on `beasy.info`. Native read of the Khmer copy done by the owner.

## 2026-11-02 · [Portal] fix(pagination): sticky footer, equal spacing, clipped table corners, Customer tabs
- Owner: SNLRLONG (Seavlong)
- Task: Tasks/5-done/Portal-pagination-sticky-bottom.md
- Branch/PR: feat/pagination-sticky-bottom · #543
- What changed: The v2 list footer (`ListPageFooter`, used by `ListPage`, Cleaner and Partner payout) sticks to the bottom of the scrolling area with the same space above and below, no top line and iOS safe-area padding. The list card clips its contents so the table corners and last row line stay inside the frame. The v2 tab bar no longer collapses on large or zoomed screens, which had hidden the Customer Card View / List View tabs.
- Needs: nothing to deploy beyond the normal portal release. Not migrated: Order, Service bundle, Category add-on, Product and Product option still use the old pagination (owned by the RDS orders and catalog tasks).

## 2026-11-02 · [Portal] feat(chat): Source filter in a popover, no auto-selected chat
- Owner: SNLRLONG (Seavlong)
- Task: Tasks/5-done/Portal-chat-source-filter-popover.md
- Branch/PR: feat/chat-source-filter · #541
- What changed: The Chat list's Source dropdown moved behind a filter icon next to search; it opens a v2 popover with Source chips plus Clear and Apply. Opening Chat from another module now lands on the no-selection state instead of reselecting the last chat (the selection is cleared when leaving the page).
- Needs: nothing to deploy beyond the normal portal release.

## 2026-10-30 · [Portal] fix(journey): "Not tracked yet" for unmeasured OTP reasons, shared date range picker
- Owner: SNLRLONG (Seavlong)
- Task: Tasks/5-done/Portal-journey-registration-fixes.md
- Branch/PR: fix/journey-otp-reasons · #531 (squash `c0c4d9d2`)
- What changed: The Registration "Why the OTP was not verified" card lists a reason as "Not tracked yet" (no number, no bar) unless it has a count or `otpReasonAvailability` marks it measured; SMS delivery is the one backend does not record. The journey date filter now uses the shared v2 `DateRangePicker` (presets stay presets, 92-day limit and no future days kept).
- Needs: nothing to deploy beyond the normal portal release. Khmer native read of the new "Not tracked yet" string.

## 2026-10-23 · [Landing-Page] feat(media): week 2 SEO article instant vs deep cleaning
- Owner: SNLRLONG (Seavlong)
- Task: Tasks/5-done/Landing-Page-seo-article-week2.md
- Branch/PR: feat/seo-week2 · #119, #120
- What changed: Article id 11 "Instant Cleaning vs Deep Cleaning: Which Do You Need?" released in km and en, dated 23 Oct 2026, with two banners; slug `instant-cleaning-vs-deep-cleaning`.
- Needs: deploy on 23 Oct 2026
