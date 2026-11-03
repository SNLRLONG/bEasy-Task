# App tracking home discovery

- **ID:** T-0064
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-home-discovery/` · **Branch:** `feat/tracking-home-discovery`
- **Tier:** T2 · new events using the generic contract from T-0063; no new names (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-03
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 A, C, G
- **Depends on:** T-0063; T-0074 merged first (it adds one `entry:` line in `home_screen.dart`, `app_screen.dart`, `banner_detail_screen.dart`, `helper.dart`)
- **Touches:** `lib/view/screen/{splash,home,notification}/**`, `lib/view/custom/{custom_banner_list,custom_telegram_banner,custom_announcement_*,custom_notification_*,custom_promo_*,custom_button_sheet_update_app}.dart`, `lib/util/local_notifications_service.dart`, `lib/view/app/app_screen.dart`

## Goal
Home language icon fires `element_clicked` `language_picker` (the choice itself is tracked in T-0067's controller hook). Instrument spec §4 A (launch), C (home and discovery) and G (notifications): element clicks, home search events, banner clicks, notification inbox/detail actions, tab navigation, dual-fire legacy events with `screen_name`.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [ ] G4 run it
- [ ] G5 code review  - [x] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [x] Every live handler in the listed files is mapped to an `element_id` or listed as intentionally untracked with a reason
- [ ] Search fires `search_performed`/`search_result_clicked` (debounced, ≤30 chars)
- [x] Legacy events still fire
- [ ] DebugView matrix (screen, action, event, params) recorded in this file
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress (3 Nov 2026)

Branch `feat/tracking-home-discovery` (from `feat/registration-tracking`). Nothing committed. `fvm flutter analyze --no-pub`: no issues. `fvm flutter test`: all pass except `test/widget_test.dart` (fails on main too). New: `lib/analytics/discovery_tracking.dart` (element and overlay ids, fire-and-forget wrappers: `click`, `overlay`, `support`, `permission`, `error`) and `test/home_discovery_tracking_test.dart` (19 tests: id pattern and uniqueness, 1-based position, `item_id` allow-list, closed sets, search query rules, allowed/forbidden keys, contract validation, never throws).

Rules kept: collection only, no flow change (the only structural edits are closures wrapping an existing callback, and `onSubmitted` on the search field which only reports). Every call is `unawaited` inside try/catch. `item_id` accepts only `[A-Za-z0-9_-]{1,40}` (entity ids, server codes), so a label can never be sent. `position` is 1-based like search `rank`. Coupon codes, names, URLs and notification text are never sent. Legacy events (`service_click`, `bcombos_click`, `language_selection`, `contact_us_telegram`, `join_telegram_channel`, `turn_on_notifications_click`, `tracking_pages`, `banner_*`, `splash_*`, `register_button_on_home`) are untouched, so they still dual-fire.

### Handler map

`screen_name` is the screen on top (observer) unless noted. "overlay" = `overlay_viewed` (sheets/popups opened with `overlaySettings`; auto/push popups and dialogs opened by shared helpers use `DiscoveryTracking.overlay` with the right `trigger`).

| Where | Handler | Event |
|---|---|---|
| splash_ad_screen | skip tap | legacy `splash_skip_button` + `element_clicked splash_ad_skip` (screen `splash_ad`) |
| splash_ad_screen | CTA tap | legacy `splash_cta_button` + `element_clicked splash_ad_cta` |
| splash_screen | no internet branch | `app_error` network / `no_internet` (screen is `splash`, the failure happens there) |
| splash_screen | user-info failure branch | `app_error` api / `user_info` |
| no_internet_screen | Retry | `element_clicked retry_connection` (screen `no_internet`) |
| no_internet_screen | dialog OK | untracked (closes a dialog) |
| handle_server_error_screen | Retry | `element_clicked retry_server` (screen `server_error`) |
| custom_button_sheet_update_app | shown (once per variant per run) | `overlay_viewed force_update` / `optional_update` (dialog, auto) + `app_error` update / `app_version` (`fatal`=1 when forced) |
| custom_button_sheet_update_app | Update now (both variants), Maybe later | `element_clicked update_now` / `update_later` (screen `force_update`) |
| home_screen | language icon | `element_clicked language_picker` + legacy `language_selection`; sheet = overlay `language_picker`; the choice itself is T-0067's controller hook, so the option rows are untracked here |
| home_screen | pull to refresh | `element_clicked refresh_home` |
| home_screen | service tile (`_openCategory`) | `element_clicked service_card` (`item_id`, `position`, `section=services`, `target=coming_soon` when so) + legacy `service_click`; coming soon dialog = `overlay_viewed coming_soon` (dialog) |
| home_screen | See more tile | `element_clicked see_all_services` |
| home_screen | services error Retry | `element_clicked retry_services` |
| home_screen | home API failure (`hasHomeError` after `getHomePage`) | `app_error` api / `home_page` (on load, refresh and retry) |
| home_screen | bCombos card | `element_clicked bcombo_card` (`item_id`, `position`) + legacy `bcombos_click` |
| home_screen | guest sign-in card | `element_clicked sign_in_card` (`registration_started` stays with T-0074's `entry:`) |
| home_screen | floating promo button (guest and member closures) | `element_clicked promo_fab` (`item_id` popup id); popup = overlay `promo_popup` (user) |
| home_screen | session promo popup, new-user popup, Ang Pao dialog | `overlay_viewed promo_popup` / `lucky_draw` with `trigger=auto` |
| home_screen | the two greeting `InkWell`s | untracked: empty handlers (commented-out debug showcase) |
| home_screen | FAB drag | untracked: layout gesture |
| home_services_section / service_category_card | tile, See more card, Retry button | callbacks only; tracked by the caller above |
| home_latest_from_beasy_section | promotion card | `element_clicked latest_promo_card` (`item_id`, `position`) + legacy `bcombos_click` |
| home_latest_from_beasy_section | Turn on notifications | `element_clicked turn_on_notifications` + `permission_result push` (`pre_prompt`) + legacy |
| home_latest_from_beasy_section | Join Telegram | `element_clicked join_telegram` + legacy `join_telegram_channel` |
| telegram_widget (not mounted anywhere today) | chat button, close | `element_clicked telegram_float_button` + `support_contacted telegram` + legacy; `close_telegram_float` |
| home_service_search_screen (no navigation reaches it today) | typing | `search_performed` after 600 ms, or at once on submit; same text once; `result_count`, can be 0 |
| home_service_search_screen | result row | `search_result_clicked` (`rank` 1-based, `query`, `service_id`); coming soon = overlay `coming_soon` |
| home_service_search_screen | back | untracked: observer records `screen_exited` and `screen_viewed back` |
| see_more_screen | CS icon | `element_clicked contact_support` (target telegram) + `support_contacted telegram` + legacy |
| see_more_screen | copy code (outer and inner tap) | `element_clicked copy_promo_code` (no code) |
| see_more_screen | service icon | `element_clicked service_card` (`item_id` category id, `position`, `section=explore`) |
| banner (custom_banner_list) | carousel slide tap | `element_clicked banner_card` (`item_id`, `position`, `section`=placement) + existing `banner_clicked` |
| banner_detail_screen | main button | `element_clicked banner_cta` (`section` bcombo/announcement/banner, `target`=server action type); guest dialog = overlay `guest_sign_in_prompt`; Continue = `element_clicked guest_sign_in_continue` + legacy |
| banner_detail_screen | coupon card select | `element_clicked banner_coupon_select` (`item_id`) |
| banner_detail_screen | terms checkbox | `element_clicked banner_terms_check` |
| banner_detail_screen | payment sheet / ABA sheet / receipt source popup | overlays `banner_payment_methods`, `banner_aba_transfer`, `receipt_source` |
| banner_detail_screen | payment method row, close X, upload, submit, take photo, choose album, cancel | `banner_payment_method` (`item_id`=method code), `banner_sheet_close`, `receipt_upload`, `receipt_submit`, `receipt_take_photo`, `receipt_choose_album`, `receipt_cancel` |
| banner_detail_screen | hero image tap | untracked: builds an `ImageDetailScreen` and discards it (no navigation), nothing happens for the customer |
| banner_detail_screen | back, tap outside payment sheet, `_isLoading` guards | untracked: navigation and dismissals |
| banner_detail_screen2 | image tap, body link, CTA button | `banner_image_zoom`, `banner_cta` (`section` body_link / cta_button), the same taps still raise `banner_clicked` through `Helper.handleBannerCtaTap` |
| bundle_android_pdf_screen, image_details_screen | back, page change | untracked: navigation and scrolling (the PDF screen is not reachable today) |
| announcement_list_screen | bottom tab Notifications opened (app_screen) | `notification_inbox_viewed` (`unread_count`) |
| announcement_list_screen | two tab labels | `element_clicked tab_notifications_list` / `tab_announcements` |
| announcement_list_screen | swipe between tabs (`_onPageChanged`) | untracked: only taps are clicks |
| announcement_list_screen | mark all as read icon | `element_clicked mark_all_read`; sheet = overlay `mark_all_read`; `mark_all_read_confirm` / `mark_all_read_cancel` |
| announcement_list_screen | pull to refresh x2 | `refresh_notifications`, `refresh_announcements` |
| announcement_list_screen | permission banner Not now / Open settings | `notif_banner_not_now`, `notif_banner_open_settings` + `permission_result push` |
| announcement_list_screen | notification card | `element_clicked notification_card` (`item_id`, `target`=server type) |
| announcement_list_screen | announcement card | `element_clicked announcement_card` + `notification_detail_viewed` (`source=inbox`) |
| custom_announcement_detail_sheet | sheet shown | overlay `announcement_detail` |
| custom_announcement_detail_sheet | Close, CTA | `announcement_close`, `announcement_cta` (`target`=server type, `item_id`); Telegram CTA also `support_contacted telegram`; Ang Pao CTA = overlay `lucky_draw` |
| announcement_not_details (not pushed anywhere today) | CTA button | `element_clicked announcement_cta` (`section=detail_screen`); back untracked |
| custom_notification_offer_banner | Not now, Open settings | `notif_banner_not_now`, `notif_banner_open_settings` (`section=offer_banner`, all callers) |
| custom_telegram_banner | tap | `element_clicked join_telegram` (`section=telegram_banner`, all callers) + legacy when default |
| custom_promo_content_view | body tap, close | `element_clicked promo_popup` (`item_id` popup id), `promo_close` |
| custom_promo_overlay, custom_announcement_widget, custom_notification_card | no tap handler (pass-through refresh only) | untracked |
| local_notifications_service | push tap routing | `push_opened` already fires first (untouched); Ang Pao case = overlay `lucky_draw` `trigger=push`; Telegram/Support case adds `support_contacted telegram` (`source_screen=push`); announcement case already fires `notification_detail_viewed` |
| app_screen | bottom tab tap | `element_clicked tab_home / tab_bookings / tab_notifications / tab_chat / tab_profile` (`element_type=tab`, from the tab being left) + legacy `tracking_pages`; `screen_viewed entry_type=tab` already comes from `ScreenTracker.enterTabIndex` |
| app_screen | guest tap on a locked tab | `element_clicked tab_locked_guest` (`target`=`tab_*`) before the T-0074 sign-in push |

### Differences from the spec (§4 A, C, G)

- No `search_bar`, `notification_bell`, `top_up_card`, `history_card`, `greeting_avatar` on Home: those widgets are commented out or removed in the current `home_screen.dart`; the only home search UI is `HomeServiceSearchScreen`, which nothing navigates to. Wired anyway, so it reports the day the entry returns.
- `notification_deleted` not built: no delete or swipe-to-delete handler exists. `notification_cta_clicked` not built: new event name (this task adds none) and the inbox items carry no `send_id`; CTA taps go through `element_clicked announcement_cta`. `mark_as_read` per item is not a separate control (a card tap marks it read, covered by `notification_card`).
- `trigger` on `overlay_viewed`: the foundation `overlaySettings` cannot carry it, so auto and push popups call `DiscoveryTracking.overlay(trigger: auto|push)` instead of `overlaySettings` (no duplicate with the observer). User-opened sheets and popups use `overlaySettings`. Dialogs opened through `CustomShowDialog` / `CustomDialog` / `CustomCategorySelectionAlertSheet` (outside `Touches`) get a direct call (`coming_soon`, `guest_sign_in_prompt`); the category-selection sheet itself is left to its own owner.
- `banner_pdf_open`: the PDF screen is only referenced in commented code. `banner_share`: no share control exists. `banner_image_zoom` only on `BannerDetailComponent` (the other screen's tap does nothing).
- `custom_notification_offer_banner` and `custom_telegram_banner` are inline cards, not popups: no `overlay_viewed` (it would fire on every rebuild); clicks are tracked.
- `app_error` for no-internet and server-error is raised in `splash_screen.dart`, so `screen_name` is `splash`, not `no_internet` / `server_error`. `trackAppError` reads the screen on top and the new screen is not on top yet.
- Element ids live in `DiscoveryIds` (constants) rather than `TrackingRegistry`, to avoid merge conflicts with parallel tracking branches; the registry has no element table today. They still need adding to the backend `ref_app_element` list.
- Debounce and the `search_performed` call sit in the `HomeServiceSearchScreen` state (not unit tested: the screen needs the full GetX graph and is unreachable); the event shape, query rules and 0-result case are tested through `AnalyticsHelper`.

### Still open

- [ ] Done-when "Search fires ... (debounced, <=30 chars)" stays unticked until G4: needs a device with the search screen reachable (temporary entry) to prove the 600 ms debounce.
- [ ] G4 device matrix (DebugView: screen, action, event, params) for the rows above: splash ad, home taps, banners, notification inbox/sheet, tab bar, update dialog (needs a forced version), no-internet and server-error screens
- [x] G5 code review  - [x] G7 docs sync (API spec / architecture: new element ids, `app_error` groups `home_page`, `user_info`, `no_internet`, `app_version`; backend `ref_app_element` rows)  - [ ] G8 PR message  - [ ] G11 changelog
- Decision for owner: should `retry_*`, `refresh_*` and tab clicks stay (more volume) or should only discovery taps remain? Everything is one-line removable per call site.

## Review round 1 (fresh agent, 3 Nov 2026)
Fixed: (1, medium) a result tapped inside the 600 ms debounce now reports `search_performed` first and on the search screen (it used to fire later with the wrong `screen_name` and after the click); typing the same text again after clearing it is reported as a new search. (2, medium) `permission_result push` was sent as `denied` when the app only opened Settings: it is now sent only for a new grant (not granted before, granted after).
Accepted: (3) `notification_inbox_viewed.unread_count` is the badge value the customer saw when tapping the tab (the fetch runs in parallel); opening the inbox by push or deep link does not fire it. (4) Search text: `_searchQuery` drops emails and long digit runs only; typed names or addresses still go out lower-cased and cut to 30 characters. This is the foundation's rule; decide before release whether to bucket or hash the query. (5) The optional update prompt is reported as `app_error type=update fatal=0`; it is a prompt, not an error, and a later reappearance in the same run is not reported. (6) `banner_card` click and the legacy banner event both fire on one tap by design. (8) `app_error home_page` covers the home screen loads only (not `helper.dart` / `deep_link_service.dart` calls); the no-internet overlay in `my_app.dart` and photo/camera permissions are not wired. (7) The update dialog's "maybe later" button is now always enabled; `my_app.dart` always passes the handler, so nothing changes today.
Merge note: tracking lines sit next to T-0074's `entry:` lines in `app_screen.dart`, `home_screen.dart` and `banner_detail_screen.dart`; expect adjacent-hunk conflicts only.
Left: tests cover helpers and ids, not the debounce or once-per-occurrence on widgets; the device matrix covers those.
After fixes: analyze clean; 172 tests pass (only the old `widget_test.dart` fails).

## Status (3 Nov 2026)
Committed as ebbebb01 on `feat/tracking-home-discovery`. **Not merged to `main`: the owner holds all merges until the whole app side is done; branches are merged into `feat/tracking-integration` for the combined device check (G4).** Open: G4, G6, G8-G11.
