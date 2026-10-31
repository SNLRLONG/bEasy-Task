# UTM template: fix generated URL

- **ID:** T-0032
- **Repo:** cross (xlsx, marketing)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T0 · spreadsheet formula only, no code
- **Claimed by:** Claude session, 2026-10-21
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-30
- **Docs:** `Tasks/_plans/Cross-utm-meta-tracking.md` · `bEasy_UTM_Tracking_Template.xlsx`
- **Depends on:** none
- **Touches:** `bEasy_UTM_Tracking_Template.xlsx` (sheet UTM Register, column P)

## Goal
Generated UTM URLs open correctly and carry all UTM fields.

## Scope
1. Root cause: column P tested the destination with `SEARCH("?",G)`. In Excel `?` is a wildcard, so it always matched and the first separator was always `&`. Fixed by using `FIND("?",G)` on all rows (7 shared formulas, rows 6 to 205). Cached values of the 6 example rows refreshed; workbook set to recalculate on open.
2. `utm_term` was already included when filled.
3. Rows 7 and 8 (UTM-0007/8) still have no destination URL; marketing fills them.
4. Test 3 links on a device: UTMs reach the app (full test is T-0037).
5. Tell marketing to regenerate every published link.

## Gates
Skipped G2–G7, G11: no code. G1 and G8 only.

## Done when
- [x] Formula fixed and rows regenerated (21 Oct 2026; backup of the old file kept outside the repo)
- [ ] A test OneLink opens the app with UTMs intact
- [ ] Marketing told to replace old links
- [ ] Root `CHANGELOG.md` entry added (if code or shared docs changed)
