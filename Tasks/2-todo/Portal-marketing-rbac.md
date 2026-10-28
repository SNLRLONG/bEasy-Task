# Permission modules for the new marketing areas

- **ID:** T-0013
- **Repo:** bEasy-portal
- **Owner:** Suntel
- **Tier:** T3 · permissions
- **Claimed by:** none
- **Worktree:** `bEasy/bEasy-portal-<short-name>/` (create when claimed) · **Branch:** `feat/<short-name>`
- **Docs:** Notes in Figma Push `199:94178`, Promotions `199:120403`, Search `199:125365` · plan: `Tasks/_plans/Portal-marketing-suite.md`
- **Depends on:** Portal-marketing-foundations-ui (route guards)
- **Touches:** `app/routes/rbac/*` data, `app/hooks/use-permission.ts` usage, new module keys

## Goal
Add permission modules for Push, Promo codes, Package deals, Content library, Search insights so the Figma rules hold: pause/resume/stop and activate/pause/end promo or package = Marketing manager + Super admin; global push limits = Super admin; full keyword access = Super admin; Content Lead can review content.

## Gates (T3 · permissions, from `Tasks/PIPELINE.md`)
- [ ] G1–G8  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog (G6 security review required)

## Done when
- [x] Each module appears in the role editor with view/add/update actions
- [x] Buttons and routes hidden or disabled by permission, tested with two roles
- [x] No change to existing modules
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge
