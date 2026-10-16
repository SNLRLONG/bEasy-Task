# Backend Backlog: Customer Journey

Not requested yet. Moved here from the pending list (16 Oct 2026). The backend returns `501` for the CS and audience endpoints today.

## Parked until the portal builds it
- **CS inbox and overview:** create cases from a list, first picker owns, contact log (channel, result, said, problem, next follow-up), suppression on "no more calls", overview aggregates (30-day windows, first sale credited to the last sales follow-up). Portal branch: `draft/cs-overview`.
- **Recovery push:** preview and send, eligibility ladder (will send, weekly limit, just bought, not abandoned yet, push off, guests, suppressed), order re-check at send time, English/Khmer text, deep link to unfinished checkout.
- **Weekly marketing push cap:** shared across all marketing pushes. Value to confirm (UI shows 3).
- **Audiences:** destinations Push, Meta Custom Audience (daily sync), TikTok, CSV; unique names. Hidden in portal (`ENABLE_JOURNEY_AUDIENCES=false`).

## Later
- **Meta Ads import:** clicks, impressions, spend per campaign and ad set per day. Needed for the Ad clicked stage, campaigns compare and CAC. Today `meta_not_connected`.
- **AI customer summary:** what happened, next step, best offer, English/Khmer, no phone or email in the prompt, Refresh.
- **Clarity recording link per session:** needs an app-side session tag.
- **Zone mapping:** map `area_cell` to zone, or return `zoneId` in the address API.
