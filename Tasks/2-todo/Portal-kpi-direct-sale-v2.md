# Direct Sale v2: components and UI

- **ID:** T-0096
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG (Seavlong)
- **Due:** 2026-11-11
- **Started / updated:** – / 2026-11-02
- **Schedule:** 2026-11-05 → 2026-11-11 (5 working days, order 2 of 12)
- **Worktree:** `bEasy/bEasy-portal-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Tier:** T2 · UI and flow adjustments on existing order screens, no payment or refund logic changes (re-tier up if the diff touches them) (see `PIPELINE.md`)
- **Claimed by:** none
- **Docs:** Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) page `1:2` section `11:343` Direct Sale v2 — Handoff; KPI plan: November 2026 (added by the user after the 12 master tasks); `Tasks/_plans/Portal-responsive-design-system.md`
- **Depends on:** Backend direct-sale endpoints (deposit, balance link, slot hold); confirm contract before building. `Portal-kpi-order-management-revamp` for shared order components.
- **Touches:** `app/routes/order/direct-sale/*`, `app/components/order/*`

## Goal
Build the Direct Sale create flow and order UI on v2 components: create (customer search, new customer, duplicate phone, address list/add/save, service type, other type, scope of work, deposit 50% minimum, date and time after services, time re-check), add service dialogs, and the Instant Cleaning disabled states.

## Notes
Scope cut 2 Nov 2026 to 5 days: the user confirmed this is UI and flow adjustment on screens that already exist, not a rebuild.
Estimate from about 40 current frames in section `11:343` (create about 20, add service 4, details states tied to deposit and balance links, disabled states). Superseded V3, V3.5 and V4 frames marked in the file are ignored. Cancel, reschedule and refund frames are counted in `Portal-kpi-cancellations`, not here. Overlaps `Portal-rds-orders` (direct-sale new/edit).

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog (G9 optional)

## Done when
- [ ] Screens match the Figma frames and states
- [ ] Tested on dev backend
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
