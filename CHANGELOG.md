# Changelog

Merges to `main` across all projects, newest first. Prefix: `[Portal]` `[App]` `[Mini-App]` `[Landing-Page]` `[Backend]` `[Cross]`. Format and rules: `Tasks/README.md`.

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
