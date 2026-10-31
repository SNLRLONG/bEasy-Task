# RDS 2b: v2 form fields and pickers

- **ID:** T-0052
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-10-30
- **Started / updated:** 2026-10-26 / 2026-10-26
- **Worktree:** umbrella `bEasy/bEasy-portal-responsive-design-system/` (`feat/responsive-design-system`); task branch `feat/rds-form-fields` off it
- **Tier:** T2 · new shared components
- **Claimed by:** claude session, 2026-10-22
- **Docs:** `Tasks/_plans/Portal-responsive-design-system.md`, `bEasy-portal-responsive-design-system/docs/responsive-design-system.md`
- **Depends on:** RDS primitives
- **Touches:** `app/components/v2/ui/`, `app/components/v2/patterns/`

## Goal
Form field wrapper (label, helper, error at 12px+), date/range/time picker, combobox + multi-select (one of eight select variants), uploader, multi-language input, rich text wrapper. Keep react-hook-form and Zod usage identical.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [x] G4 run it (viewports 360/428/768/1024/1280/1536)
- [ ] G5 code review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-10-22, uncommitted in worktree)
Built: `patterns/form.tsx` (react-hook-form wrappers), `FormInput` (text/decimal/number/password, same props as old), `FormSelect`, `DatePicker` (+ `ui/calendar`, `ui/popover`; time picker kept; 44px days on phones, fits 360px), `FormInputMultipleLanguages` (+ dialog, en/km/vi/tw/cn), `StatCard`, `ui/card`, `layout/FormPageHeader` (sticky, same props as `CustomHeader`). Used end to end by the Coupon form (see pilot task).
Deferred until a page needs them: combobox / multi-select, uploader and image crop, date-range picker, rich text. These stay on the old components for now.
Checked: eslint, prettier clean, no new `tsc` errors, coupon tests 11/11.

## Update
2026-10-26: added `DateRangePicker` (same props as the old `DateRangePickerV2`; presets as chips, single range calendar, popover from md, bottom sheet below, 2 months from lg). Still not built: combobox / multi-select and uploader (needed by Product form, Popup Banner form and the customer filter sheet); build with the Marketing and Customer Service tasks.

## Update 3 (2026-10-26)
Built `Uploader` (same props as the old `Uploader`; inline errors instead of browser `alert()`, keyboard accessible, validation in `upload-validation.ts` with 5 tests). Not built: combobox / multi-select. After auditing every page, nothing remaining needs it: the only users are the hidden Product form and the heavy embedded selectors (`category-and-product-selection-v2`, `target-users-form-v2`, customer filter sheet) which already work on a phone. Build it when one of those is migrated.

## Done when
- [ ] Only `app/components/v2/`, new styles and the listed pages changed; no old component edited
- [ ] No raw hex, palette class, text under 12px or arbitrary radius in new code
- [ ] typecheck, lint and build pass; no horizontal page scroll at any tested width
