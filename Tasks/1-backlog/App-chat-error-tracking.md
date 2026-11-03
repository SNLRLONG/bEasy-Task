# App chat error tracking

- **ID:** T-0100
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-chat-error-tracking/` · **Branch:** `feat/chat-error-tracking`
- **Tier:** T2 · new `app_error` groups for chat (no flow change) (see `PIPELINE.md`)
- **Claimed by:** none
- **Started / updated:** – / 2026-11-03
- **Docs:** `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 I (Chat)
- **Depends on:** T-0067 merged (chat events and `ProfileTracking` exist)
- **Touches:** `lib/chat_feature/sockets/**`, `lib/chat_feature/controllers/message_controller.dart` (tracking lines only)

## Goal
Report chat failures that T-0067 could not reach because they live outside its scope: `app_error` for MQTT / socket connect and drop, and the media upload outcome (`success` / `error`), so `chat_message_sent` for media can be matched with a real result. Closed sets and ids only; no message text, no URLs, no file names.

## Gates
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (new + edge cases)  - [ ] G4 run it
- [ ] G5 code review  - [ ] G6 security  - [ ] G7 docs sync
- [ ] G8 PR message  - [ ] G9 human review  - [ ] G10 release plan  - [ ] G11 changelog

## Done when
- [ ] MQTT/socket connect failure and unexpected disconnect each send one `app_error` (`endpoint_group=chat`), not one per retry
- [ ] Media upload sends a success or error event; the Continue-tap `chat_message_sent` can be matched with it
- [ ] No message text, URL or file name in any param; tests assert the allow-list
- [ ] Tested on device (Firebase DebugView)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Notes
Added to the backlog by the owner on 3 Nov 2026 (not in this round).
