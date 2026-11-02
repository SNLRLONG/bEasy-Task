# App interaction inventory (generated 2 Nov 2026)

Companion to [App_Tracking_Coverage_Spec.md](App_Tracking_Coverage_Spec.md). One row per Dart file under `bEasy-app/lib/view` and `lib/chat_feature` that has at least one interaction handler or tracking call.

**How counted:** `Handlers` = occurrences of `onTap`, `onPressed`, `onChanged`, `onLongPress`, `onSubmitted`, `onRefresh`, `onPageChanged`, `onSelected`, `onDoubleTap`. `Tracking calls` = occurrences of `AnalyticsHelper.`, `FacebookHelper.`, `AppFlyerHelper.` in the file. Counts are a coverage signal, not exact element counts (one handler can sit on a shared widget, one tracking call can serve several handlers). Re-run the scan in the delivery task before coding; line-level handler lists come from that run.

**Totals:** 137 files, 554 handlers, 105 tracking calls.

| Area | File | Lines | Handlers | Tracking calls |
|---|---|---:|---:|---:|
| App shell | `view/app/app_screen.dart` | 534 | 2 | 1 |
| Auth | `view/screen/account/new_auth/new_signup_screen.dart` | 531 | 8 | 3 |
| Auth | `view/screen/account/new_auth/new_signin_screen.dart` | 369 | 7 | 4 |
| Auth | `view/screen/account/new_auth/new_verification_code_screen.dart` | 537 | 7 | 5 |
| Auth | `view/screen/account/customer_source_screen.dart` | 238 | 6 | 2 |
| Auth | `view/screen/account/signin_screen.dart` | 448 | 5 | 2 |
| Auth | `view/screen/account/signup_screen.dart` | 389 | 4 | 1 |
| Auth | `view/screen/account/verification_code_screen.dart` | 145 | 3 | 1 |
| Auth | `view/screen/account/set_password_screen.dart` | 125 | 2 | 0 |
| Auth | `view/screen/account/homepage_notification_screen.dart` | 203 | 1 | 3 |
| Auth | `view/screen/account/new_auth/login_telegram_button.dart` | 150 | 1 | 0 |
| Auth | `view/screen/account/welcome_screen.dart` | 143 | 0 | 1 |
| Chat | `chat_feature/views/message_details.dart` | 757 | 17 | 0 |
| Chat | `chat_feature/views/create_story_screen.dart` | 398 | 8 | 0 |
| Chat | `chat_feature/views/media_preview_screen.dart` | 201 | 5 | 0 |
| Chat | `chat_feature/components/chat_video_player.dart` | 173 | 4 | 0 |
| Chat | `chat_feature/components/custom_dialog_widget.dart` | 167 | 4 | 0 |
| Chat | `chat_feature/components/gallery_bottom_sheet.dart` | 315 | 4 | 0 |
| Chat | `chat_feature/views/message_view.dart` | 124 | 3 | 0 |
| Chat | `chat_feature/components/chat_media_card.dart` | 356 | 2 | 0 |
| Chat | `chat_feature/components/chat_error_banner.dart` | 52 | 1 | 0 |
| Chat | `chat_feature/components/chat_image_details.dart` | 44 | 1 | 0 |
| Chat | `chat_feature/components/chat_tile_widget.dart` | 294 | 1 | 0 |
| Checkout | `view/screen/home/category_building/confirm_and_pay_checkout_screen.dart` | 3601 | 20 | 5 |
| Checkout | `view/screen/home/category_building/apply_coupon_list_screen.dart` | 543 | 2 | 2 |
| Home | `view/screen/home/home_screen.dart` | 1402 | 17 | 11 |
| Home | `view/screen/home/see_all/see_more_screen.dart` | 287 | 5 | 3 |
| Home | `view/screen/home/components/home_latest_from_beasy_section.dart` | 401 | 3 | 3 |
| Home | `view/screen/home/components/home_service_search_screen.dart` | 200 | 3 | 0 |
| Home | `view/screen/home/components/services/home_services_section.dart` | 134 | 3 | 0 |
| Home | `view/screen/home/components/services/service_category_card.dart` | 344 | 2 | 0 |
| Home | `view/screen/home/components/telegram_widget.dart` | 51 | 2 | 1 |
| Home / banners | `view/screen/home/banner/banner_detail_screen.dart` | 1017 | 11 | 2 |
| Home / banners | `view/screen/home/banner/banner_detail_screen2.dart` | 187 | 3 | 0 |
| Home / banners | `view/screen/home/banner/bundle_android_pdf_screen.dart` | 116 | 2 | 0 |
| Home / banners | `view/screen/home/banner/image_details_screen.dart` | 43 | 1 | 0 |
| Launch | `view/screen/splash/no_internet_screen.dart` | 86 | 2 | 0 |
| Launch | `view/screen/splash/splash_ad_screen.dart` | 219 | 2 | 3 |
| Launch | `view/screen/splash/handle_server_error_screen.dart` | 49 | 1 | 0 |
| Notifications | `view/screen/notification/announcement_list_screen.dart` | 1107 | 11 | 0 |
| Notifications | `view/screen/notification/announcement_not_details.dart` | 232 | 1 | 0 |
| Orders | `view/screen/activity/order_detail_screen.dart` | 2038 | 23 | 8 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_reschedule_sheet.dart` | 974 | 12 | 0 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_cancel_sheet.dart` | 577 | 10 | 0 |
| Orders | `view/screen/activity/booking_screen.dart` | 494 | 5 | 2 |
| Orders | `view/screen/activity/leave_review.dart` | 370 | 5 | 2 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_bottom_actions.dart` | 113 | 5 | 0 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_extend_sheet.dart` | 383 | 4 | 0 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_reschedule_cancel_card.dart` | 123 | 4 | 0 |
| Orders | `view/screen/activity/cleaners_not_finished_dialog.dart` | 110 | 3 | 0 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_app_bar.dart` | 94 | 3 | 0 |
| Orders | `view/screen/activity/service-add-on/add_on_screen.dart` | 303 | 3 | 0 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_instant_progress_card.dart` | 343 | 1 | 0 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_services_card.dart` | 220 | 1 | 0 |
| Orders | `view/screen/activity/order_detail/widgets/order_detail_where_when_card.dart` | 98 | 1 | 0 |
| Orders | `view/screen/activity/order_detail_screen_v2.dart` | 571 | 1 | 10 |
| Orders | `view/screen/activity/payment_list.dart` | 207 | 1 | 0 |
| Orders | `view/screen/activity/review_done_screen.dart` | 60 | 1 | 0 |
| Orders | `view/screen/activity/service-add-on/main_service.dart` | 218 | 1 | 0 |
| Profile & settings | `view/screen/profile/new_location_screen.dart` | 559 | 16 | 0 |
| Profile & settings | `view/screen/profile/profile_info_screen.dart` | 788 | 14 | 1 |
| Profile & settings | `view/screen/profile/profile_screen.dart` | 514 | 13 | 3 |
| Profile & settings | `view/screen/profile/toturial_detail.dart` | 765 | 13 | 0 |
| Profile & settings | `view/screen/profile/location_screen.dart` | 601 | 8 | 1 |
| Profile & settings | `view/screen/profile/profile_help_center.dart` | 102 | 8 | 0 |
| Profile & settings | `view/screen/profile/change_avatar_profile_screen.dart` | 173 | 6 | 0 |
| Profile & settings | `view/screen/profile/profile_about_beasy.dart` | 79 | 6 | 0 |
| Profile & settings | `view/screen/profile/new_search_location.dart` | 138 | 5 | 0 |
| Profile & settings | `view/screen/profile/change_password_screen.dart` | 195 | 4 | 0 |
| Profile & settings | `view/screen/profile/toturial_center.dart` | 482 | 4 | 0 |
| Profile & settings | `view/screen/profile/language_screen.dart` | 67 | 1 | 0 |
| Profile & settings | `view/screen/profile/profile_coupon_screen.dart` | 229 | 1 | 2 |
| Service & booking | `view/screen/home/category_building/category_building_screen.dart` | 2546 | 17 | 11 |
| Service & booking | `view/screen/home/category_building/lucky_draw.dart` | 282 | 6 | 0 |
| Service & booking | `view/screen/home/category_building/verify_order_detail_screen.dart` | 568 | 6 | 0 |
| Service & booking | `view/screen/home/category_building/sub_category_screen.dart` | 518 | 5 | 0 |
| Service & booking | `view/screen/home/category_building/checkout_booking_schedule_sheet.dart` | 297 | 2 | 0 |
| Service & booking | `view/screen/home/category_building/checkout_cancellation_policy_sheet.dart` | 184 | 2 | 0 |
| Service & booking | `view/screen/home/category_building/lucky_draw_scroll.dart` | 286 | 2 | 0 |
| Service & booking | `view/screen/home/category_building/product_option_selector.dart` | 179 | 2 | 0 |
| Service & booking | `view/screen/home/category_building/sub_topup_point_screen.dart` | 76 | 2 | 0 |
| Service & booking | `view/screen/home/category_building/appointment_confirm_screen.dart` | 240 | 1 | 0 |
| Service & booking | `view/screen/home/category_building/appointment_status_screen.dart` | 291 | 1 | 0 |
| Service & booking | `view/screen/home/category_building/appointment_status_screen_v3.dart` | 95 | 1 | 0 |
| Service & booking | `view/screen/home/category_building/appointment_status_v2_screen.dart` | 200 | 1 | 0 |
| Service & booking | `view/screen/home/category_building/category_product_selector.dart` | 135 | 1 | 0 |
| Service & booking | `view/screen/home/category_building/task_info_detail_screen.dart` | 61 | 1 | 0 |
| Shared widgets | `view/custom/custom_show_dialog.dart` | 291 | 11 | 3 |
| Shared widgets | `view/custom/custom_button_widget.dart` | 323 | 8 | 0 |
| Shared widgets | `view/custom/custom_show_modal_pop_up.dart` | 470 | 6 | 0 |
| Shared widgets | `view/custom/custom_calendart.dart` | 343 | 5 | 0 |
| Shared widgets | `view/custom/custom_banner_list.dart` | 183 | 4 | 2 |
| Shared widgets | `view/custom/custom_contact_support_sheet.dart` | 155 | 4 | 1 |
| Shared widgets | `view/custom/custom_service_addon_group.dart` | 351 | 4 | 0 |
| Shared widgets | `view/custom/custom_app_bar.dart` | 111 | 3 | 0 |
| Shared widgets | `view/custom/custom_button_input_value_screen.dart` | 122 | 3 | 0 |
| Shared widgets | `view/custom/custom_button_sheet_update_app.dart` | 75 | 3 | 0 |
| Shared widgets | `view/custom/custom_country_code_sheet.dart` | 230 | 3 | 0 |
| Shared widgets | `view/custom/custom_instant_rooms_section.dart` | 479 | 3 | 0 |
| Shared widgets | `view/custom/custom_announcement_detail_sheet.dart` | 613 | 2 | 1 |
| Shared widgets | `view/custom/custom_apply_coupon_section.dart` | 126 | 2 | 0 |
| Shared widgets | `view/custom/custom_code_picker_widget.dart` | 311 | 2 | 0 |
| Shared widgets | `view/custom/custom_coupon_card.dart` | 284 | 2 | 0 |
| Shared widgets | `view/custom/custom_notification_offer_banner.dart` | 161 | 2 | 0 |
| Shared widgets | `view/custom/custom_payment_tile.dart` | 121 | 2 | 0 |
| Shared widgets | `view/custom/custom_promo_content_view.dart` | 392 | 2 | 0 |
| Shared widgets | `view/custom/custom_text_form_field_widget.dart` | 95 | 2 | 0 |
| Shared widgets | `view/custom/custom_textfield_widget.dart` | 130 | 2 | 0 |
| Shared widgets | `view/custom/custom_address_listtile.dart` | 68 | 1 | 0 |
| Shared widgets | `view/custom/custom_announcement_widget.dart` | 59 | 1 | 0 |
| Shared widgets | `view/custom/custom_argree_tc.dart` | 202 | 1 | 0 |
| Shared widgets | `view/custom/custom_button_qty.dart` | 74 | 1 | 0 |
| Shared widgets | `view/custom/custom_category_selection_alert_sheet.dart` | 248 | 1 | 0 |
| Shared widgets | `view/custom/custom_content_task_info_screen.dart` | 118 | 1 | 0 |
| Shared widgets | `view/custom/custom_fill_widget.dart` | 36 | 1 | 0 |
| Shared widgets | `view/custom/custom_instant_time_selector.dart` | 106 | 1 | 0 |
| Shared widgets | `view/custom/custom_payment_result_sheet.dart` | 264 | 1 | 0 |
| Shared widgets | `view/custom/custom_payment_sheet.dart` | 115 | 1 | 0 |
| Shared widgets | `view/custom/custom_schedule_date_selector.dart` | 140 | 1 | 0 |
| Shared widgets | `view/custom/custom_schedule_slot_chips.dart` | 157 | 1 | 0 |
| Shared widgets | `view/custom/custom_schedule_time_picker.dart` | 81 | 1 | 0 |
| Shared widgets | `view/custom/custom_schedule_week_calendar.dart` | 154 | 1 | 0 |
| Shared widgets | `view/custom/custom_selected_image.dart` | 94 | 1 | 0 |
| Shared widgets | `view/custom/custom_sheet_close_button.dart` | 33 | 1 | 0 |
| Shared widgets | `view/custom/custom_slidable_item.dart` | 221 | 1 | 0 |
| Shared widgets | `view/custom/custom_small_button.dart` | 55 | 1 | 0 |
| Shared widgets | `view/custom/custom_telegram_banner.dart` | 99 | 1 | 1 |
| Shared widgets | `view/custom/custom_unavailable_schedule_sheet.dart` | 132 | 1 | 0 |
| Top-up | `view/screen/home/top_up/top_up_screen.dart` | 995 | 24 | 2 |
| Top-up | `view/screen/home/top_up/top_up_history_screen.dart` | 441 | 6 | 1 |
| Top-up | `view/screen/home/top_up/top_up_aba_payment_screen.dart` | 113 | 1 | 0 |
| Top-up | `view/screen/home/top_up/top_up_status_screen.dart` | 118 | 1 | 1 |
| Web views | `view/screen/home/web_view/custom_web_app_screen.dart` | 153 | 2 | 0 |
| Web views | `view/screen/home/web_view/web_view_without_appbar.dart` | 126 | 2 | 0 |
| Web views | `view/screen/home/web_view/custom_webview_screen.dart` | 76 | 1 | 0 |
| Web views | `view/screen/home/web_view/payment_aba_webview_screen.dart` | 243 | 1 | 0 |
| Web views | `view/screen/home/web_view/slider_web_view_screen.dart` | 83 | 1 | 0 |

## By area

| Area | Files | Handlers | Tracking calls |
|---|---:|---:|---:|
| App shell | 1 | 2 | 1 |
| Auth | 11 | 44 | 22 |
| Chat | 11 | 50 | 0 |
| Checkout | 2 | 22 | 7 |
| Home | 7 | 35 | 18 |
| Home / banners | 4 | 17 | 2 |
| Launch | 3 | 5 | 3 |
| Notifications | 2 | 12 | 0 |
| Orders | 18 | 84 | 22 |
| Profile & settings | 13 | 99 | 7 |
| Service & booking | 15 | 50 | 11 |
| Shared widgets | 41 | 95 | 8 |
| Top-up | 4 | 32 | 4 |
| Web views | 5 | 7 | 0 |
