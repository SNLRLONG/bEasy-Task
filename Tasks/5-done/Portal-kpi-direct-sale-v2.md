# Direct Sale v2: components and UI

- **ID:** T-0096
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-11-11
- **Started / updated:** 2026-11-04 / 2026-11-06
- **Schedule:** 2026-11-05 → 2026-11-11 (5 working days, order 2 of 12)
- **Worktree:** `bEasy/bEasy-portal-order-management-revamp/` (shared with T-0095 on the user's say-so, 4 Nov 2026; T-0095 is committed and pushed, work continues on the same branch) · **Branch:** `feat/order-management-revamp`
- **Tier:** T2 · UI and flow adjustments on existing order screens, no payment or refund logic changes (re-tier up if the diff touches them) (see `PIPELINE.md`)
- **Claimed by:** claude (Sonnet 5.5), 2026-11-04
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) page `1:2` section `11:343` Direct Sale v2 — Handoff; KPI plan: November 2026 (added by the user after the 12 master tasks); `Tasks/_plans/Portal-responsive-design-system.md`
- **Depends on:** Backend direct-sale endpoints (deposit, balance link, slot hold); confirm contract before building. `Portal-kpi-order-management-revamp` for shared order components.
- **Touches:** `app/routes/order/direct-sale/*`, `app/components/order/*`

## Goal
Build the Direct Sale create flow and order UI on v2 components: create (customer search, new customer, duplicate phone, address list/add/save, service type, other type, scope of work, deposit 50% minimum, date and time after services, time re-check), add service dialogs, and the Instant Cleaning disabled states.

## Decisions (user, 4 Nov 2026)
- Same worktree and branch as T-0095 (`feat/order-management-revamp`).
- Adjust UI and UX to Figma using the functionality and data that exist today. Anything blocked by backend is skipped, not built.

## Progress (4 Nov 2026)
Figma read through the Chrome extension (MCP seat rate-limited): create frames (empty, customer search, customer not found, new customer, duplicate phone, ready, date and time), `note: Rules`, `note: Dev notes`. Checked the running create page at 1440px on the dev backend.
- Already in the portal and matching Figma (no change): customer search with inline Add, phone-vs-name split, duplicate phone "Use existing", referral source, address list from the selected customer, service Category → Service → Type with auto-filled details, deposit 30/50/100 locked after create, ABA KHQR default, fees, VAT, deposit due and balance, sales agent locked in the header, Instant Cleaning excluded from add service.
- Done: create page layout to Figma order. Cards are now Customer & address, Services, Date & time (own card after Services, hint "Times depend on the job's length…"), Notes & attachments ("From the customer", "Staff see this on the job."). Service detail fields in one row (Bedrooms, Floor, Duration, Cleaners, Qty, Amount) for the main and extra service blocks. Payment: "Fees (optional)", "Balance on booking day (70%)". New strings under `directSaleLayout.*` in `en.json` only (km falls back to English for this block).
- Left out (no data or API): "Price edited" badge, App price and Reset price (catalog price is not kept in form state), Internal note (Ops only) as a separate field (one `note` on the payload), riel estimate and automatic exchange rate (the portal has an editable Exchange Rate field, kept), "Add a service ↑" lock link on the date field.
- Done (4 Nov, later): details states tied to the deposit and balance links. `pending · deposit link sent`, `confirmed · deposit paid` and `booking day · balance link sent` already match the payment card built in T-0095 (deposit paid with bank ref, balance Not due / Link sent, Copy link, Operations). The "Waiting for deposit" banner now follows Figma: amber, clock icon, "Waiting for deposit · $5.94 (30%)" and "The order starts once the customer pays. Staff are reserved once the deposit is paid. No reminders are sent." (hint hidden for 100% deposits). Checked on DS #0000166619 and on its edit page (same new layout, saved values load).
- Left out (no API or data): `pending · link expired` and "Generate new link" (no Expired state in the payment attempts and no endpoint to regenerate a deposit link; only the balance link can be created), "Link on {date}" wording for the balance row (unclear whether `balanceAvailableAt` is the link time or the due time), the "If none are free the order goes into Assignment pending" sentence (that state is not in the data).
- Not done: `create | date & time · time no longer fits (re-check)`. The form already clears the pending request and asks for another slot when the server answers `BOOKING_SLOT_UNAVAILABLE` or 400, with the existing "This time is no longer available" text. An early client check when services change after a date is picked is not built.
- Add-service dialog on Direct Sale details: main opens the editor (T-0095 decision), so no dialog.

## Second pass against Figma `create | empty` (4 Nov 2026, later)
Compared the dev create page with frame 59:4693 side by side (Brave, Claude in Chrome). Fixed: header crumb Direct sale / New order; VAT number (optional) with the Figma placeholder; Category / Service / Service type labels and Select category / service / type placeholders; the service defaults hint now shows in the empty state; Date & time field shows a lock and Add a service first while locked, with the Times depend on the job hint always shown; fee labels (Additional fee, Transport fee, Service fee); deposit 30/50/100 is a pill segmented control. Customer flow done (4 Nov, later): search input with the search icon, results with initials, name and phone, `N customers match`, Add new customer inside the dropdown (no Add link outside), no-match prompt with `Add 'number' as new customer`, and the New customer dialog (description, phone and name icons, name hint, referral optional). Left out of the duplicate-phone frame: Create separate customer (backend rejects a shared number) and the orders-count / last-order line (not in the data). Address: Add new address now sits at the bottom of the address dropdown, and the dialog follows Figma (title, For customer description, labels, filled-from-pin hints, editable Google Maps address filled from the starting pin, House no, Cancel and Save address). Save as (optional) is not built: the address API is not confirmed (request item 16). Search field restyled to the Figma bordered input with the icon on the right. Left out: the Drag the map pill (the pin is dragged, not the map), the blue pin on the map. Not changed yet: red *required markers (Figma shows none), payment summary rows (Figma: Services, Discount, Fees, VAT 10%, Total with KHR line), Exchange Rate field kept, Internal note still skipped, flat card style.

The attachment upload box for managed orders is not built: the create call answered Unexpected field on dev (request item 17). Legacy orders keep their payment upload. The Existing customer dialog is wider and its footer wraps instead of overflowing.

## Merge of main (4 Nov 2026)
Merged `origin/main` (18 commits: operations dispatch board, AC quantity and duration rules, early final payment link) into the branch with `--no-commit`; the merge commit is left to the user. Three conflicts: `en.json` (kept our `orderDetail` and `manageStaff` plus main's `staffing`), `service-details-v2.tsx` (default layout takes main's `JobStaffingControls`; the v2 layout keeps our `OrderStaffBox`; the old `AssignCleaner` helper is removed), `order-payment.tsx` (kept ours: no Assigned Cleaners card). Main's read-only `OrderTelegramDispatch` panel is now a card under Operations on the order page so that feature stays visible. The six-field service row was re-applied on top of main's `ServiceWorkingTime` note in both service blocks. Lint and tsc clean on touched files; dev server compiles and serves the order and create routes (200). Not viewed in the browser after the merge (extension disconnected).

## Notes
Scope cut 2 Nov 2026 to 5 days: the user confirmed this is UI and flow adjustment on screens that already exist, not a rebuild.
Estimate from about 40 current frames in section `11:343` (create about 20, add service 4, details states tied to deposit and balance links, disabled states). Superseded V3, V3.5 and V4 frames marked in the file are ignored. Cancel, reschedule and refund frames are counted in `Portal-kpi-cancellations`, not here. Overlaps `Portal-rds-orders` (direct-sale new/edit).

## Gates (T2, from `Tasks/PIPELINE.md`)
- [x] G1 scope  - [x] G2 static (prettier, eslint, tsc clean on touched files)  - [ ] G3 tests  - [x] G4 run it (dev, 4 Nov)
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog (G9 optional)
Moved to `4-testing/` 4 Nov 2026: user commits, pushes and merges to dev to test. Remaining: G3, G5, G7. Branch is shared with T-0095, which is T3; the shared diff carries its permission guards.

## Done when
- [ ] Screens match the Figma frames and states
- [ ] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Closed (6 Nov 2026)
Merged to `main` in PR #551 (feat/order-management-revamp). Changelog entry is in the root `CHANGELOG.md`. Backend-blocked items stay in `Docs/tracking/v2/backend-request/Order_Revamp_Backend_Request.md`.
