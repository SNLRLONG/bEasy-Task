# UTM template: fix generated URL

- **ID:** T-0032
- **Repo:** cross (xlsx, marketing)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T0 · spreadsheet formula only, no code
- **Claimed by:** none
- **Started / updated:** – / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-utm-meta-tracking.md` · `bEasy_UTM_Tracking_Template.xlsx`
- **Depends on:** none
- **Touches:** `bEasy_UTM_Tracking_Template.xlsx` (sheet UTM Register, column P)

## Goal
Generated UTM URLs open correctly and carry all UTM fields.

## Scope
1. Column P builds `<url>&utm_source=…`. First separator must be `?` (or `&` only if the destination already has `?`).
2. Include `utm_term` when filled.
3. Fix the 8 example rows; rows 7 and 8 have no destination URL yet.
4. Test 3 links on a device: UTMs reach the app (see T-0034/T-0037 for full test).
5. Tell marketing to regenerate every published link.

## Gates
Skipped G2–G7, G11: no code. G1 and G8 only.

## Done when
- [ ] Formula fixed and rows regenerated
- [ ] A test OneLink opens the app with UTMs intact
- [ ] Marketing told to replace old links
- [ ] Root `CHANGELOG.md` entry added (if code or shared docs changed)
