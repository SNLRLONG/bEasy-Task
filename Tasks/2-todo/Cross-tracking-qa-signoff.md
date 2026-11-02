# Cross tracking qa signoff

- **ID:** T-0073
- **Repo:** cross
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** none
- **Tier:** T2 · verification across app, backend and portal (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-02
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §8, §9
- **Depends on:** T-0064, T-0065, T-0066, T-0067, T-0069, T-0070, T-0074, T-0076, T-0077, T-0079, T-0081, T-0082
- **Touches:** `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md`, `Docs/tracking/v2/README.md`, `Tasks/` notes

## Goal
Run the DebugView matrix for all slices on iOS and Android, compare one real staging batch against the registry, check legacy vs new daily counts, update docs, and decide legacy retirement with marketing (O-3).

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [ ] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [ ] Matrix covers every screen group, with pass/fail
- [ ] Unregistered ids = 0 on the staging batch
- [ ] Legacy vs new count difference ≤5% or explained
- [ ] Retirement decision for legacy events recorded; spec status set to final
