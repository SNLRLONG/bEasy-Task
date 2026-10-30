# Journey: assisted conversion, finance amounts, consent, OTP instrumentation

- **ID:** T-0002
- **Repo:** cross (backend, bEasy-app, bEasy-portal)
- **Owner:** SNLRLONG (Seavlong)
- **Tier:** T3 · money, personal data, tracking contract
- **Claimed by:** none
- **Started / updated:** – / 2026-10-30
- **Dropped:** 2026-10-30 · not a task; items are deferred and listed in `backend-request/backlog.md`. Reopen as separate tasks when product decides.
- **Docs:** `Tasks/_plans/Cross-journey-tech-review.md` · handoff "Deliberately not fabricated" · `pending-request-backend.md` 3.2, 3.3
- **Depends on:** product decisions; backend data sources

## Items (each needs product input before it becomes a task)

| Item | Needs |
|---|---|
| Assisted conversion (T09, direct sale after app browsing) | A traceable contact/case-to-order identifier and an approved attribution window; show "assisted" separate from "app-native" |
| Finance summary (gross, promo, net paid, cancellation fee, refundable, refund to original method) | A reliable finance source; today refund amount and fee kept are `null` (`not_tracked`) |
| Consent and suppression for contacting leads | Consent / opt-in data, retention and access rules; guest and device-only records are not callable leads |
| OTP reasons (delivery, gateway, wrong code, rate limit) | New OTP request record and events (`pending-request-backend.md` 3.2) |
| Paid ad clicks, spend, CAC, ROAS | Meta/Google connectors: already tracked in `Cross-acquisition-ad-data` |

## Done when
- [ ] Each item has a decision (build, defer, drop) and, if build, its own task file
