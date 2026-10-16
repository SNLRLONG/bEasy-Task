# Project Task Tracker

> **Estimates** are in working days of client-side (front-end) work, including a small buffer for QA and fixes.
> They don't include backend work, design changes, or time spent waiting on blocked items.

## bEasy App

| # | Task / Deliverable | Priority | Status | Est. (days) | Remarks |
|---|---|---|---|---|---|
| 1 | Customer Journey Tracking | High | Not Started | 5 | Work along with backend. Add events across the booking/order flows (see `bEasy-app/docs/`) |
| 2 | UI checkup for inconsistencies | Low | Not Started | 3 | Audit and fix spacing, typography and components |
| 3 | App Onboarding adjust flow & UI | Medium | Not Started | 4 | [Figma](https://www.figma.com/design/FNgvlTj2AAal3I5QZ1bJ6I/bEasy-Revamp?node-id=2-593&t=vqbBOIDXVBREVvs2-4) |
| 4 | Cancellation & Reschedule | Medium | Blocked | 5 | [Figma](https://www.figma.com/design/Q6phvltGCcVJK41XlOdE7c/bEasy---v3?node-id=324-2819&t=Ura0BD4WXS7By0WG-4). Some reschedule logic already exists (`order_detail_reschedule_logic.dart`) |
| 5 | Quotation | Low | Blocked | ~7 (TBD) | Not yet clearly defined. Re-estimate once the scope is confirmed |
| 6 | Referral flow | High | Not Started | 5 | [Figma](https://www.figma.com/design/Q6phvltGCcVJK41XlOdE7c/bEasy---v3?node-id=142-4687&t=Ura0BD4WXS7By0WG-4) |
| 7 | Location behavior adjustment | Medium | Not Started | 3 | [Figma](https://www.figma.com/design/OwAMYCTuEnZu6a9yd4blFA/bEasy---v2.?node-id=2222-6153&t=p83e6f3zfFcJBejV-4) |
| 8 | Prompt user to register and get $5 credit | High | Not Started | 2 | |
| 9 | Reference Source API integration | High | Not Started | 1 | `customer_source_screen.dart` already exists |
| 10 | Company Contact Info GET API | High | Not Started | 1 | |
| | **Subtotal** | | | **~36** | ~29 without the blocked Quotation task |

## bEasy Portal

| # | Task / Deliverable | Priority | Status | Est. (days) | Remarks |
|---|---|---|---|---|---|
| 1 | Support CMS data overview & tracking display UI | High | In Progress | 5 | Total effort, including work already done |
| 2 | Pull tracking data from Firebase by category and show it in the portal | High | In Progress | 4 | |
| 3 | Push notification segmentation UI adjustment & API integration | – | Not Started | 4 | Priority not set yet |
| | **Subtotal** | | | **~13** | |

## bEasy Landing Page

| # | Task / Deliverable | Priority | Status | Est. (days) | Remarks |
|---|---|---|---|---|---|
| 1 | Configure deeplinks for all categories and add them to the website | Medium | Not Started | 2 | |
| 2 | Consider adding cookies (consent) | Low | Not Started | 1 | Consent banner + gate analytics on it |
| 3 | Figure out what can be fetched from the API | Medium | Not Started | 2 | Investigation. Follow-up work gets estimated afterwards |
| 4 | Tag all image assets | Medium | Not Started | 2 | alt/title/SEO tagging, see `IMAGE_SEO_GUIDE.md` |
| | **Subtotal** | | | **~7** | |

## Fresh Plush

| # | Task / Deliverable | Priority | Status | Est. (days) | Remarks |
|---|---|---|---|---|---|
| 1 | Continue working on the portal, PWA & landing page | High | Not Started | TBD | Too broad to estimate. Break it into tasks first |

## Summary

| Project | Est. (days) |
|---|---|
| bEasy App | ~36 (~29 without Quotation) |
| bEasy Portal | ~13 |
| bEasy Landing Page | ~7 |
| Fresh Plush | TBD |
| **Total** | **~56** (one developer, done in sequence) |
