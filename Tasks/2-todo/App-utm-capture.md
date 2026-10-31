# App: capture utm_id, utm_content, utm_term

- **ID:** T-0034
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-utm-capture/` · **Branch:** `feat/utm-capture`
- **Tier:** T3 · tracking event params and API header the backend depends on
- **Claimed by:** none
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-utm-meta-tracking.md` · `Docs/tracking/v2/architecture/` (from T-0033) · `Docs/tracking/v2/backend-request/Campaign_Source_Request.md`
- **Depends on:** T-0033 (names agreed); backend must accept the fields (T-0035) before release
- **Touches:** `lib/app_tracking/appflyer_helper.dart`, `lib/analytics/attribution_store.dart`, `lib/analytics/analytics_helper.dart`, API header code, `test/`

## Goal
App keeps and sends the full UTM set from the template, not only campaign, source and medium.

## Scope
1. Parse `utm_id`, `utm_content`, `utm_term` in the AppsFlyer/OneLink/direct-link parsers.
2. Store them in `AttributionStore` with the same 7-day TTL.
3. Send on `app_opened`, registration event, Firebase user properties, and the account-creation header next to `X-Campaign-Code`.
4. Do not let an organic open overwrite a stored touch (existing rule).
5. Unit tests for parsing and storage. Dart-only: check whether a Shorebird patch is enough.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review (strict)  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] Parser and store tests pass
- [ ] Device test: OneLink with UTMs shows all fields in Firebase DebugView and the API call
- [ ] Backend deployed and accepts the fields before release
- [ ] Release plan agreed (Shorebird or store)
- [ ] Root `CHANGELOG.md` entry added (if code or shared docs changed)
