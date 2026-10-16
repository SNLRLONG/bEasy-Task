# Marketing foundations: routes, sidebar, shared form pieces

- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T2 · new routes, shared building blocks used by 5 areas
- **Claimed by:** none
- **Worktree:** `bEasy/bEasy-portal-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Docs:** Figma file `8bDJcg0dDmieddfLLjkloh`: Push `199:94165`, Promotions `199:115522`, Search & Content `199:125362` · plan: `Tasks/_plans/Portal-marketing-suite.md`
- **Depends on:** none
- **Touches:** `app/routes.ts`, `app/components/sidebar/*`, new `app/components/marketing/*`, `app/lib/marketing/*`, `app/locales/marketing/{en,km}/`

## Goal
Create the shared pieces every new marketing screen needs, as wrappers (no edits to shared components): menu entries and empty routes for Push, Promo codes, Package deals, Content library, Search insights; EN+KM paired field; wizard stepper (done = green tick, reopenable); status badge set; Phnom Penh time formatting from UTC; activity-log row component.

## Gates (T2 · new routes, shared building blocks used by 5 areas, from `Tasks/PIPELINE.md`)
- [ ] G1–G5 (new tests for the time helper)  - [ ] G7  - [ ] G8  - [ ] G11

## Done when
- [ ] Five routes render a placeholder page behind a permission check
- [ ] `<BilingualField>` requires both languages and shows per-language errors
- [ ] `<WizardStepper>`, `<StatusBadge>` use existing shadcn parts and Suntel tokens only
- [ ] UTC to GMT+7 helper unit-tested
- [ ] EN + Khmer strings in one file per area
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
