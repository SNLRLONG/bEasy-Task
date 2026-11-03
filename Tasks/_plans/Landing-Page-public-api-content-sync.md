# Plan: Landing page public API audit and content sync

**Updated:** 3 Nov 2026 · Owner: SNLRLONG (Seavlong) · Phase: investigation and planning only (nothing implemented, website untouched)

## Goal
The landing page lists services, tasks, equipment and app facts from static JSON. When the app changes (a service goes live or is hidden, tools change, task lists change), the site is stale. Find which **unauthenticated** API routes can feed the site, map them to sections, and decide what stays static. No redesign.

## How this was verified
- `bEasy-api/` is **not cloned**, so no backend `path:line` evidence. Routes come from the app (`bEasy-app/lib/util/app_constants.dart`), the mini-app (`src/api/endpoint.ts`) and the site (`lib/payment-client.ts`).
- Each route was called with a plain `GET`, **no token, no cookies**, on `api.beasy.info` (prod) and `api-dev.beasy.info` (dev), checking status, body shape and CORS. Only read-only GETs; nothing was written. Evidence is in this file; raw responses were kept in the session scratchpad only.
- Base path is `/api/customer/...`. The site already has `NEXT_PUBLIC_API_BASE_URL=https://api.beasy.info/api` (used only for payments today). All JSON is wrapped `{ code, status, message, data }`.
- A route being reachable without a token does **not** make it safe to use (see the security finding below).

## Finding: anonymous requests resolve to a shared guest user (lower severity)
`GET /api/customer/user/info` with **no token** returns 200 with a user record of `type: GUSTMODE` (`guest_…` username): the shared guest account the app uses for anonymous browsing, so no customer PII. Still odd: every anonymous caller maps to the same guest row, and the response includes that row's `deviceId` (a real device's id), `count` and `updatedAt`, which change with other sessions. `GET /api/customer/order/v5/list` also returns 200 without a token (confirmed empty: both buckets are `[]` for the guest, so no order data is exposed). The site must not call either. Logged as backend request 13A (corrected 3 Nov 2026 after the owner confirmed it is guest mode). Values are not copied here.

## Public endpoints that work without auth (prod, checked 3 Nov 2026)
Legend for use: **Use** = safe and useful for the site · **Maybe** = works, low value or needs a decision · **No** = do not use.

| Method + route | Purpose | Key fields | Use |
|---|---|---|---|
| `GET customer/category/list` | Full list of live bookable categories (7 on prod: 16 Post Renovation, 2 Deep, 9 Upholstery, 15 AC, 7 Office, 13 Pest Control, 17 Washing Machine) | `id`, `nameEn/Km/Cn/Vi/Tw`, `iconUrl`, `isComingSoon`, `badge`, `bookingMode`, `minLeadTimeMinutes`, `noteXx`, `taskInfoEn/Km/Cn/Vi/Tw` = `[{key, value[]}]` (the same "what's included" lists the site types by hand) | **Use** (core) |
| `GET customer/category/{id}/equipments` | Tools/equipment per category (works for 1, 2, 9, 15, 17) | `id`, `name`, `nameEn/Km/Cn/Vi/Tw`, `imgUrl` | **Use** (equipment sync) |
| `GET customer/homepage` | What the app home shows | `categoryList` (only 3 on prod: Upholstery, AC, Washing Machine; it is the app's home subset, not all services), `bannerList`, `promotionAnnouncements` (title/content + 5 language banners), `productBundleList`, plus user-shaped fields (`balance`, `primaryAddress`, `isGuest`) | **Maybe**: take banners/announcements only; do not use `categoryList` as the service list |
| `GET customer/banner/list`, `GET customer/banner/{id}` | App banners and detail images | `imgUrlEn/Km/Cn/Vi/Tw`, `name`, `type`, `deeplink`, `deeplinkType`, `imgDetailUrlXx` | **Maybe** (promo strip; deeplinks are app-internal) |
| `GET customer/category/list/detail` | App "See more" grouping | `categories[].items[]` (type CATEGORY or PRODUCT, ids, names, icons), `coupon` | **Maybe**: app layout, not content. Skip |
| `GET customer/config/app/version` | Store release state | `version`, `versionIos/Android`, `playstoreStatus`, `appstoreStatus`, `forceUpdate*`, `saleTarget` | **Maybe**: version badge only. `saleTarget`/force flags are internal; ask backend for a slim public shape |
| `GET customer/config/app/languages` | App languages | `defaultCode`, `languages[]` (en, km, zh-cn on dev/prod) | **No** (decided: languages stay manual) |
| `GET customer/help-video/list` | Tutorial videos | `videoFileXx`, `thumbnailXx`, `titleXx`, `descXx`, `durationXx`, `isRated` (user-specific) | **Maybe**: "How it works" video later; most non-English fields are null |
| `GET customer/config/app/system-config/RATING` | Rating criteria (Quality, Professionalism, Punctuality…) | names in 5 languages | **No** (labels for the in-app review form) |
| `GET customer/payment-method/list` | Payment methods | names, logos, `code`, `type` | **Maybe**: "ways to pay" icons; needs a decision (see gaps) |
| `GET customer/popup/getPopup`, `customer/enable-date`, `customer/coupon/list/preview` | App popups, booking calendar (stale 2025 dates on dev), coupons (empty) | – | **No** (app behaviour, not content) |
| `GET customer/product/{id}/detail` | One product incl. `taskInfoXx`, `category`, `amount` | `amount` is `0` on every product checked | **No** for prices (see gaps) |
| `GET customer/category/{id}/product-add-on` | Add-on options for a category | `nameXx`, `amount`, `duration`, `vat`, `imgUrl` | **Maybe**: only source of any real amounts; not "starting from" prices |
| `GET customer/product/v2/{id}/options` | Product options | returned `[]` for ids tried | **No** |
| `GET customer/category/popular`, `/customer/services`, `/slider`, `/news` | – | 500 on `popular`; 404 on the others | **No** (do not exist) |
| `GET customer/user/info`, `GET customer/order/v5/list` | User and orders | see the guest-user finding | **No, report** |
| 404 on prod for guessed names (`faq`, `reviews`, `stats`, `partners`, `testimonials`, `blog`, `announcement`, `service-area`) | – | – | no public source exists |

**Browser access (CORS):** `access-control-allow-origin: *`, `OPTIONS` returns 204. **Caching:** responses are `cache-control: private, no-store` and `vary` includes `Authorization`, `User-Agent` and app headers, so a CDN will not cache them as-is. The site should fetch **server-side** in Next.js (ISR / `revalidate`) and cache itself; client-side calls would hit the API per visitor.

## Landing page content map

Current site data: service list `app/translations/*/serviceSection.json` + slug/visibility map `lib/services.ts` (`SERVICE_SLUGS`), per-service detail `serviceDetail.json` (banner text, `taskList`, `serviceType.items` with typed prices like "$15", images in `/public/images`). Sites ids (2,3,4,6,7,8,9) differ from API category ids; `serviceDetail.json` already stores the API id in `categoryId` (2→2, 3→7, 4→9, 5→13, 6→17, 7→16, 8→15; 1 and 9 both map to 1).

| Site section | Today | Source after sync | Type |
|---|---|---|---|
| Home services list, navbar/footer service links, sitemap, JSON-LD service list | Static items + `SERVICE_SLUGS` (pest control and general cleaning are hidden by commenting out slugs) | `category/list`: show a service only if its `categoryId` is returned and `isComingSoon` is false. Order, name, icon from API; **slug, description, hero image, SEO keywords stay static** | **Both** |
| Service detail: "what is included" task list | Static English/Khmer text | `taskInfoXx` from `category/list` (5 languages incl. `zh`) rendered into the existing task-list UI; static copy is the fallback | **API-driven** (static fallback) |
| Service detail: equipment ("tools we bring") | None on the site today | `category/{id}/equipments` (name + image in the user's language) | **API-driven, new block** (needs design OK; see Open questions) |
| Service detail: hero, intro text, benefits, FAQ, CTA, booking OneLink | Static | – | **Static** (marketing copy, SEO, OneLinks in `lib/services.ts`) |
| Service detail: prices ("From $15", Instant $15/$30) | Static typed strings | **No usable public price today** | **Static until backend adds a price** (gap 1) |
| Instant Cleaning (site id 9) | Static page | `category/list` returns no separate Instant category (id 1 "Cleaning" is not in the prod list) | **Static**; gate on backend answer (gap 2) |
| Home promo/banner strip | None | `homepage.promotionAnnouncements` or `banner/list` | **API-driven, optional** (backlog) |
| Install section: app version / store links | Static OneLinks | `config/app/version` is usable, but exposes internal fields | **Static** now; **API** after a slim public shape |
| Hero, About, Our Story, Why choose us, Compare, Seen On, Media/press, Testimonials, FAQ, Contact, Privacy/Terms, Stats numbers | Static | no public source | **Static** (stats and testimonials need new backend endpoints; backlog) |

## Decision (3 Nov 2026, owner)
Service detail **banner/hero stays static**; it does not use API response data. The API is used only to **map each site service to its app category** (site id → `categoryId`) so the site knows whether that service is live. Everything the API supplies beyond that (task lists, equipment) goes only in the body blocks below the banner, as in T-0101.

## App languages (`GET customer/config/app/languages`): not used
Public, same on prod and dev (`en` default, `km`, `zh-cn`). **Decision 3 Nov 2026 (owner): not worth it; languages stay manual in code.** Reasons: they change rarely; adding or removing one needs code anyway (translations, routes, hreflang, sitemap); hide-only gives a half-state and an extra dependency. Site keeps `en/km/zh` and default `km`.

## Gaps and decisions
1. **No public price.** `product.amount` is 0; `product-add-on.amount` is per option, not "from" price. Site price strings stay static. Ask backend for a public `startingPrice` (+ currency, unit) per category in `category/list`. Without it, prices drift as before.
2. **Instant Cleaning and General Cleaning have no live category.** Site id 9 (Instant) and id 1 (General) map to API category 1, which prod `category/list` does not return. Decision: keep them static and hidden/shown by the existing slug map until backend states which category/product represents each; do not auto-hide them from the API.
3. **Site id vs API id.** Add one mapping table (site service id → API `categoryId`) in `lib/services.ts`; do not renumber site ids (slugs/redirects depend on them, see CLAUDE.md).
4. **Language keys differ.** API uses `Km/Cn/Tw/Vi/En`; site uses `km/en/zh`. Map `zh` → `Cn` (Simplified). `Tw`/`Vi` unused. Some fields are null in non-English (help videos), so always fall back to English then to static text.
5. **Pest Control.** API returns it as live while the site hides it on purpose. Decision: the static slug map stays the final gate (API can hide, never show a service the site deliberately hides).
6. **Caching and availability.** `no-store` + `Vary: Authorization`. Decision: server-side fetch with `revalidate` (for example 10 minutes) and a build-time snapshot; if the API fails the site renders the existing static content. The site must never go blank or fail its build because the API is down.
7. **Data minimisation.** Use an allowlist mapper (only the fields in the table). Never spread whole API objects into props; `homepage` carries user-shaped fields.
8. **Auth/security.** The two exposed user routes (finding above) go to backend; the site never calls anything under `user/`, `order/`, `auth/`.
9. **SEO.** Sitemap, JSON-LD and metadata must remain stable (CLAUDE.md: keep slug maps stable). API data must only change visibility, task text and equipment, not URLs.
10. **Backend repo not cloned**, so routes, public intent and guards are unconfirmed in code. Backend should confirm that the routes in the **Use** rows are intentionally public and will stay stable.

## Open questions (recommended answer in bold)
1. Show equipment on the service pages (new block, needs layout)? **Yes, small "Tools we bring" strip using the existing card style; ask design for approval before building.**
2. Should the API decide which services appear, or stay a hand-kept list? **API can hide, the site slug map is the final allow-list (decision 5).**
3. ~~Language switcher follows the app?~~ **Decided: no, languages stay manual.**
4. Should backend add a public price and a slim version endpoint, or accept static prices? **Ask for `startingPrice` on `category/list`; keep static until live.**

## Task breakdown
| ID | Task | Repo | Depends on | Tier |
|---|---|---|---|---|
| T-0098 | `Cross-public-api-backend-contract`: backend confirms public routes, fixes the user/info + order list exposure, adds price/stable ids/cache headers | Cross (backend asks) | none | T3 |
| T-0099 | `Landing-Page-public-api-client`: server-side fetch layer, allowlist types, language mapper, id mapping, cache + static fallback, tests | Landing page | none (routes already live) | T2 |
| T-0100 | `Landing-Page-services-sync`: home services list, nav/footer, sitemap and JSON-LD driven by `category/list` through the static allow-list | Landing page | T-0099 | T2 |
| T-0101 | `Landing-Page-service-detail-sync`: task list from `taskInfoXx`, new equipment block, static fallback | Landing page | T-0099; design OK for equipment | T2 |
| backlog | `Landing-Page-promo-banners-sync`, `Landing-Page-price-sync` (needs T-0098 price), `Landing-Page-app-version-sync` (needs slim version endpoint) | Landing page | T-0098 | T2 |

Order: T-0098 and T-0099 run in parallel (T-0099 uses routes already live). T-0100 and T-0101 both touch `lib/services.ts` and the service pages, so run them **one after the other** (T-0100 first), not in parallel.

## Out of scope
Redesigning the site, changing URLs or slugs, auth/booking flows on the site, payments, new tracking events, testimonials/stats/blog/press APIs (no source exists; backlog if wanted), any change to the app or portal.
