# RDS 5e: Marketing module

- **ID:** T-0048
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-26 / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-marketing` off it
- **Tier:** T2
- **Claimed by:** claude session, 2026-10-26
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS pilot-coupon
- **Touches:** `routes/{marketing,payment-link,banner,popup-banner,push-notification}/*`, `otp`, `components/{journey,customer-journey}`

## Goal
Migrate: overview, customer journey, payment link, banner, popup banner, push notification, OTP, data export. Follow the pilot recipe. Check `git worktree list` for overlapping branches first; own worktree `bEasy-portal-rds-marketing/`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-26, uncommitted in worktree)
Migrated to v2: **Push Notification**, **Payment Link**, **OTP**, **Banner** (with drag-to-reorder handles on rows and cards) and **Popup Banner** (Listing + Tracking tabs) lists, their headers (`FilterBar`) and column card layouts. Every New / Export button, permission check and the delete confirmations are unchanged. `tests/payment-link-actions.test.mjs` mocks updated for the new imports (26/26 pass, all original assertions kept). `components/journey/funnel/journey-page-shell.tsx`: the three funnel tabs forced the page 56px wider than a phone; the strip now scrolls inside its own box (67 journey tests pass).
At 360px sideways scroll before/after: push-notification 1,553 to 0, payment-link 2,723 to 0, otp 480 to 0, banner 320 to 0, popup-banner 562 to 0.
Left alone on purpose: the create/edit forms (`push-notification/new` 845 lines, `payment-link/new`, `banner/new-banner` 813, `popup-banner/new-popup-banner` 689): audited at 360, no overflow, stacked layout; they need the uploader/combobox v2 fields and carry the most risk, so they are the last item here. `/overview` (intentional carousels inside widgets), `/data-export`.
Not tested: drag to reorder (writes), delete confirmations, send/resend of push notifications, payment link status refresh, the Customer Profile / Journey detail pages (still deferred from the customer-service task).

## Update 2 (2026-10-26): create/edit forms
Migrated the frame of **Popup Banner**, **Banner** and **Push Notification** create/edit forms to v2 (`FormPageHeader` sticky Save/Send, `Card*`, `Form*`, `FormInput`, `FormInputMultipleLanguages`, `SelectField`, `DatePicker`, `Checkbox`, `Uploader`; grids now 1 column on phones, 2 from sm, 3 or 4 from lg; push notification puts the side panel under the main column below lg). Logic, validation, payloads and language handling untouched. `tests/admin-languages.test.mjs` mocks updated for `@/components/v2`; 28/28 pass. Embedded old widgets kept as they are: rich-text editor, date-time picker, category/product and target-user selectors, form-custom-select.
Verified in Chrome at 390px (read-only, nothing saved): no overflow, sticky header, stacked fields on all three; the uploader shows `bad.gif is not a JPG, PNG, or PDF.` inline for an invalid file.
Not migrated on purpose: **Payment Link** form (creates payment links with amounts and coupons, heavy test mocks; already stacks) and the **Cleaner** create/edit form (already a responsive grid; tests exercise real expertise validation). Not tested: Save/Send on any form, real uploads.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
