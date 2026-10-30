# Plan: Meta app events + UTM tracking in Journey > Acquisition

**Updated:** 21 Oct 2026 · Owner: SNLRLONG (Seavlong) · Related: T-0001 (`Cross-acquisition-ad-data`)

## Goal
1. Meta app events are verified so marketing can build audiences and retarget.
2. Every UTM link in `bEasy_UTM_Tracking_Template.xlsx` shows up in Journey > Acquisition, per placement (IG bio, FB page, TikTok bio, DMs, KOCs, paid ads).

## Flow
| Step | State |
|---|---|
| Marketing posts a OneLink URL with UTMs (template) | **fixed (T-0032)**: first separator was `&` instead of `?` |
| OneLink opens/installs the app; `appflyer_helper.dart` parses `utm_source/medium/campaign` | exists |
| App keeps last touch 7 days (`attribution_store.dart`): code, source, medium | **change**: add `utm_id`, `utm_content`, `utm_term` |
| App sends them on `app_opened`, registration, Firebase user props, `X-Campaign-Code` header | **change** |
| Backend stores the touch, maps `APP_OPENED` without campaign to stored touch | exists (request) / **change** (new fields) |
| Portal Acquisition campaign table and filters | **change**: show UTM ID and content |
| Meta SDK logs ViewContent, SelectService, InitiateCheckout, AddPaymentInfo, ViewBooking, CompleteRegistration, Purchase | exists, **verify** |

## Gaps and decisions
- **Template URL bug.** Links came out as `…/xxx&utm_source=…` instead of `…/xxx?utm_source=…`. Cause: `SEARCH("?")` treats `?` as a wildcard. Fixed with `FIND` (T-0032).
- **`utm_campaign` is the only campaign key.** Template uses `always_on` for IG, FB and TikTok profiles, so they collide. Decision: `utm_id` is the unique per-placement key (T-0033, recommended).
- **Meta event names stay standard** (not `ServiceViewed`/`BookingStarted` from the ChatGPT list). Custom names break audiences and optimisation.
- **"Meta credited zero installs"** is likely AppsFlyer/Meta partner config, not proof of no installs (T-0031).
- **Messages (DM) links** track clicks only; message counts come from Meta/Telegram inbox reports (template rule).
- **Mini-app and website bookings** are not covered by OneLink capture. Out of scope here; ask marketing which links go where.
- No UTM may contain personal data (template rule).

## Open questions (recommendation)
1. Join key: `utm_id` (recommended) vs `utm_campaign + utm_content`.
2. Does the backend need `utm_term`? Recommend capture, store, do not show.

## Task breakdown
| ID | Task | Repo | Depends on | Tier |
|---|---|---|---|---|
| T-0031 | Cross-meta-app-events-setup | cross | none | T2 |
| T-0032 | Cross-utm-template-fix | cross | none | T0 |
| T-0033 | Cross-utm-join-key | cross | T-0032 | T3 |
| T-0034 | App-utm-capture | bEasy-app | T-0033 | T3 |
| T-0035 | Cross-utm-backend-store | cross (backend request) | T-0033 | T3 |
| T-0036 | Portal-utm-acquisition-columns | bEasy-portal | T-0035 | T2 |
| T-0037 | Cross-utm-e2e-test | cross | T-0034, T-0035 | T2 |

Parallel now: T-0031, T-0032. Then T-0033. Then T-0034 and T-0035 in parallel. Then T-0036, T-0037.
Overlap: T-0001 item D (campaign names on app opens) and `Campaign_Source_Request.md`; link, do not duplicate.

## Out of scope
Meta/TikTok ad spend and click import (T-0001), CAC/ROAS, mini-app attribution, push campaigns.
