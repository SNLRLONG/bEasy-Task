# CS Overview: what we need from backend

**To:** Backend · **From:** Portal · **Updated:** 2 Nov 2026
**Environment checked:** `api-dev.beasy.info`, admin token, read-only calls
**Related:** [Frontend handoff 29 Oct](../backend-response/CS_Overview_Frontend_Handoff_29Oct2026.md) · [Your answers 31 Oct](../backend-response/CS_Overview_Backend_Answers_31Oct2026.md)

## Summary

The CS Overview page is built and works against dev. It shows real numbers where your API returns them, and "Not implemented yet" where `availability` says the feature does not exist. The page is mostly empty because the data behind it is not recorded yet, not because of a bug on either side.

We can close the portal task as it is. For the page to be useful we need **contact capture (ask 1)** and **CS case storage (ask 2)**. Ask 6 (permissions) blocks us from testing the target editor. Asks 3 to 8 are smaller.

Please reply per ask number with: **yes / no / later**, the route and fields if they differ from our proposal, and a date.

## What we see on dev today

| Area | Result |
|---|---|
| `POST /customer/counts` | Works. Thank you. Saved-segment cards show real counts. |
| `GET /cs-overview` (October 2026) | 7 contacted customers, 1 first sale ($18.00), repeat 0 of 4. 3 of the 7 contacts have no employee ("Employee not recorded"). |
| `GET /cs-overview` (November 2026) | No contacts yet (new month). |
| Today's follow-up work (waiting, not contacted, due today, overdue, on-time %) | `not_implemented`. The portal shows "—". |
| Response time | Not in the API. The portal shows "—". |
| Problems found by CS | `out_of_scope`. The portal shows a placeholder. |
| `GET /cs-overview/targets/:month` | 403 for the test user `cs-overview`, although summary and people return data for the same user (see ask 6). With an admin token it returned the defaults 30 / 30 / 90 on 31 Oct. |

## Asks

### 1. Contact capture (most important)

**Problem:** first sales, repeat sales and "contacted" are credited from retained legacy records, so there are few of them and some have no employee. The designed Log call and chat replies have nowhere to be saved.

**Ask:** store every contact as an immutable record (never overwritten) and tell us the route and body to write it.

Proposed fields (from the design notes):

| Field | Values |
|---|---|
| employee | the CS user who made the contact |
| customer | customer or registered-customer id |
| contactedAt | date and time |
| purpose | `sales_follow_up` or `booking_confirmation` (only the first counts as CS contact for sales credit) |
| channel | call, SMS, Telegram, bEasy chat, Meta (Facebook, Instagram, Messenger), other |
| outcome | `no_answer`, `spoke_to_customer`, or `replied` for messages |
| nextFollowUpAt | optional; this is the follow-up deadline |
| problem | optional type and remark, only if ask 4 is yes |

Chat messages in bEasy chat should be saved automatically; calls outside the app come from Log call.

**Questions:** can chat replies be captured without a mobile release? Can the `cs-overview` summary use the new records from the day they exist, keeping the legacy records for earlier months?

### 2. CS case storage and the Today numbers

**Problem:** the four Today cards and the per-employee Today table cannot be filled.

**Ask:** store cases (customer, source group Registered or Existing, owner, picked-up time, deadline, closed time) with these transitions: **pick up** (first person wins; a second person gets "already picked up by X"), **log contact** (with next follow-up keeps it open, without closes it), **close**. No assignment, reassign or release (product decision of 30 Oct).

Then add to `GET /cs-overview` (names are a proposal):

```json
"today": {
  "waitingToBePickedUp": 9,
  "notContacted": 23,
  "dueToday": 17,
  "overdue": 6,
  "completedOnTimeRate": 0.89,
  "byEmployee": [
    { "csId": "…", "openRegistered": 16, "openExisting": 9, "notContacted": 4,
      "dueToday": 5, "overdue": 1, "completedOnTimeRate": 0.92,
      "attempts": 88, "reached": 61 }
  ]
}
```

Rates are 0 to 1, like the existing fields. `null` means unavailable, never 0. `completedOnTimeRate` is the share of closed cases this month whose last contact was before the deadline. Every number must open a list, so please also tell us the list routes (task list filtered by employee and status).

### 3. Response time

Averages per employee and team, in minutes, for the month:

- list to first contact (from when the customer enters "Needs follow-up")
- pick-up to first contact
- customer chat message to first CS reply
- number of cases picked up this month

`null` when there is not enough data.

### 4. Decision: Problems found by CS

The design (30 Oct) says it is back; your `availability.problemsFound` says out of scope. **Which is it?** If it is back: counts per problem type and per employee for the month, and a list route (one row per problem with type, remark, who logged it, channel).

### 5. `doubleContacted`

Please add the number of customers contacted by two or more employees to the first-sales summary. The page footnote needs it ("10 customers were contacted by two people, so the rows add up to 430 but the team total is 420").

### 6. Permissions: one user, two different answers

Thank you for the server-side checks (answer B1). On dev we see this for the same user (login `cs-overview`, role `CUSTOMER_SERVICE`, `CS-OVERVIEW` shown in the sidebar):

- `GET /cs-overview` and `GET /cs-overview/people` return data.
- `GET /cs-overview/targets/:month` returns **403** `CS-OVERVIEW VIEW permission is required`.

Both should need the same VIEW permission, so one of them is not behaving as in B1. Please check which is right on dev:

- Is B1 deployed on dev for summary and people, or only for targets?
- How does the permission lookup read this user's role (`user.role` or `user.type`)? Which role name should carry the `CS-OVERVIEW` grant for `CUSTOMER_SERVICE` users?

Until this is clear we cannot test saving targets (`PUT`, needs UPDATE). Please also answer: does `POST /customer/counts` need `CS-OVERVIEW` VIEW, or the same permission as `customer/existed`?

### 7. Contracts still open from your handoff

- **Export list:** a per-list export so a number's list can be downloaded. We will keep the button disabled until this exists.
- **Repeat bookings status tiles** (completed, scheduled, refunded): the list that matches each tile.
- **Partial refund revenue rule:** how partial refunds reduce revenue.

### 8. Legacy contacts without an employee

Can old contacts with no employee be matched to a person (from chat or call logs)? If not, they stay in the "Employee not recorded" row, counted in the team total but in nobody's row.

### Also confirm (not blocking)

For `audience: "registered"` on `POST /customer/counts`, are saved filters such as `journey` and `contactDate` read the same way as on `GET /customer/register-customer`? We saw a possible difference between a segment's count and the Registered page's count.

## Not asking for

Assignment, reassign, release, personal KPIs on My cases, and anything about the Customers or Registered Customers pages. Those are out of scope for this request.
