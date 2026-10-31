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
- [ ] Unsupported items (today's work, send to CS, problems found, per-list export) show "Not implemented yet", never 0
- [ ] Rates are 0-1 in the API and shown as percent; null money is "Unavailable", not $0
- [ ] Page uses v2 components and tokens; no horizontal scroll at 360, 768, 1280; `typecheck`, `lint`, `build` pass
- [ ] API spec and README parked note updated; root `CHANGELOG.md` entry `[Portal]` added
- [ ] Worktree and branch handled after merge (ask before removing if there are uncommitted changes)
- No commit or push until the user says so (`AGENTS.md`)

## Progress (2026-10-31)
- Worktree created from `origin/feature/cs-kpi` (a8a5d65d); `origin/main` merged in, **merge not committed yet** (waiting for the user's go-ahead). Four conflicts resolved by taking `main` and re-adding only the branch's additive bits: `overview-card.tsx` (`note`, `onClick`), `combobox.tsx` (`ariaLabel`), `table-pagination.tsx` (main's v2 Pagination, branch tweak dropped), `locales/en.json` (kept both key groups). Fixed `meta.width` type in `cs-employee-table.tsx`.
- Baseline after merge: `test:cs-overview` 18/18 and `test:v2-pagination` 6/6 pass; typecheck has no errors in CS files. Remaining typecheck errors exist on plain `origin/main` (duplicate imports in `new-service-bundle.tsx`, `location.tsx`, `chat-store.ts`, `vite.config.ts` and others); not touched.
- Not yet checked: `yarn lint` on the whole repo, `km.json` (the branch only added `en.json` keys), running the page against dev.
