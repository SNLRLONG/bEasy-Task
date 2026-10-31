# Marketing suite: waves 2–4 (sub-tasks)
- **Due:** 2026-10-30

- **ID:** T-0007

Plan: `Tasks/_plans/Portal-marketing-suite.md`. Move a block to `2-todo/` as its own task file when its dependencies are done. Tiers are the auto-picker result.

## Wave 2
**Package deals (T3, L)** · needs 🔌 package endpoints, foundations-ui
- list + filters; create (services, variants, sessions, price with catalogue reference and override reason, validity: none / days-months / fixed date, who can see and buy: all / group / specific); detail (sessions, purchases); purchase detail with session ledger (issued = available + reserved + consumed + expired); edit dialog (new purchases only); activate/pause/end by permission. Fee > paid state last (open question).
- Figma: `199:117708` list, `199:118244` create, `199:118753` detail, `199:119322` purchase, `199:119816` edit.

**Push list + campaign detail (T2, M)** · needs 🔌 campaign endpoints
- campaigns list (types, statuses, needs-attention bar, blue numbers open customer list); create-push dialog (objective, then one-time or automation); campaign detail scheduled / completed / blocked; cancel dialog; staff view.
- Figma: `199:94205`, `199:100423`, `199:101992`, `199:102477`, `199:103039`, `199:103532`, `199:111367`.

**Content library (T2, L)** · needs 🔌 content endpoints, rbac
- all content list; editor with full review (option A) and checking view; needs-review tab; service FAQs; help centre; status On hold; review steps owner → Finance → Content Lead; channel visibility toggles; changing price/policy sends back to review.
- Figma: `199:125411`, `199:126034`, `199:126680`, `199:127959`, `199:128598`, `199:129127`. Skip option B frames.

## Wave 3
**Push wizards (T2, L)** · needs promo codes, package deals, 🔌 push endpoints
- Promo push 8 steps (details, promo code, audience from CRM segment, message EN+KM, destination, schedule, review, confirm); General push (Marketing / Announcement only); Package push; Send test (staff accounts only, up to 5); limit checks 1/day, 4 per 7 days, quiet hours 9 PM–8 AM; offer re-check at review, schedule, 5 min before, checkout.
- Figma: `199:94906`, `199:95332`, `199:101015`, `199:95797`, `199:101500`, `199:96283`, `199:96757`, `199:97307`, `199:97904`, `199:98335`, `199:104030`, `199:104502`, `199:112939` to `199:114975`.

**Search insights (T2, M)** · needs content library, 🔌 search endpoints
- overview; keywords (full access = Super Admin); keyword detail sheet; tasks per topic with owner, due date, proof, done only after 30-day impact check. Search saved on submit with PII stripped (server-side).
- Figma: `199:130778`, `199:131454`, `199:132070`.

**Content import + publishing log + website feed status (T2, M)** · needs FAQ workbook V3
- import maps statuses (Draft / Hold / Review); canonical FAQ ID per search word; publishing log with re-publish of an older version; website Failed + Retry, never "Published".

## Wave 4
**Push automations (T2, L)**: trigger and audience, timing and repeat (new joiners only, re-enter after 30 days), review and turn on, detail active / needs attention, edit, resume, stop. Figma `199:104966` to `199:107450`, `199:111852`, `199:112389`.
**Push analytics (T2, L)**: overview, campaigns, automations, offers; booking counts if paid within 7×24 h of the latest tapped push; open rate label (delivered vs sent). Figma `199:108015` to `199:109723`, `199:98809`.
**Templates + CRM push history (T2, M)**: Figma `199:99441`, `199:99901`, `199:110875`, `199:110353`. Dynamic-content templates are on hold.
**Landing-Page: website articles consumer (T2, M)**: reads the approved-content feed; separate repo task.
