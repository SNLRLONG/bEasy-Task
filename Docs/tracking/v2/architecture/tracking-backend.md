**bEasy Engineering**

**bEasy Customer Journey Tracking Backend and Mobile Integration Review**

Implementation contract and staging sign off guide

| **Review item** | **Current value** |
| --- | --- |
| Audience | Backend engineers mobile engineers QA and technical leads |
| Reviewed branch | feature/bunlong-tracking |
| Review date | 4 October 2026 |
| Source requirement | bEasy Customer Journey Tracking Developer Functional Specification v2 |
| Current decision | Backend contract ready for mobile review and staging validation |

This document explains how the current backend connects mobile Firebase Analytics events, GA4 BigQuery export, authenticated API headers and authoritative order lifecycle events. It also defines the exact work the mobile team must review before both sides can be marked as matched.

The current backend changes are designed not to block registration, login or order creation when tracking fails. Automated tests pass, but production safety still requires the migration, BigQuery configuration, scheduler and an end to end staging test. The reviewed changes are currently in the local working tree and this document does not prove that they are deployed.

# **1 Review Conclusion**

The backend implementation is ready to be reviewed against the mobile Firebase implementation. It is not yet possible to declare the two implementations fully matched because the mobile source code was not available in this workspace. Mobile must confirm the event names, parameter names, identifier lifecycle and API headers in this document, then both teams must run the staging sign off test.

| **Area** | **Current result** | **Meaning** |
| --- | --- | --- |
| Order path safety | Verified by tests | Tracking runs after commit and does not hold the order response open |
| Authentication safety | Implemented | Tracking calls are dispatched without awaiting database tracking writes |
| Mobile Firebase contract | Backend defined | Mobile code still needs review against the exact contract |
| BigQuery ingestion | Implemented in code | Production export credentials dataset and schedule still need confirmation |
| Backend lifecycle truth | Implemented | Paid cancelled and completed events come from order data |
| Full boss brief | Partially complete | Data foundation exists but funnel interest audiences and automated recovery remain future work |

# **2 Current System Flow**

| **Step** | **Owner** | **What happens** | **Reason** |
| --- | --- | --- | --- |
| 1 | Mobile | Creates stable journey and registration identifiers and logs exact Firebase event names | Preserves the customer path across screens |
| 2 | Firebase | Stores mobile analytics events and exports them through GA4 to BigQuery | Keeps high volume interaction tracking outside the order request |
| 3 | Mobile API client | Sends the same identifiers in authentication and order request headers | Links anonymous activity to the authenticated user and order |
| 4 | Backend API | Completes authentication or creates the order without waiting for tracking persistence | Tracking failure must not break a customer operation |
| 5 | Order model | Dispatches purchase cancellation and completion tracking only after the database commit | Prevents events for rolled back orders |
| 6 | Scheduler | Imports recent BigQuery events and reconciles recent authoritative order states | Deduplicates events and recovers missed background writes |
| 7 | Admin API | Returns the chronological event log for a customer | Supports CRM review and later funnel work |

# **3 Why Existing Order Features Should Remain Safe**

The order controller performs normal booking validation before using tracking context.

An invalid or missing tracking header becomes null and does not reject the order request.

The journey identifier is optional and is added only to primary orders created through the version 6 order endpoint.

Identity linking is started without awaiting it, so a slow tracking table does not extend order response time.

Paid cancelled and completed tracking runs after the transaction commits. Tracking cannot cause an already committed order to roll back.

Tracking services catch and log their own failures and return null instead of throwing into customer flows.

The order journey index is non unique, so legitimate related records are not rejected because they share a journey.

The scheduled reconciliation repeats recent authoritative order checks with deterministic event identifiers, so a retry does not create duplicate lifecycle events.

This is strong protection against regression, but it is not an absolute production guarantee. Database migration, deployment configuration and real payment callbacks must still be tested in staging before release.

# **4 Identifier Contract**

The identifiers below are the most important part of the mobile and backend agreement. The mobile application must create and reuse them exactly as described.

| **Identifier** | **Format** | **Created by** | **Lifecycle and usage** |
| --- | --- | --- | --- |
| event_id | UUID version 4 string | Mobile | Create a new value for every Firebase event. Never reuse one value for two meaningful events |
| journey_id | UUID version 4 string | Mobile | Create when one service booking journey starts. Reuse through the same journey and send it on the version 6 order create request |
| reg_flow_id | UUID version 4 string | Mobile | Create when registration starts. Reuse through OTP sign up and registration completion |
| user_id | Backend user ID as a string | Backend then mobile | Call Firebase Analytics setUserId after login or registration. Clear it on logout or account switch |
| user_pseudo_id | Firebase Analytics app instance ID | Firebase SDK | Automatically exported to BigQuery. The API x app instance id header must use the same Analytics app instance ID |
| session_id | GA4 session identifier | Firebase SDK | Automatically exported as ga session id and stored for event log grouping |
| service_id | Positive backend category ID | Mobile from backend catalog | Use the stable numeric service key. Do not send a translated service name as the identifier |
| seq | Integer zero or greater | Mobile | Increase in event order within the local flow so ordering remains clear when timestamps are close |
| schema_version | Positive integer | Mobile | Send 1 for the current contract and increase only after an agreed schema change |

# **5 API Header Contract**

| **Header** | **Required on** | **Value** | **Backend behavior** |
| --- | --- | --- | --- |
| x-app-instance-id | Authentication and version 6 order calls | Firebase Analytics app instance ID | Links pre login BigQuery events to the authenticated user |
| x-reg-flow-id | Registration and OTP calls | Current registration UUID version 4 | Groups backend registration milestones without blocking authentication |
| x-journey-id | POST /api/customer/order/v6/create | Current service journey UUID version 4 | Stores the journey on the primary order and later lifecycle events |
| x-campaign-code | Authentication when campaign context exists | Campaign code up to 100 characters | Creates an attribution touch after a user is known |

Header names are case insensitive over HTTP. Invalid optional tracking headers are ignored. The customer request continues with normal business validation.

# **6 Authentication Endpoint Mapping**

| **Endpoint** | **Headers to send** | **Backend tracking milestone** |
| --- | --- | --- |
| GET /api/customer/auth/account/{username}/status/check | x-app-instance-id and x-reg-flow-id | PHONE_STATUS_CHECKED |
| POST /api/customer/auth/send/otp | x-app-instance-id and x-reg-flow-id | OTP_SENT |
| POST /api/customer/auth/account/login | x-app-instance-id and x-reg-flow-id | OTP_VERIFIED for pending users or LOGIN_COMPLETED for existing users |
| POST /api/customer/auth/account/create | x-app-instance-id x-reg-flow-id and optional x-campaign-code | ACCOUNT_CREATED plus identity link and optional campaign attribution |
| POST /api/customer/auth/account/auto-login | x-app-instance-id and optional x-reg-flow-id | LOGIN_COMPLETED and identity link |

# **7 Firebase Event Contract**

Firebase event names and parameter keys must use lower snake case exactly as shown. The backend maps them to uppercase CRM event names during import.

| **Event group** | **Accepted Firebase event names** |
| --- | --- |
| Discovery | app_opened service_list_viewed service_viewed |
| Booking | booking_started address_selected service_option_selected duration_selected date_selected time_selected booking_details_completed slot_selected booking_summary_viewed promo_applied checkout_started payment_started order_confirmation_viewed |
| Registration | registration_started otp_viewed otp_submitted otp_failed otp_resend_clicked otp_verified signup_viewed registration_completed |
| Add ons | addon_group_viewed addon_selected addon_quantity_changed addon_removed task_info_viewed equipment_viewed |
| Pair service | pair_service_viewed pair_service_selected pair_service_added pair_service_removed |
| Order and payment reports | order_created payment_failed purchase |
| Notification | push_opened notification_inbox_viewed notification_detail_viewed |

# **8 Common Firebase Parameters**

| **Parameter** | **Requirement** | **Validation or use** |
| --- | --- | --- |
| event_id | Required for every accepted event | UUID value used for deduplication |
| schema_version | Required for every accepted event | Positive integer current value 1 |
| seq | Required for every accepted event | Integer zero or greater |
| journey_id | Required for service journey events | Valid UUID and the same value used on order creation |
| service_id | Required for service journey events | Must be an existing backend category ID |
| reg_flow_id | Required for registration events | Valid UUID shared across one registration attempt |
| product_id | Optional | Stored only when it matches an existing backend product |
| product_option_id | Optional | Stored only when it matches an existing backend product option |
| order_id | Optional mobile report | Attached when it matches an existing backend order |
| value | Required for values used by CRM | booking_details_completed becomes estimated value and checkout_started becomes booking value |
| currency | Recommended with value | Three letter code and defaults to USD |
| slot_start | Required for slot_selected | Parseable date and time value |
| campaign_code utm_source utm_medium | When acquisition context exists | Preserved in event metadata |
| notification_id send_id action_id | For push and notification events | Connects sends opens and actions |

# **9 Core Event Requirements**

| **Firebase event** | **Required business parameters** | **Notes** |
| --- | --- | --- |
| service_viewed | journey_id service_id | Start or continue a service journey |
| booking_started | journey_id service_id | Marks booking intent |
| booking_details_completed | journey_id service_id value currency | Value is stored as estimated value |
| slot_selected | journey_id service_id slot_start | Use the selected booking date and time |
| checkout_started | journey_id service_id value currency | Value is stored as booking value |
| purchase | event_id and optional order context | Imported only as PURCHASE_REPORTED and never treated as authoritative payment |
| registration_started through registration_completed | reg_flow_id | Use one registration flow ID across all registration events |
| push_opened | notification_id send_id action_id | Preserves campaign action context when present |

# **10 Source of Truth Rules**

| **Data** | **Authoritative source** | **Current behavior** |
| --- | --- | --- |
| Browsing and screen interactions | Mobile Firebase events | Imported from GA4 BigQuery after user identity can be resolved |
| Order created report | Mobile report plus backend order records | Mobile event is useful for UX analysis but backend order data controls real order existence |
| Payment and purchase | Backend order payment status | PAID creates PURCHASE_COMPLETED after commit |
| Cancellation | Backend order status | CANCELLED creates ORDER_CANCELLED after commit |
| Service completion | Backend order status | COMPLETED creates SERVICE_COMPLETED using the actual status update time |

The mobile purchase event is intentionally recorded as PURCHASE_REPORTED. It can show that the confirmation screen fired, but it cannot prove that payment settled. The backend PURCHASE_COMPLETED event controls conversion reporting.

The importer does not accept payment_success booking_confirmed order_cancelled or service_completed as mobile lifecycle facts. Payment success cancellation and service completion must come from backend order state.

# **11 Anonymous Identity Linking**

1.  Before login the mobile app logs events with Firebase user pseudo ID and the current journey or registration flow ID.

2.  The mobile app sends that same Analytics app instance ID as x-app-instance-id when authentication succeeds.

3.  The backend stores a link between the app instance and the backend user ID without delaying authentication.

4.  The scheduled importer uses the direct Firebase user ID when available or the stored app instance link when it is not.

5.  Events that arrive before the identity link may be skipped during one import. The default three day lookback allows a later scheduler run to import them after the link exists.

Mobile must verify that x-app-instance-id is the Firebase Analytics app instance ID represented as user_pseudo_id in BigQuery. A different installation identifier will prevent anonymous activity from linking to the user.

# **12 BigQuery and Scheduler Requirements**

| **Configuration** | **Requirement** |
| --- | --- |
| GA4_BIGQUERY_PROJECT_ID | Google Cloud project containing the GA4 export or use the existing GCS project fallback |
| GA4_BIGQUERY_DATASET_ID | GA4 export dataset name or derive it from GA4_PROPERTY_ID |
| GA4_BIGQUERY_LOCATION | Optional dataset location when required |
| PLAYSTORE_GCS_KEY_FILE | Optional credential key file used by the current Google Cloud client configuration |
| GA4 export | Firebase Analytics property must export daily events tables to BigQuery |
| Scheduler | POST /api/scheduler/customer-activity-import with the existing Cloud Scheduler authentication |
| Lookback | Defaults to 3 days and accepts 1 through 30 days in the request body |

# **13 Import Validation and Quarantine**

Duplicate event IDs are ignored by a unique database index.

Invalid event IDs schema versions sequence numbers journey IDs registration flow IDs service IDs timestamps and slot times are quarantined for investigation.

Unknown Firebase event names are skipped.

Events with no resolvable registered user are skipped and may be picked up by a later lookback import after identity linking.

Raw Firebase identifiers and useful campaign device booking and notification fields remain in metadata for audit and debugging.

# **14 Backend and Mobile Matching Matrix**

| **Contract item** | **Backend status** | **Mobile confirmation needed** | **Overall status** |
| --- | --- | --- | --- |
| Exact event names | Accepted mapping implemented | Confirm all emitted names match the list | Review needed |
| Event UUID | Required and deduplicated | Confirm a new UUID version 4 per event | Review needed |
| Journey UUID | Validated stored on primary order | Confirm stable reuse across one service journey | Review needed |
| Registration flow UUID | Validated and grouped | Confirm reuse across OTP and sign up | Review needed |
| Service ID | Validated against categories | Confirm numeric backend category ID is sent | Review needed |
| Firebase user ID | Direct user lookup supported | Confirm setUserId after authentication and clear on logout | Review needed |
| Anonymous link | App instance link implemented | Confirm header equals Analytics user pseudo ID | Review needed |
| Version 6 order header | Journey accepted without blocking order | Confirm x-journey-id is sent on create | Review needed |
| Authoritative lifecycle | Backend implemented | No mobile action beyond normal API use | Matched by design |
| BigQuery export | Importer implemented | No mobile code action after Firebase setup | Environment check needed |

# **15 Mobile Review Checklist**

☐ Firebase event names match Section 7 exactly

☐ event_id is a new UUID version 4 for every event

☐ schema_version is 1 and seq is a nonnegative integer

☐ journey_id remains unchanged through one service booking attempt

☐ service_id is the backend category ID and not a display name

☐ reg_flow_id remains unchanged through registration OTP and sign up

☐ Firebase Analytics setUserId uses the backend user ID after authentication

☐ Firebase user ID is cleared on logout and account switch

☐ x-app-instance-id matches BigQuery user_pseudo_id

☐ x-reg-flow-id is sent to registration and OTP endpoints

☐ x-journey-id is sent to POST /api/customer/order/v6/create

☐ checkout_started sends value and currency

☐ slot_selected sends a valid slot_start date and time

☐ push_opened preserves notification_id send_id and action_id

☐ No full street address or sensitive personal value is logged to Firebase

# **16 Backend Release Checklist**

☐ Review and commit the current branch changes

☐ Run migration 20261004090000-create-customer-journey-tracking.js

☐ Confirm the GA4 BigQuery project dataset location and credentials

☐ Configure Cloud Scheduler for the customer activity import endpoint

☐ Confirm application logs alert on import and reconciliation failures

☐ Run the staging sign off procedure in Section 17

☐ Verify one paid one cancelled and one completed order lifecycle

☐ Confirm the admin customer activity endpoint returns both FIREBASE_BIGQUERY and BACKEND sources

# **17 Joint Staging Sign Off Procedure**

1.  Mobile records its Firebase Analytics app instance ID and creates one new journey UUID and one new registration flow UUID for the test.

2.  A test user opens one service and logs service_viewed booking_started booking_details_completed slot_selected and checkout_started with the required parameters.

3.  If registration is required the user completes registration while mobile sends x-app-instance-id and x-reg-flow-id on the authentication calls.

4.  After authentication mobile calls Firebase Analytics setUserId with the backend user ID.

5.  Mobile creates the order through POST /api/customer/order/v6/create and sends the same journey value in x-journey-id.

6.  The test payment reaches PAID through the normal staging payment path. Do not manually insert a tracking event.

7.  Run or wait for POST /api/scheduler/customer-activity-import and capture fetched accepted inserted quarantined skipped and reconciliation counts.

8.  Call GET /api/admin/customer/{userId}/activity with an authenticated admin account.

9.  Confirm the mobile events have source FIREBASE_BIGQUERY and the paid event has source BACKEND.

10.  Confirm every service event and the primary order use the same journey ID and valid service ID.

11.  Repeat the scheduler import and confirm duplicate events are not added.

12.  Test cancellation and service completion through normal order flows and confirm their backend events use the status transition time.

# **18 Expected Staging Evidence**

| **Evidence** | **Pass condition** |
| --- | --- |
| Order API response | Normal success response and no added delay caused by tracking persistence |
| Order row | Primary order contains the mobile journey UUID |
| Customer activity | Expected Firebase events appear in chronological order |
| Purchase event | PURCHASE_COMPLETED comes from BACKEND with the final order value |
| Idempotency | A repeated import does not increase the same event count |
| Anonymous link | Pre registration events appear under the final backend user after the lookback import |
| Quarantine | No unexpected invalid journey service event or schema errors |
| Existing features | Order creation payment cancellation completion notifications and assignment behave as before |

# **19 Automated Test Evidence**

| **Test command** | **Result on 4 October 2026** | **Coverage** |
| --- | --- | --- |
| npm run test:customer-activity | 6 suites and 8 tests passed | Nonblocking hooks migration reconciliation timestamps headers and scheduler coordination |
| npm run test:instant | 18 active suites and 597 tests passed | Instant booking transaction routing pricing contracts scheduling and related order behavior |
| npm run test:audit | 2 active suites and 39 tests passed | Existing request audit and list behavior |

These results show no known regression in the covered areas. They do not replace staging validation for external Firebase GA4 BigQuery Cloud Scheduler payment gateway and production database configuration.

# **20 Requirement Coverage**

| **Boss brief requirement** | **Current status** | **Remaining work** |
| --- | --- | --- |
| Reliable event schema service journey identifiers and deduplication | Implemented | Mobile contract review and staging proof |
| Anonymous to registered linking | Implemented foundation | Confirm privacy consent and mobile app instance identifier |
| Backend source of truth for order lifecycle | Implemented | Staging proof across real transitions |
| Customer chronological activity endpoint | Implemented foundation | Admin user interface grouping and filters |
| Furthest step per journey and service | Not implemented | Calculation and display |
| Journey status active abandoned converted cancelled completed | Not implemented | Rules thresholds and storage or query layer |
| Unique journey funnel conversion and drop off | Not implemented | Aggregation API and dashboard |
| Interest from furthest step recency and repeat sessions | Not implemented | Classification logic and CRM display |
| CRM audience builder | Not implemented | Saved segment behavior time frequency and exclusion logic |
| Recovery notifications and attribution | Partial identifiers only | Eligibility checks frequency caps deep links campaigns and reporting |

# **21 Known Risks and Limits**

The current branch contains local working tree changes. Release management must commit review and deploy them before the behavior exists in an environment.

In process background dispatch can be interrupted by a process restart. Scheduled order reconciliation reduces this risk but a transactional outbox and worker would provide a stronger long term guarantee.

The importer depends on a working GA4 BigQuery export and correct credentials. Backend lifecycle tracking can work without BigQuery but mobile interaction history cannot.

Only the version 6 create order path currently accepts the mobile journey header. A mobile client using version 5 or the mini app order path will not attach journey context through this controller.

The backend discards unresolved users during an import run. The lookback retries them after identity linking, so the schedule must cover enough time for delayed registrations.

Full customer journey business features remain incomplete even though the data foundation is strong.

# **22 Recommended Sign Off Decision**

Backend review can be approved for staging after the migration and environment configuration are checked. Mobile integration should be approved only after every item in the mobile checklist is confirmed and the joint staging procedure produces the expected evidence. Production release should wait until one complete service journey is visible from Firebase interaction through backend purchase without affecting the normal order result.

# **23 Code Evidence Reviewed**

| **Area** | **Repository path** |
| --- | --- |
| Firebase BigQuery importer | services/customer/importCustomerActivityFromBigQuery.js |
| Tracking headers identity and registration | services/customer/trackingHeaderService.js |
| Authoritative lifecycle events | services/customer/customerActivityEventService.js |
| Order reconciliation | services/customer/reconcileCustomerActivityOrders.js |
| Scheduler endpoint | controllers/scheduler/importCustomerActivity.js |
| Version 6 order creation | controllers/customer/order/v6/createBulkOrder.js |
| Order hooks | models/order.js |
| Database migration | migrations/20261004090000-create-customer-journey-tracking.js |
| Admin activity endpoint | controllers/admin/customer/getCustomerActivity.js |
| Tracking tests | tests/customer/activity |

| Customer Journey Tracking | Backend and Mobile Review |  |
| --- | --- | --- |
# **24 Addendum, 8 October 2026: gaps against the UI v2.1 design**

This review dated 4 October predates the approved admin UI ([Journey_Tracking_UI_Flow_v2.1.md](../requirements/Journey_Tracking_UI_Flow_v2.1.md)). Sections 1 to 23 are unchanged. The table lists what the new UI needs that this contract does not yet provide. Detail and field lists are in the architecture guide section 11.

| **UI v2.1 need** | **Current contract** | **Gap** |
| --- | --- | --- |
| Guest and device lists (registration funnel, acquisition devices, Customer type = Guest) | Section 13: events with no resolvable registered user are skipped | **Blocker.** Keep guest events keyed by app instance ID and merge them at identity link |
| Per-step abandon rules and Abandoned status | Journey furthest step and status listed as not implemented (section 20) | Rule table, sweep job, purchase clears status at once |
| OTP requested stage and Why OTP fails | OTP_SENT is recorded | Store gateway code, delivery status, wrong attempts, lock and resend count, and the phone number for people who never register |
| Payment failed then paid | payment_failed accepted from the mobile app | Backend writer from payment records, linked to the retry |
| Auto promo versus manual code | promo_applied only from mobile | Source field and a backend record when the system applies a promo |
| Reschedule, refund, cancellation fee | order_cancelled and service_completed only; mobile order_rescheduled is skipped | Backend events for reschedule (old and new slot, fee), refund (amount, destination), cancellation fee fields |
| Event log rows for banner and search | banner_clicked is skipped; search events do not exist | Whitelist decision, see matrix N5 |
| Recovery push with live eligibility | Push IDs deferred to the notification phase | Decision service with order re-check, weekly cap, suppression, send ledger |
| CS inbox, contact log, CS overview | None | New tables and endpoints |
| Meta and TikTok audiences, ad clicks | None | Daily sync job, ad stats import |
| Clarity link per session, AI summary | None | Session recording table, summary service |

Requested items are also listed in [backend-request/pending-request-backend.md](../backend-request/pending-request-backend.md) sections 3 to 5.
