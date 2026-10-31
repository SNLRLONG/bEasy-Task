# Journey sheets: show the full phone

- **ID:** T-0004
- **Repo:** bEasy-portal (+ backend change)
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-portal-journey-phone/` · **Branch:** `feat/journey-phone-reveal`, from `origin/feat/journey-cms`
- **Tier:** T3 · personal data (phone), permissions, list export parity (see `PIPELINE.md`)
- **Claimed by:** none
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-30
- **Dropped:** 2026-10-30 · owner said skip (2026-10-20); personal data + backend change. Name display shipped separately.
- **Status:** backlog (owner, 2026-10-20: can skip for now). The customer-name display was split out and is required: `Portal-journey-registration-fixes` item 5.
- **Docs:** `Docs/tracking/v2/architecture/bEasy-tracking-architecture-guide.md` (lines 1072, 1481: masked by default, reveal needs a permission and an audit log) · `Docs/tracking/v2/requirements/Journey_Tracking_Requirements_Matrix.md` N2 · `Tasks/_plans/Cross-journey-tech-review.md` (DQ-01, UX-05)
- **Depends on:** backend item in `backend-request/backlog.md` (was 6.17); owner decision below
- **Touches:** `app/components/journey/list/{people-row,row-shell}.tsx`, `app/components/journey/customer/{customer-overview-sheet,key-facts,log-contact-dialog,customer-push-dialog,customer-actions}.tsx`, `app/types/journey.ts`, `app/lib/journey/list-search` helpers. Overlaps with `Portal-journey-registration-fixes` (registration row title)

## Why the phone is masked today
- By design: the architecture guide says phone and email are masked by default, and revealing them needs an explicit permission and is audit-logged. Phones saved at OTP request belong to people who may never register (matrix N2: consent wording and retention still open).
- Technically: the journey API returns only `phoneMasked`. The portal cannot show more without a backend change.
- Already inconsistent: the list **export** CSV contains real phones, and on 19 Oct the profile page was changed to show the full phone (owner decision, via the customer detail API). The lists and sheets stayed masked.

## Decision (owner, 2026-10-20)
Reveal the full phone in the journey sheets so staff can search and recognise customers by phone or name. Postponed. Recommended conditions (to confirm): show it only to users with `MARKETING_CUSTOMER:VIEW` (the gate the profile already uses); show it for registered customers, and for OTP-only people only after the N2 retention and consent wording is agreed; keep devices and guests as they are (no phone).

## Scope
1. **Backend:** add a full `phone` to people rows, customer summary and registration rows, returned only when the caller has the permission, and audit-logged (see `backlog.md`). Keep `phoneMasked` for users without it.
2. **Portal:** show `phone` when present, else `phoneMasked`, in: list rows, overview sheet, key facts, log-contact dialog, push dialog, registration rows. Format with `formatPhoneNumber`. No extra per-row API call (rejected: N+1 on customer detail).
3. **Name:** not here. Name display is in `Portal-journey-registration-fixes` item 5.
4. **Search:** `q` already matches name and phone on the backend. Check that `012…`, `+855 12…` and `85512…` all find the same person; if not, ask backend to normalise.
5. Do not show the phone in URLs, query params or analytics events.

## Gates (T3)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security review  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] A user without the permission still sees the masked phone (test with a restricted role)
- [ ] The same customer shows the same phone in the list, overview sheet, dialogs and profile
- [ ] Searching a full phone in 3 formats returns the person
- [ ] Backend confirms audit logging of reveals; retention for OTP-only phones answered
- [ ] No phone in logs, URLs or events
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
