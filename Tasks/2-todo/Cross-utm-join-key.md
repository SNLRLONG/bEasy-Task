# UTM join key: agree and document

- **ID:** T-0033
- **Repo:** cross
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T3 · tracking contract the backend depends on (headers, params)
- **Claimed by:** none
- **Due:** 2026-10-30
- **Started / updated:** – / 2026-10-21
- **Docs:** `Tasks/_plans/Cross-utm-meta-tracking.md` · `Docs/tracking/v2/architecture/`, `Docs/tracking/v2/backend-request/Campaign_Source_Request.md`
- **Depends on:** T-0032
- **Touches:** docs only: `Docs/tracking/v2/architecture/`, `api/Journey_Backend_API_Spec.md`

## Goal
Product, marketing and backend agree one key per UTM placement before code is written.

## Scope
1. Decide join key. Recommended: `utm_id` unique per placement; `utm_content` = creator/creative; `utm_campaign` = campaign grouping.
2. Decide header/param names the app will send (e.g. `X-Utm-Id`, `utm_content` on `app_opened`), values lowercase_snake_case, no personal data.
3. Decide how Acquisition groups: by campaign, then content, with `utm_id` as the row key.
4. Write it into architecture doc and API spec; link from T-0001.

## Gates
- [ ] G1 scope  - [ ] G7 docs sync  - [ ] G8 commit message  - [ ] G9 human review

## Done when
- [ ] Names and rules written in one architecture doc
- [ ] Backend and marketing confirmed
- [ ] T-0034 and T-0035 unblocked
- [ ] Root `CHANGELOG.md` entry added (if code or shared docs changed)
