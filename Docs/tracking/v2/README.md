# Customer Journey tracking — v2 (UI v2.1, 7 Oct 2026)

Start here. Read top to bottom for the full picture, or jump to the folder you need.

| Folder | What is in it | Read when |
|---|---|---|
| [requirements/](requirements/) | What we are building and why | You need the business rules or screens |
| [architecture/](architecture/) | How tracking works: mobile events, pipeline, tables, metrics | You touch the app, backend jobs or data |
| [api/](api/) | The portal ↔ backend contract (current) | You call or build an endpoint |
| [backend-request/](backend-request/) | What portal/mobile asks backend to do (open items only) | You are about to ask backend something |
| [backend-response/](backend-response/) | What backend replied and what is live | You want the real status |
| [archive/](archive/) | Superseded; do not follow | Only for history (v1 is in `../v1/`) |

## Files

**requirements/**
- `Journey_Tracking_UI_Flow_v2.1.md` — approved screens and rules (canonical for UI)
- `Journey_Tracking.md` — business / functional spec
- `Journey_Tracking_Flow_Breakdown.md` — use cases: event → pipeline → portal
- `Journey_Tracking_Requirements_Matrix.md` — traceability and conflicts N1–N14

**architecture/**
- `bEasy-tracking-architecture-guide.md` — events, tables, jobs (the main model)
- `tracking-backend.md` — backend + mobile integration contract, staging sign-off
- `Journey_Tracking_Analytical_Model.md` — metric definitions
- `bEasy-automation-engagement-guide.md` — segments, notifications, offers
- `Acquisition_Ad_Data_Readiness.md` — what blocks Meta/TikTok numbers on Acquisition and who owns each piece

**api/**
- `Journey_Backend_API_Spec.md` — every endpoint, request and response

**backend-request/**
- `pending-request-backend.md` — open requests · `backlog.md` — later · `Campaign_Source_Request.md` — Organic vs campaign · `Journey_Acquisition_API_Request.md` — acquisition contract · `Marketing_Suite_Backend_Request.md` — push, promotions, content

**backend-response/**
- `Journey_V2_Portal_Review.md` — backend review of 14 Oct (what is built, counting rules, guest rules)
- `2026-10-19-journey-list-filters.md` — Status on Reached and Converted filter are live

**archive/**
- `Journey_Tracking_API_Contract.md` — early mock-based contract, replaced by `api/`

Also: export-report docs live in `bEasy-portal-export/docs/`, BigQuery setup in `lsdb-bEasy/bigquery-setup-guide.md`, portal plan in `bEasy-portal/TODO.md`.

**Scope (9 Oct 2026):** the portal builds Customer Journey + Customer page only. CS Overview and the CS inbox are parked (branch `draft/cs-overview`); do not request or change `/cs/overview`, `GET /cs/cases` or `/cs/cases/{id}/pick-up` yet.

**Where to put new docs:** a request to backend → `backend-request/`; a reply from backend → `backend-response/`; a finished or replaced doc → `archive/`.
