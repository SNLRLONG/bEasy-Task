# App tracking account profile chat

- **ID:** T-0067
- **Repo:** bEasy-app
- **Owner:** SNLRLONG (Seavlong)
- **Worktree:** `bEasy/bEasy-app-tracking-account-profile-chat/` · **Branch:** `feat/tracking-account-profile-chat`
- **Tier:** T2 · new events using the generic contract from T-0063 (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-11-03
- **Started / updated:** 2026-11-03 / 2026-11-03
- **Docs:** `Tasks/_plans/Cross-tracking-coverage.md` · `Docs/tracking/v2/architecture/App_Tracking_Coverage_Spec.md` §4 B, H, I
- **Depends on:** T-0063, T-0074, T-0077 merged first (rebase)
- **Touches:** `lib/view/screen/{profile,message}/**` (not `profile/*location*.dart`, owned by T-0077; not `account/new_auth/*`, owned by T-0074), `lib/chat_feature/{views,components}/**`, `lib/controller/auth_controller.dart` (tracking lines only), `lib/controller/localization_controller.dart` (`selectLanguage`, `syncServerLanguage`, tracking lines only), `lib/view/screen/account/customer_source_screen.dart`, `lib/view/custom/{custom_code_picker_widget,custom_country_code_sheet,custom_contact_support_sheet,custom_argree_tc,custom_text*}.dart`

## Goal
Instrument spec §4 H (profile, language, help, tutorials; address and location moved to T-0077) and I (chat); auth and registration moved to T-0074: menu rows, forms, address management events, language `setting_changed`, permission results, chat events without any content.

## Gates
- [x] G1 scope  - [x] G2 static  - [x] G3 tests  - [ ] G4 run it
- [x] G5 code review  - [x] G7 docs sync  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [x] All 13 profile menu rows map to `element_id`s
- [ ] `address_*` events never carry street text (moved to T-0077, not built here)
- [x] Chat events carry message type only
- [x] Language switch from every picker (Home sheet, Profile) fires `setting_changed` with `source` (the silent server sync after sign-in does not: it is not a customer change) and sets user property `app_language`, also at app start
- [x] `CustomContactSupportSheet`: sheet view, Telegram, hotline, email each fire `support_contacted` with `source_screen`; Customer source CS button tracked
- [ ] DebugView matrix recorded
- [ ] Tested on dev backend / device (Firebase DebugView matrix recorded below)
- [ ] Root `CHANGELOG.md` entry added
- [ ] Worktree and branch removed after merge

## Progress (3 Nov 2026)

Collection only: no screen, navigation, request or return value changed. All calls are fire-and-forget through `ProfileTracking` (`lib/analytics/profile_tracking.dart`, new; ids in `ProfileIds`, not in `TrackingRegistry`). `fvm flutter analyze --no-pub` is clean; `fvm flutter test`: 226 pass, only `test/widget_test.dart` fails (known). New tests: `test/account_profile_chat_tracking_test.dart` (18 tests).

### Helper and contract changes
- `AnalyticsHelper.trackSettingChanged` gained optional `source`, `previousValue`; new `trackLanguageChanged`, `languageSourceFor`, `trackChatOpened`, `trackChatMessageSent`. `EventContract`: `chat_opened` needs `chat_kind`, `entry`; `chat_message_sent` needs `message_type`.
- Language codes are the app's real codes (closed set `en|km|vi|zh-cn|zh-tw`, else `other`), not only `km|en|zh`.
- `app_language` user property is set in `LocalizationController._setLocale`, so it covers app start (constructor), picker, server sync and the default fallback.

### Handler map
| Where | Handler | Event |
|---|---|---|
| `localization_controller.dart` `selectLanguage` | any picker (Home sheet, Profile) | `setting_changed` `setting_id=language`, `setting_value`, `previous_value`, `source=home_sheet|profile|other` (only when the language really changed) + user property `app_language` |
| `localization_controller.dart` `syncServerLanguage` | silent sync after sign-in (only when it calls the server) | nothing (not a customer change; `app_language` property unchanged) |
| `localization_controller.dart` `_applyAllowedLanguage` | language switched off in admin portal | `setting_changed` `source=default_fallback` |
| `localization_controller.dart` `_setLocale` | constructor, every change | user property `app_language` |
| `language_screen.dart` | option tap | `element_clicked language_option` (`item_id`=code, `menu_row`) + the `setting_changed` above (`source=profile`) |
| `profile_screen.dart` | header tap to profile info | `element_clicked menu_profile_info` |
| | My location row | `menu_my_location` |
| | My coupons row | `menu_my_coupons` |
| | Languages row | `menu_language` |
| | Share app row | `menu_share_app` |
| | Help center row | `menu_help_center` |
| | Contact supports row | `menu_contact_support` (`target=sheet`) |
| | About row | `menu_about` |
| | Logout row | `menu_logout` + `overlay_viewed logout_confirm` (dialog, manual: shared dialog takes no route settings) |
| | Logout confirm | `element_clicked logout_confirm` (`sheet_action`) |
| | Notification banner Not now / Turn on | already `notif_banner_not_now` / `notif_banner_open_settings` (T-0064, widget); added `permission_result push` (`pre_prompt`, new grant only) and user property `push_enabled` (on open and on resume) |
| | Telegram banner | already `join_telegram` (T-0064, widget) |
| `profile_help_center.dart` | FAQ / Terms / Privacy rows | `help_faq`, `help_terms`, `help_privacy` (`menu_row`) |
| | Email row | `help_email` + `support_contacted channel=email` |
| | `buildModalBottomSheet` (Profile contact support action sheet) | `overlay_viewed contact_support`; Telegram and Phone actions fire `support_contacted` (`source_screen=profile`, `element_id=contact_telegram|contact_phone`) |
| `profile_about_beasy.dart` | website, Facebook, Instagram, TikTok, Telegram, Rednote | `about_website|facebook|instagram|tiktok|telegram|rednote` (`target` = network; not `support_contacted`, these are social links) |
| `profile_info_screen.dart` | avatar tap | `change_avatar` + `overlay_viewed avatar_source` |
| | Take photo / Choose from album / Save image | `avatar_take_photo`, `avatar_choose_album`, `avatar_save_image` |
| | Gender field / option | `gender_field` + `overlay_viewed gender_picker`; `gender_option` (position only, never the value) |
| | Date of birth field / Cancel / Done | `dob_field` + `overlay_viewed dob_picker`; `dob_cancel`; `dob_done` (never the date) |
| | Save | `save_profile` + `form_submitted profile_info` (`field_count=5`; `success` when the controller replaced `userInfo`, else `error`) |
| | Delete account link / confirm | `delete_account` + `overlay_viewed delete_account_confirm`; `delete_account_confirm` |
| | `askPerMission` (on open) | `permission_result` photos and camera, `context=auto_open` |
| `profile_coupon_screen.dart` | pull to refresh | `refresh_coupons`; banner: same `permission_result push` + `push_enabled` as Profile |
| `toturial_center.dart` | category chip / video card | `tutorial_category` (position only), `tutorial_card` (`item_id`=video id) |
| `toturial_detail.dart` | play overlay / play-pause / fullscreen | `tutorial_play`, `tutorial_toggle_play`, `tutorial_fullscreen` (`item_id`=video id) |
| | related tile | `tutorial_related` (`item_id`) |
| | Was this helpful yes / no | `tutorial_helpful_yes`, `tutorial_helpful_no` (`item_id`) |
| `customer_source_screen.dart` | top-left back/skip | `skip_customer_source` (`back`) |
| | CS button | `contact_support` (`icon`, `target=sheet`) |
| | option row | `source_option` (`item_id` = short id: facebook, instagram, youtube, tiktok, nikki, influencer; never the typed text) |
| | Continue | `customer_source_continue` + `form_submitted customer_source` (`success` = the answer was submitted; the call itself is fire-and-forget) |
| `custom_contact_support_sheet.dart` | sheet opened (by `show`) | `overlay_viewed contact_support` (route settings; so Sign in, OTP, Sign up and Customer source get it too) |
| | Telegram / hotline / email | `support_contacted` (`telegram|phone|email`, `source_screen` = screen on top, `element_id` `contact_telegram|contact_phone|contact_email`); legacy `contact_us_telegram` kept |
| `message_view.dart` | pull to refresh / chat tile | `refresh_chats`; `chat_tile` (`item_id`=order id when an order chat, `target=order|support`) |
| `no_chat_found.dart` | Place an order | `place_order` |
| `message_details.dart` | room opened | `chat_opened` (`chat_kind=order|support`, `entry=chat_list|order_detail|other` from the previous screen, `order_id` only when a plain id) |
| | back / send / camera icon | `chat_back`; `send_message` + `chat_message_sent message_type=text`; `attach_camera` (+ `permission_result camera denied` when refused) |
| `create_story_screen.dart` | shutter / switch camera / close / gallery | `story_shutter`, `story_switch_camera`, `story_close`, `story_gallery` (+ `overlay_viewed story_gallery` on iOS); `permission_result photos` (`denied` or `limited`, `auto_open`) |
| `media_preview_screen.dart` | back / retake / continue | `media_back`, `media_retake`, `media_send` + `chat_message_sent message_type=image|video` |
| `gallery_bottom_sheet.dart` | pick | `gallery_pick` |
| `chat_media_card.dart` | open image/video | `media_open` (`target=image|video`) |

### Customer-service (CS) button: every screen
| Screen | File | Fires `element_clicked contact_support` today |
|---|---|---|
| Sign in | `account/new_auth/new_signin_screen.dart:257` | yes (T-0074; no `screen_name` passed, uses the screen on top) |
| OTP | `account/new_auth/new_verification_code_screen.dart:171` | yes (T-0074) |
| Sign up | `account/new_auth/new_signup_screen.dart:318` | yes (T-0074) |
| Customer source | `account/customer_source_screen.dart` | yes (this task) |
| Profile menu row "Contact supports" | `profile/profile_screen.dart` | yes as `menu_contact_support` (this task) |
| Order detail v2 (`OrderDetailAppBar`, `customer_servicesvg.svg`) | `activity/order_detail/widgets/order_detail_app_bar.dart:57`, handler `_contactSupport` in `order_detail_screen_v2.dart:199` | **no** (opens an order chat; T-0066 / owner) |
| Order detail legacy | `activity/order_detail_screen.dart:~770-785` | **no** (same, legacy screen) |
| See more (Home) | `home/see_all/see_more_screen.dart:130` | different id: `DiscoveryIds.contactSupport` with `target=telegram` + `support_contacted` (T-0064) |
Sheet channel taps (`support_contacted`) did **not** exist before; they are added here. Other Telegram support links without tracking: `home/category_building/category_building_screen.dart:998`, `home/category_building/confirm_and_pay_checkout_screen.dart:1384`.

Follow-ups for the owner (files outside this task's Touches, not edited): (1) order detail v2 and legacy CS button: `element_clicked contact_support` (`icon`, `target=order_chat`, `item_id`=order id) and `support_contacted channel=chat` with `order_id`; (2) the two Telegram links above: `support_contacted channel=telegram`; (3) sign in / OTP / sign up could pass `screen:` explicitly (works today through the screen on top).

### Intentionally untracked
- `change_avatar_profile_screen.dart`, `change_password_screen.dart` (Save is a no-op `() {}`), `payment_method_screen.dart`: no route reaches them (dead code); Profile rows `menu_payment_methods`, `menu_notifications`, `menu_terms`, `menu_chat` are not built (commented out).
- Back arrows and app-bar back buttons on profile screens (`screen_exited` covers leaving), pop-ups' Cancel buttons, `Get.snackbar` results.
- Text inputs (name, email, password, customer-source description, chat text field): focus and typing are never recorded (PII). Chat `onChanged` typing indicator: not tracked.
- Fullscreen tutorial player controls, chat video/image viewers (`chat_video_player.dart`, `chat_image_details.dart`, `chat_video_thumb.dart`), `typing_dot.dart`, `chat_tile_widget.dart` (tap tracked at call site), `chat_error_banner.dart` close, `custom_dialog_widget.dart` (generic confirm used by the camera permission prompt in `services/camera_service.dart`).
- Gallery "file too large" snackbar, album dropdown, refresh icon (invisible no-op).
- `custom_code_picker_widget.dart`, `custom_country_code_sheet.dart`, `custom_argree_tc.dart`, `custom_text*.dart` (in Touches): no event; country code and T&C ticks belong to the sign-in/sign-up flows tracked by T-0074, text fields are PII.
- `message_screen.dart` (Coming soon placeholder, no handlers; not used by the chat tab, `MessageView` is).

### Differences from the spec
- `setting_value` is the app's code (`en|km|vi|zh-cn|zh-tw`), not `km|en`; extra source `default_fallback`.
- Profile menu ids are `menu_*` with `menu_row` type as in §4 H, but `menu_terms`/`menu_notifications`/`menu_payment_methods`/`menu_chat` do not exist in the UI; Terms is under Help center (`help_terms`).
- `chat_opened`/`chat_message_sent` for media fire at the Continue tap, not after upload success (upload lives in `message_controller.dart`, outside Touches). Text messages fire when MQTT publish is called.
- `permission_result` fires only when the call could decide it (status before was still open; Android cannot tell never-asked from denied, so a repeated Android denial is not reported): profile info screen (photos, camera), chat camera icon, story photos.
- `app_error error_type=network` for MQTT/socket failures is **not** done: it lives in `chat_feature/sockets/*` and `message_controller.dart` (outside Touches).
- `form_submitted profile_info` outcome is inferred (`identical(userInfo)`), because `ProfileController.updateProfileInfo` returns nothing.
- `has_ordered` not set here: no screen in this task knows it (belongs to the order/booking slices). `location_permission` belongs to T-0077.
- Tutorial `screen_viewed + item_id` is not possible through the observer: `tutorial_card` click carries the video id instead.

### Open
- G4 run it on device (DebugView matrix: language from Home sheet and Profile, CS sheet channels, profile rows, profile info save, chat open/send/media), G7 docs sync (API spec/architecture, inventory counts), G8 PR message, G11 changelog.
- Owner decisions: (a) OK that `setting_value` uses the real language codes; (b) legacy `customer_source_continue` event puts the typed text in a parameter NAME (`trackName: 1`, `customer_source_screen.dart`): PII risk, recommend dropping it after the new events are live; (c) follow-ups for CS buttons outside Touches (above); (d) MQTT `app_error` and media upload outcome need a task touching `message_controller.dart`/`sockets`.

## Review round 1 (fresh agent, 3 Nov 2026)

Fixed
- Finding 1: `profile_info_screen.dart` reads the permission status before `request()` and reports photos/camera only when it was still open and the result changed (no more `granted auto_open` on every open).
- Finding 2: `message_details.dart` (camera icon) and `create_story_screen.dart` (`_loadLastImage`) read the status before the request and report the real status (granted, denied, denied_forever, limited) only when the call decided or changed it.
- Finding 3: `syncServerLanguage` no longer fires `setting_changed` (not a customer change); `server_sync` removed from `languageSources`; `default_fallback` kept; tests updated.
- Finding 7: mis-indented `URLLauncher.openApp` in `profile_about_beasy.dart` fixed.

Accepted
- Media `chat_message_sent` fires at the Continue tap, before upload; story type is never produced.
- `form_submitted profile_info` success is inferred from the userInfo replacement: can misreport on a concurrent refresh and sends no `error_code`.
- `customer_source` `form_submitted success` fires at the tap.
- `app_language` set in the constructor before Firebase init may be swallowed until the next locale change.
- Legacy `customer_source_continue` puts typed text in a parameter name and is unchanged: recommend a separate ticket.

Left
- Tests cover the helper layer, LocalizationController and ids; permission paths and chat screens are covered by the device matrix (G4).

After the fixes: `fvm flutter analyze --no-pub` clean; `fvm flutter test` 227 pass, only `test/widget_test.dart` fails (pre-existing).
