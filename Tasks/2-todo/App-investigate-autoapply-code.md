# Can the app apply a promo code from a push link?

- **ID:** T-0008
- **Repo:** bEasy-app
- **Owner:** Suntel
- **Tier:** T0 · investigation, no code change
- **Claimed by:** none
- **Worktree:** `bEasy/bEasy-app-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Docs:** Figma file `8bDJcg0dDmieddfLLjkloh`: Push notes `199:94168` (code carried into checkout, checked again there) · plan: `Tasks/_plans/Portal-marketing-suite.md`
- **Depends on:** none
- **Touches:** reads `lib/helper/deep_link_service.dart`, coupon/checkout flow; writes a short note

## Goal
Answer the Figma open question "Auto-apply code: does the app support it?" Find how a deep link or push payload could carry a code into checkout (apply, then re-validate), and what the app does if the code is no longer valid. Write the finding in `Docs/tracking/v2/architecture/` or as an inbox note to Portal-push tasks; do not change app code.

## Gates (T0 · investigation, no code change, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G8 message (docs/investigation only)

## Done when
- [ ] Written answer with file references
- [ ] List of app changes needed (if any) as backlog tasks
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
