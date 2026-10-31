# Portal: CS Overview, CS target KPI and v2 UI

- **ID:** T-0058
- **Repo:** bEasy-portal
- **Owner:** SNLRLONG
- **Due:** 2026-11-04
- **Started / updated:** 2026-10-31 / 2026-10-31
- **Schedule:** 2026-10-31 → 2026-11-04 (5 days)
- **Worktree:** `bEasy/bEasy-portal-cs-overview/` (created 2026-10-31) · **Branch:** `feature/cs-kpi` (name set by the user; it already exists on `origin`, see Starting point)
- **Tier:** T2 · new page and route, new API calls, shared nav/permission wiring, UI migration to v2 (see `PIPELINE.md`)
- **Claimed by:** claude session, 2026-10-31
- **Docs:** **backend contract `Docs/tracking/v2/backend-response/CS_Overview_Frontend_Handoff_29Oct2026.md` (source of truth for this task)** · Figma `wA47r8vSgQmqYZDMnL99RI` (bEasy-Admin-Portal) node `1637:36622` · `Docs/tracking/v2/requirements/Journey_Tracking_UI_Flow_v2.1.md` §7 · `Docs/tracking/v2/api/Journey_Backend_API_Spec.md` §5 · `bEasy-portal/docs/new-page-guide.md` · RDS plan `Tasks/_plans/Portal-responsive-design-system.md`
- **Depends on:** backend `/api/admin/cs-overview*` (built). Target GET/PUT also needs `sql/cs-overview-targets.sql` applied and `CS-OVERVIEW` VIEW/UPDATE granted on the dev backend; verify before building the editor. Coordinate with the author of the existing branch (see Starting point). UI primitives from the in-progress `Portal-rds-*` tasks.
- **Touches:** `app/routes/customer-services/cs-overview.tsx`, `app/components/cs-overview/*`, `app/hooks/query/use-cs-overview*-query.ts`, `app/lib/cs-overview.ts`, `app/types/cs-overview.ts`, `app/lib/permission.ts` (only if the module changes), `app/components/sidebar/nav-config.ts`, `app/routes.ts`, `app/api/endpoint.ts`, `app/constants/query-key-enum.ts`, `app/components/headers/header-title.tsx`, `app/locales/*.json`, `tests/cs-overview.test.mjs`
- **Overlaps:** `Portal-rds-customer-service` (T-0045, in progress) migrates the other pages in `routes/customer-services/`; it does not own `cs-overview.tsx`, but both edit `nav-config.ts`, `routes.ts` and locale files. Re-read before editing; do not touch its files.

## Starting point (do this first, nothing is done yet)
1. `git fetch origin` and `git -C bEasy-portal worktree list`; confirm no live worktree already uses `feature/cs-kpi`.
2. `origin/feature/cs-kpi` **already exists** with a working CS Overview (6 commits by bunlongly, 2026-10-25: ~830-line `cs-overview.tsx`, employee table, people sheet, month filter, `tests/cs-overview.test.mjs`). It is 6 ahead and 43 behind `origin/main`. Create the worktree **from that branch**, then merge latest `origin/main` into it (user asked to pull latest main). Do not recreate the page from the old `draft/cs-overview` snapshot.
3. Resolve merge conflicts (expect `package.json`, `nav-config.ts`, `routes.ts`, `en.json`, shared components touched on the branch: `overview-card.tsx`, `table-pagination.tsx`, `combobox.tsx`, `table-rows.tsx`). Run typecheck, lint and the existing tests before changing anything, and note the baseline.
4. Message the branch author in `Tasks/_inbox/` or tell the user, so two people do not edit the branch at once.

## Goal
On top of the existing branch, deliver the **Overview** (Super Admin view) as the Figma shows it, add the **CS target KPI**, and make it match portal v2.

- **Overview scope (Figma node `1637:36622`, Super Admin view), limited to what the backend supports (handoff §1):** provisional banner, First sales (Contacted vs No CS contact, by employee, vs target), Other sales without CS contact, Repeat sales by employee, recorded contact activity, saved CRM segments with live counts, monthly Excel export, definitions, plus empty, loading, error and unavailable states. **Not built by backend:** Today's follow-up work (assigned, not contacted, due, overdue, on time), send to CS, pick-up, reassign. Show `—` and "Not implemented yet" from the `availability` entries; no fabricated totals, actions disabled. **Problems found by CS is out of scope** (backend); do not show it or count it in the KPI.
- **Target KPI for CS:** monthly targets come from `GET /cs-overview/targets/:month` (fetched separately; not in the summary; defaults 30 / 30 / 90 with `configured: false`). Build the editor on `PUT` (all three values: first-sale conversion %, repeat conversion %, completed-on-time %), visible only with `CS-OVERVIEW` UPDATE. Compare `rate * 100 >= target` (equal meets target; keep target `0`); `rate: null` is N/A; `ratingStatus: not_rated` or "Employee not recorded" is Not rated. The on-time target is editable but has no actual value yet. A target 403 or 503 must not blank the report.
- **v2 UI consistency:** use `app/components/v2/` and Suntel tokens as the migrated customer-service pages do (see `Portal-rds-customer-service`); no old `CustomHeader`/`h-screen` frame, no raw hex, palette classes, text under 12px or arbitrary radius in new code; works at 360, 768 and 1280 with no horizontal page scroll.
- **Wiring checklist** (`new-page-guide.md`): route + `handle` (module/action), sidebar sub-item under Customer Service, header title, query keys, endpoints, query hooks, `useCanGoBack` only if sub-routes are added, EN + KM keys.

## Gaps and decisions
| # | Gap | Decision |
|---|---|---|
| 1 | Branch exists, the old task said to build from `draft/cs-overview` | Build on `origin/feature/cs-kpi`; ignore the draft snapshot |
| 2 | Branch calls `cs-overview` and `cs-overview/people`; the spec said `/cs/overview` | **Resolved by the handoff:** `/cs-overview*` is live, `/cs/overview` is a 501 placeholder. Spec updated 31 Oct |
| 3 | Branch gates the sidebar item and route with `MODULES.CUSTOMER_OVERVIEW`; target APIs check `CS-OVERVIEW` VIEW/UPDATE; summary and people have no CS permission check. Backend says the mapping needs the project owner to align it | **Confirmed by the user (31 Oct): module is `CS-OVERVIEW`.** Route and sidebar `CS_OVERVIEW` + VIEW, editor behind UPDATE. Do not invent role grants; the user sets up the test role's grants. Tell the user before release that summary and people are not server-protected |
| 4 | Spec §5 and README said the CS Overview is parked | Done 31 Oct (handoff filed in `backend-response/`, spec and README updated). Re-check at G7 |
| 5 | Figma also has Follow-up (Super Admin), Lists and report, and CS view (find and pick up, work my cases, send to CS) | **Out of scope.** Only the Super Admin Overview. Add a backlog task per extra view (`1-backlog/`) when this is done |
| 6 | Branch is on the old UI; RDS v2 primitives are still landing | Use v2 components that already exist on main; if a needed one is not merged yet, leave the old one and list it here rather than building a duplicate |

## Open questions (do not block; recommendation given)
1. ~~Who may change the target?~~ **Answered:** `PUT /cs-overview/targets/:month` with UPDATE permission; the editor is in scope.
2. ~~Problems-found 5% share~~ **Answered:** out of scope per backend; leave it out.
2b. Is `sql/cs-overview-targets.sql` applied on the **dev** backend (never test the target PUT on production)? (The user handles the `CS-OVERVIEW` grants for the test account.) Unknown. Recommendation: check with a GET on day 1; if 503, build the editor against the contract and keep it disabled.
3. Branch ownership: confirm with bunlongly that SNLRLONG continues on `feature/cs-kpi` now. Recommendation: yes, rebase work on top of their commits, no force-push.

## Gates (T2, from `Tasks/PIPELINE.md`)
- [ ] G1 scope  - [ ] G2 static (format, lint, typecheck)  - [ ] G3 tests (existing pass; new logic such as the vs-target state has tests)
- [ ] G4 run it (dev backend, compare with Figma `1637:36622`, viewports 360/768/1280)
- [ ] G5 code review (fresh agent)  - [ ] G6 only if permission or data handling changes
- [ ] G7 docs sync (API spec §5, `Docs/tracking/v2/README.md` parked note)  - [ ] G8 PR message  - [ ] G11 changelog

## Done when
- [x] Worktree `bEasy/bEasy-portal-cs-overview/` on `feature/cs-kpi`, based on the remote branch with the latest `origin/main` merged; baseline typecheck/lint/tests recorded
- [ ] `/cs-overview` shows the Figma Overview sections with loading, error, empty and unavailable states, behind `CS-OVERVIEW` VIEW, and appears under Customer Service in the sidebar
- [ ] First-sale (and repeat) conversion is shown against the month's target per employee and team with on / under / not rated / N/A; the target editor saves all three values and handles 403, 422 and 503
- [x] Unsupported items (today's work, send to CS, problems found, per-list export) show "Not implemented yet", never 0
- [x] Rates are 0-1 in the API and shown as percent; null money is "Unavailable", not $0
- [ ] Page uses v2 components and tokens; no horizontal scroll at 360, 768, 1280; `typecheck`, `lint`, `build` pass
- [x] API spec and README parked note updated; root `CHANGELOG.md` entry `[Portal]` added
- [ ] Worktree and branch handled after merge (ask before removing if there are uncommitted changes)
- No commit or push until the user says so (`AGENTS.md`)

## Progress (2026-10-31)
- Worktree created from `origin/feature/cs-kpi` (a8a5d65d); `origin/main` merged in, **merge not committed yet** (waiting for the user's go-ahead). Four conflicts resolved by taking `main` and re-adding only the branch's additive bits: `overview-card.tsx` (`note`, `onClick`), `combobox.tsx` (`ariaLabel`), `table-pagination.tsx` (main's v2 Pagination, branch tweak dropped), `locales/en.json` (kept both key groups). Fixed `meta.width` type in `cs-employee-table.tsx`.
- Baseline after merge: `test:cs-overview` 18/18 and `test:v2-pagination` 6/6 pass; typecheck has no errors in CS files. Remaining typecheck errors exist on plain `origin/main` (duplicate imports in `new-service-bundle.tsx`, `location.tsx`, `chat-store.ts`, `vite.config.ts` and others); not touched.
- Not yet checked: `yarn lint` on the whole repo, `km.json` (the branch only added `en.json` keys), running the page against dev.

## Progress (2026-10-31, later)
Done, uncommitted in the worktree: target logic (`lib/cs-targets.ts`, `types/cs-targets.ts`, target query + update hooks, endpoint and query key, `tests/cs-targets.test.mjs` 8 tests); contract fixes (types: `ratingStatus`, `csId: 'unknown'`, nullable `reached`, `availability`, `tiles`, 3 missing metrics; formatters `formatCsMoney`, `reachedLabel`; null shows "Unavailable"; en keys `unavailable`, `notRated`, `notApplicable`). `test:cs-overview` 21/21, `test:cs-targets` 8/8.
Audit of the existing page (read-only agent, spot-checked) still open: switch route/sidebar to `CS_OVERVIEW`; remove Problems found card; "Employee not recorded" option and `csId=unknown` click; links for completed-paid and not-reached; revenue links with `sortBy=revenue`; people sheet tiles; target indicator and editor UI (gated by UPDATE); saved-segments section; migrate to v2 components (no `h-[calc(100vh-88px)]`, no raw hex or palette classes from `segment-tone.ts`, no `rounded-[4px]`); Khmer keys (`km.json` has none); guard null `finalAt`.

## Progress (2026-10-31, evening)
- `origin/main` merge committed on the branch (`2c5e620f`); branch level with main. Backend answers filed in `backend-response/CS_Overview_Backend_Answers_31Oct2026.md`; questions archived.
- Route and sidebar use `CS_OVERVIEW` (new-page-guide steps checked; `CUSTOMER_OVERVIEW` untouched). Export gated by EXPORT, Edit targets by UPDATE.
- Page migrated to v2 (route, employee table, people sheet, number button, opportunity tiles; no 100vh, raw hex or palette classes). Problems found removed. Employee list uses `hasActivity`/`isCsStaff` with zero rows. 403 shows an access-restricted state.
- Targets wired: month query (separate from the report), vs-target badge in first-sale and repeat tables (a vs-target column was added to the repeat table), editor dialog, per-month note for range/overall, 403/503 notice. On-time target is edit-only.
- Checks: `test:cs-overview` 21/21, `test:cs-targets` 8/8, eslint and typecheck clean on CS files, `yarn build` passes. Other typecheck errors exist on plain `origin/main`.
- Not done: run in browser at 360/768/1280 against dev (targets GET returns 503 until backend deploys the SQL); Khmer keys (user: not important); saved-segments section (user: UI not ready; backend added `POST /customer/counts`); unknown-employee click (`csId=unknown`, user unsure); completed-paid / not-reached links and `sortBy=revenue` (undecided); nothing committed except the merge.

## UI review vs Figma `1637:36622` (2026-10-31, page run on dev at 1440 and 430 wide)
Verified in the browser: page loads with real dev data, mobile Filters sheet works, employee Combobox opens above the sheet, searches and selects (only its look differs: pill shape and shorter than the other selects, popover narrower than its trigger). Targets GET returns 403 for the test role (page handles it with a notice); saved-segment counts show "Unavailable" because `POST /customer/counts` is not deployed yet.

| # | Figma says | Page now | Backend supports? |
|---|---|---|---|
| 1 | Header: breadcrumb, subtitle "Super Admin only…", **Edit targets** + **Export month** | "Export report", other subtitle | yes (copy only) |
| 2 | Filters: **Month only**, CS employee, Service, Follow-up group, label inline in the pill. Staging note: remove Date range / Overall | Month / Range / Overall switch | yes |
| 3 | Provisional banner with the number of customers still in their window | generic text | `firstSales.provisional`, `period.finalAt` |
| 4 | **Saved segments**: grid of cards (icon, name, big count, rule + updated time); tap opens a side sheet: "Saved CRM segment from Marketing", view only, rows newest first with full phone, tiles Not with CS / With CS / Booked since, Send to CS | list rows with an Open button, no sheet | counts need `POST customer/counts` (not deployed); tiles and Send to CS not built |
| 5 | "Customer follow-up opportunities" removed (staging note) | still shown | n/a |
| 6 | Today's follow-up work: 4 tiles + table (Open cases reg · existing, Not contacted, Due today, Overdue, **On time · target 90%**, Attempts → reached) | placeholders, empty table | only Attempts → reached exists (`activity`); rest not implemented |
| 7 | Response time table | missing | not implemented |
| 8 | "Problems found by CS" strip is **back** (Nith, 30 Oct) | removed | out of scope this release (`availability.problemsFound`) |
| 9 | First sales: two cards (conversion %, bar, 4 rows incl. completed paid) | present, needs visual pass | yes |
| 10 | By-employee tables: **Above / On target / Below / Not rated / N/A** badges, column "vs target 30%", "Employee not recorded" row with "?" avatar and clickable numbers, team total as a footer row, footnote about customers contacted by two people | badges On/Under, no unknown click, team total is a strip, no double-contact note | `doubleContacted` is not in the handoff (ask backend) |
| 11 | Edit targets dialog: Month field, Above/Below description, close X | similar, month only in title | yes |
| 12 | Export month dialog: checklist of 8 sheets | direct download, no dialog | backend file is fixed at 6 sheets, no selection |
| 13 | Customer list sheet: icon + title + subtitle (group · month · rule), count, tiles Completed / Scheduled / Refunded, search + Service + CS employee filters, row chips, footer count + Export list | v2 sheet, simpler rows | tiles only for contact metrics; no search or extra filters; no list export |
| 14 | Attempts sheet: Reached / Not reached / Unavailable tiles, one row per attempt, Load more | partly | tiles yes; search, Result filter no |
| 15 | States: loading skeleton, empty month, filtered no results (Clear filters), error (Try again), no permission (lock card, header actions hidden) | loading/error/403 exist; filtered-empty and copy differ | yes |

### Decisions I need (defaults in bold; work continues on everything else)
1. **Month only**, remove Range / Overall (design staging note). The previous author built range and Overall, and backend still supports them.
2. **Re-add Problems found as a "Not implemented yet" strip** (design 30 Oct says it is back; backend says out of scope; the earlier "remove" was before I saw the 30 Oct note).
3. **Export month dialog lists the 6 sheets the backend really produces** (read-only list + Download), not an 8-checkbox picker.
4. Registered saved segments stay view-only count until the Registered Customers page can take a preset (note to `Portal-rds-customer-service`).

### Work breakdown: lanes that can run in parallel (disjoint files, one owner each; the route owner wires the rest)
| Lane | Scope (design rows) | Owns (files) | Needs |
|---|---|---|---|
| A. Route shell | 1, 2, 3, 5, 15, header actions per permission | `routes/customer-services/cs-overview.tsx`, `cs-state.tsx`, `cs-opportunity-tile.tsx` (delete), `cs-tone.ts` (delete) | decisions 1, 2 |
| B. Saved segments | 4 | `cs-saved-segments.tsx` (tile grid), new `cs-segment-sheet.tsx`, `lib/cs-segments.ts`, `use-segment-counts-batch-query.ts` | backend deploy for counts (not blocking) |
| C. Report tables | 9, 10 | `cs-employee-table.tsx`, `cs-target-badge.tsx`, `cs-vs-target-cell.tsx`, `lib/cs-targets.ts` + test, new `cs-first-sale-cards.tsx`, `zero-rows.ts` | none |
| D. List sheets | 13, 14 | `cs-people-sheet.tsx`, new row/chips component, tiles from `tiles` | none |
| E. Dialogs | 11, 12 | `cs-target-editor.tsx`, new `cs-export-dialog.tsx` | decision 3 |
| F. Placeholders | 6, 7, 8 | `cs-work-card.tsx`, new `cs-response-time.tsx`, new `cs-problems-strip.tsx` | decision 2 |
| G. Employee select | Combobox look | new `cs-employee-select.tsx` (v2, same behaviour, popover as wide as trigger) | none |
Shared files (`en.json`, `package.json`, tests) are edited only by the parent; each lane reports the keys it needs. After the lanes: run in browser at 1440 / 768 / 430, then G5 review. Out of this release: Follow-ups screens, CS view, reassign (marked out), response-time data, problems list, per-list export.

## Progress (2026-10-31, parallel lanes A–E done and wired)
Decisions taken (defaults): month only; Problems found as "Not implemented yet" placeholder; Export dialog lists the 6 real sheets; registered segments open the same view-only sheet.
Built: month-only shell and states, saved-segment cards + view-only customer sheet (batch counts with quiet per-segment fallback), Today's follow-up work (live Attempts → reached), Response time and Problems placeholders, first-sale cards, by-employee tables (Above / On target / Below / Not rated / N/A, Employee not recorded row, footer team total), people/attempts sheet restyle (tiles, chips, revenue sort), Edit targets and Export dialogs, v2 employee select. Targets read from `GET cs-overview/targets/:month` (dev returns defaults, `configured: false`).
Verified in Chrome on dev (admin login) at 1440 and 430: page, segment sheet, list sheet, both dialogs (Save not pressed, no write), employee select inside the phone Filters sheet, no horizontal scroll. `test:cs-overview` 62/62, eslint and typecheck clean on CS files, `yarn build` passes, no raw hex / palette / sub-12px / 100vh / arbitrary radius in CS files.
Backend: only `POST /customer/counts` is 404 on dev (request written in `pending-request-backend.md` §8); targets table is installed.
Known gaps vs Figma: Today's table team total is a strip, not a footer row; "Attempts → reached" reads "9 › → 2 ›" instead of "9 → 2 ›"; Figma's Month pill label spacing and breadcrumb (no v2 breadcrumb); search and extra filters in sheets, status tiles on first-sale lists, Send to CS, per-list export and `doubleContacted` are backend gaps; Problems found stays a placeholder until the lead confirms (design 30 Oct says it is back, backend says out of scope).
Not done: Khmer keys (user: not important); 360 and 768 widths not walked (430 and 1440 were); G5 code review and `/security-review` not run; nothing committed except the main merge; Not tested: saving targets (PUT) and the Excel download on dev.

## Review gates (2026-10-31, later)
G5 code review (fresh agent, commit 8e610b24) and G6 security review (fresh agent): security found no exploitable issue (2 low hardening points, both fixed: customer id encoded in the journey link, real Error for the 404 marker). Code review fixes applied: unknown-employee numbers now open in the activity and Today's tables (shared "Employee not recorded" cell); by-employee rows memoised so a re-render no longer resets the table page; segment and people sheets no longer show 0 or stale counts while loading or switching list; page reset on tile switch; sales-table pager clamp; target editor no longer wiped by a background refetch; N/A and toast text through i18n; employee select announces its value. Tests 63/63, build passes, typecheck and lint clean on CS files.
Not fixed (low, listed for later): CsNumberButton has no per-metric accessible name; disabled buttons explained only by `title`; sheets remount (no exit animation); duplicated `open` helpers and two employee-cell / pagination implementations; a few test-only exports (`countAt`, `hasUniqueKeys`) and weak assertions in two tests; unknown-employee click not verified live (dev has no such row in October).
Next: commit the fixes, open the PR (tier T2: G8, G9 optional), then move to `4-testing/`; backend deploy of `POST /customer/counts`; Problems found decision from the lead.
