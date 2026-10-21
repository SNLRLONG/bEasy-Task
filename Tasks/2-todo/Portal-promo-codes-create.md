# Promo codes: create and edit

- **ID:** T-0014
- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T3 · discount rules and limits
- **Claimed by:** none
- **Worktree:** `bEasy/bEasy-portal-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Docs:** Figma file `8bDJcg0dDmieddfLLjkloh`: `199:120497` create, `199:120990` free service; notes `199:120403` · plan: `Tasks/_plans/Portal-marketing-suite.md`
- **Depends on:** Portal-promo-codes-list-detail
- **Touches:** `app/routes/promo-code/*`, `app/lib/promo-code/*`

## Goal
Create/edit form: code (unique ignoring case; locks once active, duplicate to change), name EN+KM, offer (percent, amount, or free service/add-on with service, variant, quantity, scope), limits (total, per customer), who can use (all, first-time = no completed booking, group, specific), dates, services. One per order, not with a package, app only.

## Gates (T3 · discount rules and limits, from `Tasks/PIPELINE.md`)
- [ ] G1–G8  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog (G6 security review required)

## Done when
- [ ] Validation matches the rules in the notes
- [ ] Active code text is read-only with a Duplicate action
- [ ] Free-service offer variant renders (frame `199:120990`)
- [ ] Unit tests for the rule validators
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
