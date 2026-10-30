# RDS 5b: Orders module

- **ID:** T-0049
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Started / updated:** – / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-orders` off it
- **Tier:** T2
- **Claimed by:** none
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS pilot-coupon
- **Touches:** `routes/order/*`, `components/order/*`, `common/order-*`

## Goal
Migrate: order list, order detail components, direct-sale new/edit. Follow the pilot recipe. Check `git worktree list` for overlapping branches first; own worktree `bEasy-portal-rds-orders/`.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Assessment (2026-10-26): no migration needed
Read-only check in Chrome (390 and 1000px): `/order` is a full-screen workspace by design (own layout, back button, no sidebar). Phone: order list with filter chips, tap an order for the detail view (back arrow, status chips, stacked cards, payment banner, services). Desktop: list left, detail right. `/order/direct-sale/new` at 390px: stacked form with sticky action buttons. No horizontal overflow on any of them. Swapping them to v2 would put payment, MQTT assignment and status logic (and the 15 `instant-order-ui` tests) at risk for no responsive gain.
Left for the cleanup task only: the old shadcn tokens inside them (text sizes, colours), and the filter chip row on phones, which clips ("Selec…") but scrolls.
The edit route `/order/direct-sale/:orderId/edit` was not opened (needs an editable order); it shares the form component with `new`.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
