# Plan: Marketing suite (Push v3, Promotions, Search & Content) + Journey wrap-up

Updated: 2026-10-16 · Figma: file `8bDJcg0dDmieddfLLjkloh`, page `199:63736` (4 sections) · Owner: Suntel · Planner: agent (no code written)

## Goal
Build the three new admin areas drawn in Figma, on the real backend, and finish Customer Journey v2.1. All are portal work (`bEasy-portal`); each needs backend endpoints, and a few need app or landing-page changes.

## What the Figma contains

| Section (node) | Frames | Status of the design |
|---|---|---|
| **Customer Journey v2.1** (`199:63961`) | 42 frames: 3 funnels, stage sheets, abandon rules, customer page, CS, audiences (on hold) | Final (biz review 7 Oct) |
| **Push notifications v3** (`199:94165`) | 40 frames: campaign list, Promo/General/Package wizards, campaign detail, automations, analytics, templates, send test, CRM push history | Draft for review (10 Oct) |
| **Promotions** (`199:115522`) | Promo codes: list, create (+ free-service offer), detail, pause; Package deals: list, create, detail, purchase detail, edit | Draft, decisions up to 11 Oct |
| **Search & Content** (`199:125362`) | Content library (all, editor, needs review, FAQs, help centre, website articles), Search insights (overview, keywords, keyword sheet), Tasks | Updated 16 Oct |

Faded "On hold" (Audiences, templates dynamic push) and "not chosen" (review option B, connected website) frames are **not built**.

## What the portal already has

| Area | Today | Plan |
|---|---|---|
| Journey + Customer page | Built on `feat/journey-cms`, tier T2, in progress | Wrap-up only (below) |
| Coupons | `routes/coupon/*` (list, 576-line form), API `coupon/*` | Reuse API and list patterns; rebuild screens to the Promo codes design |
| Push | `routes/push-notification/*` (old 840-line wizard), recovery-push calls | Replace with the v3 flow; keep recovery push untouched |
| Packages | `service-bundle`, `voucher` routes exist but are commented out | Do not revive; build new Package deals |
| Content, Search | nothing | New |
| CRM segments | `customer-segment` endpoint + `use-saved-customer-filters` | Reuse as the push audience source |
| Banners / popup analytics | exist | Reuse chart and table patterns |

## Gaps and conflicts found (decision made unless it needs the business)

| # | Gap | Decision |
|---|---|---|
| 1 | **Push limit differs.** Journey notes say 3 a week (to confirm). Push v3 says 1 a day and 4 in any 7 days, quiet hours 9 PM–8 AM | Follow Push v3 (newer, 10 Oct). Journey UI constant changes from 3 to 4 per 7 days. Backlog item updated |
| 2 | Promo code design adds a **ledger** (applied, redeemed, returned, lost), per-customer use counts and cancel rules; the current coupon API likely has none | Backend request. Portal starts on list/detail with the fields the API already returns |
| 3 | **Cancel rules**: 11 Oct overrides earlier lines (struck through in Figma): customer cancels a paid booking → code not returned; bEasy/cleaner cancels → returned; sessions come back (free) or once the fee is paid | Use the 11 Oct lines only |
| 4 | **Package + ABA booking cancelled when fee > money paid** is undecided (biz/finance) | Open question; recommend Option A (cap fee at money paid, return sessions): it matches the promo-code rule. Purchase-detail state for it is built last |
| 5 | No backend docs exist for push campaigns, automations, packages, content, search | New request file (below). Portal stays on the real backend, so these wait for endpoints |
| 6 | "Auto-apply code: does the app support it?" | App investigation task (T0). Wizard hides the toggle until answered |
| 7 | Send test: how is a staff account identified? | Open question; recommend matching an admin user's phone (no new CRM tag) |
| 8 | Content import needs the **FAQ workbook V3** (268 answers, 853 search words, 85 intents); it is not in the repo | Ask for the file; the import task waits |
| 9 | Website articles = a separate site reading a feed; `bEasy-landing-page-v2` is a Next.js site | Out of scope here; a Landing-Page task after the feed exists |
| 10 | Roles (Marketing manager, Super admin, Content Lead) drive pause/resume, global limits, keyword access | RBAC task (T3); other roles later |
| 11 | Sidebar "3 new Marketing items are placeholders until the final menu" | Add the entries behind routes; final order later |
| 12 | Khmer type (Kantumruy Pro) not in the design system yet | Use the portal's current Khmer font; flag in the Khmer review |

## Open questions (planning continues; recommendation first)
1. **Delivery receipts:** which push provider gives reliable "delivered"? Recommend labelling open rate "of sent" until confirmed.
2. **Package field names** for expiry and limits shown in Push (from Package Deal Admin). Recommend the portal defines them in the backend request now.
3. **Content roles** (CS, Marketing, Ops, Finance, Content Lead) permissions and "nothing ranks" fallback, FAQ helpful votes: recommend Super Admin full access only for this release.
4. **Mixed payment** proposal for packages (biz). Recommend out of scope until decided.
5. **Fee > paid** (gap 4).

## Flow in one line per module
- **Promo code:** create (offer, limits, who, dates) → active → customer uses at checkout → order paid counts a use → cancel rules return or keep it → detail shows the ledger.
- **Package deal:** create (services, sessions, price, validity, who can buy) → customer buys (app or payment link) → sessions reserved/used/returned → purchase detail.
- **Push:** pick objective (General / Promo / Package) → one-time or automation → audience (saved CRM segment) → message EN+KM → destination → schedule → review → confirm → campaign detail and analytics.
- **Content:** write answer → owner checks facts → Finance if price/policy → Content Lead reviews → live in English → Khmer later; channels toggled per place.
- **Search insights:** app searches saved (PII stripped) → keywords/overview → tasks per topic with owner and 30-day impact check.

## Task breakdown

Estimates are front-end working days for one developer (S ≤ 1 · M 2–4 · L 5+), excluding backend wait. Tier from the auto picker. 🔌 = needs backend first.

| Wave | Task (file) | Repo | Depends on | Tier | Size |
|---|---|---|---|---|---|
| 0 | Journey wrap-up: Figma browser pass, Khmer review, dev backend retest (`Portal-journey-cms`) | Portal | backend dev | T2 | M |
| 1 | `Portal-marketing-foundations-ui`: sidebar entries, routes, bilingual EN/KM field, wizard stepper, status badge, PP time helpers, activity-log row | Portal | – | T2 | M |
| 1 | `Portal-marketing-rbac`: permission modules and role checks for the new areas | Portal | – | T3 | S |
| 1 | `Portal-promo-codes-list-detail`: list, filters, detail, pause dialog | Portal | foundations-ui, 🔌 ledger fields | T3 | M |
| 1 | `Portal-promo-codes-create`: create/edit incl. free-service offer, limits, audience | Portal | list-detail | T3 | M |
| 1 | `App-investigate-autoapply-code`: can the app apply a code from a push link? | App | – | T0 | S |
| 1 | `Cross-marketing-backend-request`: see `Docs/.../backend-request/Marketing_Suite_Backend_Request.md` | Docs | – | T0 | S |
| 2 | Package deals: list, create, detail, purchase detail, edit dialog 🔌 | Portal | foundations-ui | T3 | L |
| 2 | Push list + create dialog, campaign detail (scheduled/completed/blocked/cancel) 🔌 | Portal | foundations-ui | T2 | M |
| 2 | Content library: all content, editor with full review, needs-review, FAQs, help centre 🔌 | Portal | foundations-ui, rbac | T2 | L |
| 3 | Push wizards: Promo (8 steps) then General then Package, send test 🔌 | Portal | promo codes, package deals, 🔌 | T2 | L |
| 3 | Search insights: overview, keywords, keyword sheet, tasks 🔌 | Portal | content library, 🔌 | T2 | M |
| 3 | Content import (workbook), publishing log, website feed status 🔌 | Portal | library, workbook | T2 | M |
| 4 | Push automations (create, detail, resume/stop), analytics (4 tabs), templates, CRM push history 🔌 | Portal | wizards | T2 | L |
| 4 | Website articles consumer | Landing-Page | feed | T2 | M |

Parallel groups: after foundations-ui, **promo codes**, **package deals** and **content library** touch different files and can run in three worktrees. Push wizards and search insights wait for the others. Rough total about 70 FE days; two developers or agents in parallel reach the push work in about 5 weeks.

Wave 1 tasks are in `Tasks/2-todo/`. Waves 2–4 are listed with sub-tasks in `Tasks/1-backlog/Portal-marketing-suite-later-waves.md` and move to `2-todo/` when their dependencies are done.

## Out of scope
Audiences, push templates with dynamic content, options B/connected website, mixed payment, complaint compensation packages (not drawn), TikTok, CS overview (parked).
