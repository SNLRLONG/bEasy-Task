# Journey v2 — Campaign source request

**To:** backend (bEasy-api) · **From:** portal / mobile · **Date:** 16 Oct 2026

## Request

When an `APP_OPENED` event has no `utm_source` / `campaign_code`, please use the customer's stored attribution touch (from `X-Campaign-Code` at account creation, or the latest touch within 7 days) to set the Journey source, instead of returning `other:Organic`.

## Why

- Your check confirmed the test customers' `APP_OPENED` rows have `NULL` campaign fields, so they show Organic.
- Campaign data is only known when the user arrives through a campaign link. Later opens (plain relaunch, push) are not link opens, so they do not carry it.
- The customer is still a Meta customer after the first open. Showing Organic for every later open loses the attribution you already store.

## How mobile works now

| Open type | `campaign_code` / `utm_source` / `utm_medium` on `app_opened` |
|---|---|
| Campaign link / OneLink open | Sent |
| Plain relaunch (`entry_type=organic`) | Not sent |
| Push open (`entry_type=push`) | Not sent |

- The app stores the last campaign touch for 7 days.
- It sends that touch to the API as `X-Campaign-Code` on account creation.
- Mobile will keep this behaviour, so the fix is needed on the backend side.

## Expected result

Customer with a Meta touch (`Meta` / `Instant_15`) → Journey source `Meta · Instant_15` on all their app opens inside the attribution window, not only link opens.
