# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace layout

`bEasy/` is **not** a repo itself — it holds four independent git repos for the bEasy home-services product (Suntel Technology Cambodia). Each has its own git history, dependencies, and tooling; run commands from inside the relevant sub-directory and commit per repo.

| Directory | What it is | Stack |
|---|---|---|
| `bEasy-app/` | Customer mobile app (iOS/Android) | Flutter 3.38.7 (via FVM), GetX, Dio |
| `bEasy-portal/` | Admin portal (orders, cleaners/partners, finance, marketing, RBAC) | React 19, React Router 7 framework mode (SPA), TanStack Query, Zustand |
| `beasy-mini-app/` | Booking mini-app that runs **inside ABA Mobile** (bank super-app webview) | React 19, Vite, react-router, TanStack Query, Zustand |
| `bEasy-landing-page-v2/` | Marketing site (SEO, localized) | Next.js 16 App Router, Tailwind 4 |

All four talk to the same backend (`https://api-*.beasy.info/api/...`; the mini-app uses the `/api/customer` surface). The web projects deploy to Firebase Hosting; web projects use yarn (`yarn.lock`) and a `@/` path alias.

`bEasy-portal/CLAUDE.md` already documents the portal in depth — read it when working there.

## bEasy-app (Flutter)

```bash
fvm flutter pub get
fvm flutter run --flavor dev --target lib/main_dev.dart      # also: prod → lib/main_prod.dart
fvm flutter analyze
fvm flutter test                                             # all tests
fvm flutter test test/promo_popup_frequency_test.dart        # single test file
```

- **Flavors**: entry points `lib/main_dev.dart` / `lib/main_prod.dart` call `mainCommon()` in `lib/main.dart` with a `Flavor`, API base URL and MQTT URL from `AppConstants`; these are held in the `FlavorConfig` singleton. A `staging` flavor is referenced in `.vscode/launch.json`, `deploy_android.sh` and `shorebird.yaml`, but `lib/main_staging.dart` does not currently exist.
- **Releases**: `version:` in `pubspec.yaml` is used for iOS; Android build name/number are passed on the command line (see `deploy_android.sh`, which is a cheat-sheet of build commands rather than a runnable script). OTA patches ship via Shorebird (`shorebird release|patch <platform> --flavor <f> --target lib/main_<f>.dart`).
- **Architecture** (GetX): `lib/helper/get_di.dart` registers everything with `Get.lazyPut` — `DioClient` (`lib/data/api/api_client.dart`) → repositories in `lib/data/repository/` → controllers in `lib/controller/`. Screens in `lib/view/` read controllers via `Get.find`/`GetBuilder`. Navigation is mostly imperative `Get.to` (`lib/util/next_screen.dart`).
- `lib/chat_feature/` is a self-contained module (own Dio client, bindings, routes, MQTT/socket services) — see `docs/mqtt_mobile_chat_guide.md`.
- Attribution/analytics: `lib/analytics/` (Firebase), `lib/app_tracking/` (AppsFlyer, Facebook). `docs/` holds design notes and audits for these and for booking/order flows; check it before changing tracking or same-day booking logic.
- Lints: `flutter_lints` with `avoid_print` and `no_leading_underscores_for_local_identifiers` disabled.

## beasy-mini-app (ABA Mobile mini-app)

```bash
yarn dev            # vite --mode dev (also dev:uat, dev:prod)
yarn build          # client (uat mode) + SSR bundle + generate-static.js prerender
yarn lint / yarn lint:fix / yarn format
yarn deploy:dev     # build client + firebase deploy hosting:dev
```

- Runs in the ABA Mobile webview. Native bridge calls go through `web-bridge-gateway` (`callHandler`/`registerHandler`); bridge payload types are in `src/types/aba-bridge.d.ts`. `src/hooks/use-auth.ts` fetches the ABA profile + location via the bridge and posts them to the backend to auto-login (token stored in the `token` cookie). `isABAMobile()` in `src/lib/env.ts` detects the host via user agent; `VITE_CHECK_ENVIRONMENT` toggles that guard.
- Routes are declared inline in `src/App.tsx`. `src/entry-client.tsx`/`src/entry-server.tsx` + `server.js` provide SSR; `generate-static.js` prerenders `/` and `/service`.
- `src/api/api.ts` is an Axios instance that unwraps `response.data.data`; endpoints in `src/api/endpoint.ts`; query keys in `src/constants/query-key-enum.ts`. Data hooks are flat in `src/hooks/`; Zustand stores in `src/hooks/store/`.
- vConsole is injected in `dev`/`uat` modes for on-device debugging; `production` mode strips `console.*`/`debugger`.
- Husky + lint-staged on commit; commitlint enforces conventional commits.

## bEasy-landing-page-v2 (Next.js)

```bash
yarn dev             # next dev --turbopack
yarn build
yarn lint
yarn deploy          # build + firebase deploy (site: beasy-page, SSR via frameworksBackend)
yarn deploy:preview  # preview channel "stagingLink"
```

- **Locales**: `middleware.ts` redirects every non-asset path to `/{km|en|zh}/…` (default `km`); pages live under `app/[lang]/`. `app/purchase-coupon` is excluded from locale routing.
- **Translations** are per-section JSON files in `app/translations/{English,Khmer,Chinese}/`, read via `useTranslate()` (`t("section.key")`). Add keys to all three languages. Details: `TRANSLATIONS.md`.
- **Analytics**: `trackEvent()` auto-prefixes names with `web-v2:`; `EVENTS.md` is the event catalogue — update it when adding events.
- **SEO**: service/media pages use slugs (`lib/services.ts` `SERVICE_SLUGS`, `lib/media.ts`); `next.config.ts` permanently redirects old numeric URLs to slugs, so keep those maps stable. JSON-LD and metadata helpers are in `lib/jsonld.ts`/`lib/seo.ts`; see `IMAGE_SEO_GUIDE.md` and `IMAGE_PROTECTION.md` for image handling.

## Journey tracking docs (keep in sync)

`Docs/tracking/v2/README.md` is the index. One fact lives in one place; update it there and link, never copy.

| Fact | Single home |
|---|---|
| Open asks to backend | `backend-request/pending-request-backend.md` (open items only; later items in `backlog.md`) |
| What backend replied / what is live | `backend-response/` (newest dated file wins) |
| Endpoint shapes and live status | `api/Journey_Backend_API_Spec.md` |
| Mobile events and data model | `architecture/` |

When something changes, do all of these in the same session:
1. **Backend replies** → delete the answered item from `pending-request-backend.md`, update the status in the API spec, and add the reply to `backend-response/`.
2. **Portal or app behaviour changes** (new param, new call, new header) → update the API spec or `architecture/`, and add or close the matching request.
3. **Finished or replaced doc** → move to `archive/` with a one-line banner pointing to its replacement.
4. Set `Updated: <date>` at the top of any request file you edit. No history logs inside docs; git and the response files hold history.

Do not save doc contents, status or dates in Claude memory; memory only holds pointers to the index above.

## Tasks and worktrees

`Tasks/` holds one file per task; its folder is the status (`1-backlog` → `2-todo` → `3-in-progress` → `4-testing` → `5-done` → `6-deploy`). Rules and templates are in `Tasks/README.md`.

- One task = one worktree (`bEasy/<repo>-<short-name>/`) = one branch. The repo's main folder (e.g. `bEasy-portal/`) is the final checkout and is never deleted.
- Task files are named `<Prefix>-<short-name>.md` (`Portal-`, `App-`, `Mini-App-`, `Landing-Page-`). Start a task: copy `Tasks/_TEMPLATE.md`, set owner, create the worktree, move the file to `3-in-progress/`. Check `git worktree list` for overlap first.
- Merged to `main`: add an entry to the root `CHANGELOG.md` prefixed `[Portal]`/`[App]`/`[Mini-App]`/`[Landing-Page]` (owner, task, branch/PR, what changed; never a changelog inside a sub-repo), remove the worktree and branch, move the task to `5-done/`. Ask before removing a worktree with uncommitted changes.

**Parallel agents:** one agent = one task file. Claim it (`Claimed by:`) before editing, stay inside that task's worktree and `Touches`, never touch another task's worktree, and never use `git add -A`/`stash`/`reset --hard` in shared places. To reach another task, write a note in `Tasks/_inbox/<to-task>/` and read your own inbox at the start. Full rules: `Tasks/README.md` ("Working in parallel").
