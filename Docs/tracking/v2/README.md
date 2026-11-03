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
- `App_Tracking_Coverage_Spec.md` — audit of every app screen and tap, new generic events, backend tables/APIs and portal App Usage pages · `Registration_And_Location_Flow_Spec.md` — registration funnel order vs the app, where each registration came from, location flow events · `Cancellation_And_Reschedule_Spec.md` — cancel/reschedule tracking, backend records and portal decisions · `Instant_Cleaning_Tracking_Spec.md` — Instant Cleaning funnel, rooms, availability, schedule errors, extend · `App_Interaction_Inventory.md` — generated per-file handler/tracking counts
- `App_Tracking_AB_Regression_Plan.md` — A/B regression plan (main vs `feat/tracking-integration`, dev flavor): setup, scripted cases by area, highest-risk list, sign-off, optional DebugView (T-0073)
- `Meta_App_Events_Setup.md` — runbook and findings for verifying Meta app events (T-0031)

**api/**
- `Journey_Backend_API_Spec.md` — every endpoint, request and response

**backend-request/**
- `pending-request-backend.md` — open requests · `backlog.md` — later · `Campaign_Source_Request.md` — Organic vs campaign · `Journey_Acquisition_API_Request.md` — acquisition contract · `Marketing_Suite_Backend_Request.md` — push, promotions, content · `App_Tracking_Backend_Delta_3Nov2026.md` — changes since the 2 Nov handoff · `App_Tracking_Backend_Handoff_3Nov2026.md` — single extract for backend: usage events, registration, location, cancel/reschedule, Instant

**backend-response/**
- `Journey_V2_Portal_Review.md` — backend review of 14 Oct (what is built, counting rules, guest rules)
- `2026-10-19-journey-list-filters.md` — Status on Reached and Converted filter are live
- `CS_Overview_Backend_Answers_31Oct2026.md` — answers to our CS Overview questions: permissions now enforced, employee list, batch segment counts
- `CS_Overview_Frontend_Handoff_29Oct2026.md` — CS Overview report, targets, people lists, export (`/cs-overview*`); what is not built

**archive/**
- `Journey_Tracking_API_Contract.md` — early mock-based contract, replaced by `api/`

Also: export-report docs live in `bEasy-portal-export/docs/`, BigQuery setup in `lsdb-bEasy/bigquery-setup-guide.md`, portal plan in `bEasy-portal/TODO.md`.

**Scope (31 Oct 2026):** the portal builds Customer Journey + Customer page, and now the CS Overview report (branch `feature/cs-kpi`, contract in `backend-response/CS_Overview_Frontend_Handoff_29Oct2026.md`). The CS inbox, pick-up and contact-log screens are still parked; `/cs/*` returns 501.

**Where to put new docs:** a request to backend → `backend-request/`; a reply from backend → `backend-response/`; a finished or replaced doc → `archive/`.
