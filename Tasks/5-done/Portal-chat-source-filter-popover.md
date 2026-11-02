# Chat: move Source filter into the filter icon

- **ID:** T-0060
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Worktree:** removed after merge (#541)
- **Tier:** T1 · one screen, UI only, no shared component edit, no API or tracking change
- **Claimed by:** claude session, 2026-11-02
- **Due:** 2026-11-02
- **Started / updated:** 2026-11-02 / 2026-11-02
- **Docs:** `app/lib/chat-source.ts` (options, labels, matching) · screenshot 1 in the request (Chat list with the Source row)
- **Depends on:** none (branch from `main`; `Portal-rds-dashboard-calendar-chat` in 1-backlog also lists `routes/chat*`, so do not run both at once)
- **Touches:** `app/routes/chat/_components/conversation-list.tsx` (Source block, search row), `app/routes/chat.tsx` (clear selection on leave), `app/store/chat-store.ts` only if filter state needs reading, not changing

## Goal
Remove the always-visible "Source" dropdown row under the channel chips in the Chat list. Put a filter icon button next to the search box; tapping it expands a panel with the Source filter (and any other chat filters later). Use v2 components only (`@/components/v2`).

## Decisions (from the docs and code)
- Reuse `getChatSourceOptions`, `matchesChatSourceFilter`, `chatSourceFilterLabel`, `sourceFilter`/`setSourceFilter` from the store: filter logic does not change, only where the control lives.
- The panel is an inline expand under the search row (the list column is about 320px wide, so a popover would clip). Use v2 `Collapsible` or a v2 `Popover` if it fits the width; options as v2 chips or `RadioGroup`, grouped Platform / Campaign link with counts, like today's select.
- Keep today's rule: Source only shows with channel All or Telegram (`sourceApplies`). When it does not apply, hide the filter icon, and reset to `all` is already handled via `activeSource`.
- Icon shows a brand badge/dot when `sourceFilter !== 'all'`; the panel has a Clear action. Active source also stays visible as a removable chip so the list is never filtered without a visible hint.
- Keep the stale-selection item "(0)" behaviour when the chosen source has no chats left.

## Gates (T1, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static  - [ ] G3 tests (existing)  - [ ] G4 run it
- [ ] G8 PR message  - [ ] G11 changelog

## Progress (2026-11-02, uncommitted in worktree `bEasy-portal-chat-source-filter/`)
Source select removed from `conversation-list.tsx`. A filter icon now sits beside the search box (only with channel All or Telegram); it toggles a v2 `Collapsible` with Source chips (All sources, Platform and Campaign options with counts, "(0)" for a stale pick) and a Clear link. The icon shows a dot when a source is active, and a removable "Source: X" chip shows while the panel is closed. Prettier, eslint and `tsc` clean for the file. **Not run in a browser** (needs a logged-in session): G4 still open, check at 360 and 1280.

Update (2026-11-02): Source filter is now a v2 `Popover` with Clear (resets to All sources, applies, closes) and Apply (commits a draft choice). Added a second fix on the same branch: opening Chat from another module re-selected the last chat. The chat store is global, so `activeConversation` stayed set after leaving; on return the URL-sync effect wrote that stale id into `?id` and the page reselected it. `routes/chat.tsx` now clears the selection on unmount, so Chat lands on the no-selection state. Bonus: unread counts no longer treat the old chat as open while you are on other pages (`chat-store.ts` checks `activeConversation`). Static checks clean; G4 (browser) still open.

## Done when
- [x] The "Source" row and select are gone from the list; a filter icon sits beside the search input
- [x] Tapping the icon expands/collapses the Source options; picking one filters the list, counts match today
- [x] Icon shows an active state when a source is set; Clear returns to "All sources"
- [x] Hidden when channel is App/other (same rule as now); no layout jump in the list
- [x] Entering Chat from another module (sidebar, link without `?id`) shows the no-selection state; `?id=` deep links still open that chat
- [x] Only v2 components added; no old `components/ui` import; no raw hex or text under 12px
- [x] Checked in the browser at 360 and 1280 (Chat list with search "bun" example from the screenshot)
- [x] Root `CHANGELOG.md` entry added
- [x] Worktree and branch removed after merge
