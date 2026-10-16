# Marketing suite: backend needs (Push v3, Promotions, Search & Content)

**To:** backend · **From:** portal · **Updated:** 16 Oct 2026 · **Source:** Figma notes (decisions up to 11 Oct) · plan: `Tasks/_plans/Portal-marketing-suite.md`
Priority follows the build order. Shapes are for backend to propose; portal adapts. Everything is UTC in storage, Phnom Penh (GMT+7) in display; every status change goes to the activity log (who, what, when).

## 1. Promo codes (first)
Today `coupon/*` exists. Needed on top:
- Stable `promo_code_id`; code text unique ignoring case; text locked once active.
- **Ledger** per code: applied, redeemed, returned, lost. Uses left = limit − redeemed. Uses counted per customer id.
- Rules: one per order; not with a package; app only; counts when the order is paid; customer cancels a paid booking → not returned; bEasy or cleaner cancels → returned; fee capped at what was paid.
- Offers: percent, amount, or free service/add-on (service, variant, quantity, scope). Who: all, first-time (no completed booking), group, specific customer.
- List/detail fields: status, uses left, dates, who, services, pushes carrying the code. Pause/resume/end by Marketing manager or Super admin.
- Atomic checks so a code is never oversold.

## 2. Package deals
Prepaid sessions. Ids `package_id`, `purchase_id`. Services/variants per session, price (reference from catalogue, override needs permission + reason), validity (none / days-months from confirmed purchase / fixed date), who can see and buy (all / group / specific; private offers never shown to others). One package per checkout, no promo codes, non-refundable, repeat purchase while eligible; bought in app or payment link, used in app only.
**Session ledger per purchase:** issued = available + reserved + consumed + expired; booking reserves, cancel (free) returns, cancel with fee returns once the fee is paid. Purchase status: Active / Fully booked / Fully used / Expired, balance per service for mixed packages. Editing applies to new purchases only.
**Open for biz/finance:** fee larger than the money paid on a package booking (recommend: cap at paid, return sessions).
Please propose field names for expiry and limits that Push can read.

## 3. Push campaigns and automations
- Campaigns: objective (General / Promo code / Package), send type (one-time / automation), statuses (Draft, Scheduled, Sending, Completed, Active, Paused, Blocked, Needs attention), EN + KM title/body (required), optional banner (both languages if present).
- Audience = one saved CRM segment (`customer-segment`), re-checked before every send; automatic exclusions and an estimate endpoint.
- Limits: 1 marketing push a day, 4 in any 7 days, quiet hours 9 PM–8 AM; over the limit = skipped and recorded, never queued. Priority: service updates, then promo/package, then general.
- Offer re-checked at review, schedule, 5 min before send and at checkout. Promo code carried into checkout and checked again there.
- Send test to staff accounts only (up to 5, shared list). How is an account known to be staff?
- Automation: new joiners only, one wait + optional send time, re-enter after 30 days, stops at goal or when no longer qualifying.
- Tracking: delivered, opened (tap), booking within 7×24 h of the latest tapped push once per order. Which provider gives reliable delivered data?
- Permissions: pause/resume/stop = Marketing manager + Super admin; global limits = Super admin.

## 4. Content library and search
- Content: one library, each answer toggled per place (app search, service page, Help Centre, website). English enough to go live; Khmer later. Statuses incl. On hold (blocked by decision). Review: owner → Finance (price/policy) → Content Lead. Changing price/policy/inclusions/refund text returns it to review. Publishing log with re-publish of an older version. Website via a feed of approved content; failed push keeps old version live.
- Import of FAQ workbook V3 (268 answers, 853 search words, 85 intents); statuses map "Draft —" to Draft, "Hold —"/"Review —" to On hold with reason. Please confirm who supplies the file.
- Search: save a search when submitted, strip phones, addresses and names first; index published answers only; each search word points to one canonical FAQ ID. Ranking: safety/support, exact service, intent service, FAQ/Help, website article, other; never show unavailable services, on-hold content or offers the customer cannot use.
- Insights: overview, keywords (full access = Super Admin), keyword detail, tasks (owner, due date, proof, done after 30-day impact check). Revenue: same-session main number; 7-day influenced kept separate.

## 5. Mobile (not backend)
Apply a promo code from a push link (investigation: `Tasks/2-todo/App-investigate-autoapply-code.md`); app search that saves submitted searches; Help Centre content from the library; package purchase and session use.
