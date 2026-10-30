# Backend Backlog: Customer Journey

Not requested yet. Moved here from the pending list (16 Oct 2026). The backend returns `501` for the CS and audience endpoints today.

## Parked until the portal builds it
- **CS inbox and overview:** create cases from a list, first picker owns, contact log (channel, result, said, problem, next follow-up), suppression on "no more calls", overview aggregates (30-day windows, first sale credited to the last sales follow-up). Portal branch: `draft/cs-overview`.
- **Recovery push:** preview and send, eligibility ladder (will send, weekly limit, just bought, not abandoned yet, push off, guests, suppressed), order re-check at send time, English/Khmer text, deep link to unfinished checkout.
- **Weekly marketing push cap:** decided in Push v3: 1 a day and 4 in any 7 days, quiet hours 9 PM–8 AM. Journey UI still shows 3 and changes to 4.
- **Audiences:** destinations Push, Meta Custom Audience (daily sync), TikTok, CSV; unique names. Hidden in portal (`ENABLE_JOURNEY_AUDIENCES=false`).

## Later
- **Meta Ads import:** clicks, impressions, spend per campaign and ad set per day. Needed for the Ad clicked stage, campaigns compare and CAC. Today `meta_not_connected`.
- **AI customer summary:** what happened, next step, best offer, English/Khmer, no phone or email in the prompt, Refresh.
- **Clarity recording link per session:** needs an app-side session tag.
- **Zone mapping:** map `area_cell` to zone, or return `zoneId` in the address API.
- **Full phone in journey APIs (moved from pending 6.17, 20 Oct):** return a full `phone` on people rows, `GET /customers/{id}/summary` and registration rows only for callers with `MARKETING_CUSTOMER:VIEW`, keep `phoneMasked` for others, audit-log it. Confirm retention and display for phones saved at OTP request by people who never registered (matrix N2), and that `q` normalises `012…`, `+855 12…`, `85512…`. Task: `Tasks/1-backlog/Portal-journey-phone-reveal.md`.
- **Deferred journey items (dropped as tasks 30 Oct 2026):** assisted conversion identifier and window, finance amounts (refund, fee kept), consent and suppression data, people-list filters (last activity, furthest stage, customer type, value, CS status), and written "recoverable" eligibility rules with a list matching the count. Reopen as tasks when product decides.
