# CS Action: what the admin portal must implement (after backend 6 Nov)

**To:** Admin portal team · **From:** Backend · **Date:** 6 Nov 2026
**Backend branch:** `codex/cs-action-performance-v1` (commit `cdbf4bab`) · **Portal branch:** `feat/cs-action`
**Related:** backend reply `cs-action-frontend-response-6nov2026.md` (answers N1, N2, B1, B2). The 5 Nov guide is still valid for everything else.

All responses use the existing wrapper: `{ code:"SUCCESS", status:true, message:"Success", data }`. Errors: `{ status:false, message }` with the HTTP status. Show `message` as is.

---

## Checklist

| # | Task | Needs live API? | Priority |
|---|---|---|---|
| P1 | Complete button for **sales tasks** (and all tasks) built from `task.completeOptions` | No (sample data first) | High |
| P2 | Confirm dialog for **Sales decline** (permanent) | No | High |
| P3 | Hide management-only options; handle 403/409 | No | High |
| P4 | Resolution labels in history/records from `/performance/definitions` → `resolutions` | No | Medium |
| P5 | Permission gating with `CS_ACTION` / `CS_PERFORMANCE` (VIEW, UPDATE) | Role editor after SQL | High |
| P6 | Chat details save: show `csAction` result (task created / skipped) | After Chat branch merge | Medium |
| P7 | Turn off sample data after api-dev returns 200 | Yes | After deploy |
| P8 | Keep "Sync new orders" until scheduler is live | — | Keep |

---

## P1 Complete button from `completeOptions`

Every task returned by these endpoints now has `completeOptions`:

- `GET /api/admin/cs/actions` → `data.items[]`
- `GET /api/admin/cs/actions/:id` → `data.task`
- `POST /api/admin/cs/actions` → `data.task`
- `POST /api/admin/cs/actions/:id/transitions` → `data` (the updated task)

Example (open sales task):

```json
{
  "id": "42",
  "purpose": "sales_follow_up",
  "status": "in_progress",
  "version": 4,
  "ownerId": 7,
  "completeOptions": [
    { "resolution": "lost", "label": "Lost", "managementOnly": false,
      "meaning": "Ordinary lost opportunity. The customer can be followed up again later." },
    { "resolution": "declined", "label": "Sales decline (no further contact)", "managementOnly": false,
      "meaning": "Customer asked not to be contacted for sales. Permanent: future sales follow-ups are skipped." },
    { "resolution": "not_eligible", "label": "Not eligible", "managementOnly": true,
      "meaning": "Management closes work that should not be done." }
  ]
}
```

Rules:
- **Remove the "Complete disabled for sales tasks" block.** Show Complete when `completeOptions.length > 0`.
- Render one choice per option, using `label` as the text and `meaning` as helper text or a tooltip. **Do not hard-code the list**; it differs per purpose.
- `completeOptions` is `[]` when the task is completed. Hide Complete.
- The user must have taken the task (owner = current user) before completing. Otherwise the API returns 409 "Take this task before updating it". Keep the existing Take flow.
- **Never offer `converted`.** The server sets it by itself when it verifies a purchase. If it is sent anyway, the API returns 409 "Converted is set automatically when a verified purchase is found; choose lost or sales decline instead".

Options per purpose (for sample data and tests):

| purpose | completeOptions (in this order) |
|---|---|
| `sales_follow_up` | `lost`, `declined`, `not_eligible`* |
| `scope_confirmation` | `scope_verified`, `not_eligible`* |
| `review_request` | `not_eligible`* only. Received/declined come from the **contact** form (`reviewOutcome`), not from Complete. |
| `support` | `issue_resolved`, `not_eligible`* |

\* `managementOnly: true`

Request (unchanged contract):

```http
POST /api/admin/cs/actions/42/transitions
{ "action": "complete", "resolution": "lost", "expectedVersion": 4, "note": "Chose another company" }
```

Response: the updated task with `status:"completed"`, `resolution:"lost"`, `completeOptions:[]`. Replace the task in the list/detail cache with it (version changed).

`note` is required (max 2000 chars). Make the note field required in the Complete dialog.

## P2 Sales decline confirmation

`declined` on a sales task is **permanent**. The customer is suppressed, and every future sales follow-up for this customer is skipped with reason `declined`, from any segment or Chat. `lost` is not permanent.

- Before sending `declined`, show a confirm dialog, for example: "Customer asked not to be contacted for sales again. Future sales follow-ups for this customer will be skipped. Continue?"
- Style it as a warning, not as the default button. Make `lost` the first/default choice.

## P3 Management-only options and errors

- Hide options with `managementOnly: true` unless the user has `CS_PERFORMANCE / UPDATE` (or is ADMIN/SUPER_ADMIN). If sent without it, the API returns **403** "CS dashboard permission required".
- Error handling for the Complete dialog:

| HTTP | message (examples) | Portal action |
|---|---|---|
| 400 | `Choose an explicit resolution` / `Resolution does not match this task` / `note is required…` | Show inline; keep dialog open |
| 403 | `CS dashboard permission required` | Show; hide that option |
| 409 | `Task changed. Refresh before saving.` | Refetch the task, keep the user's note, ask to retry |
| 409 | `Take this task before updating it` | Show Take button |
| 409 | `Converted is set automatically…` / `Record the review outcome as a contact…` | Should not happen if you use `completeOptions`; show message |
| 409 | `Record a reached call and verify every current active service line first` (scope) | Point to the contact/checklist form |
| 409 | `Resolve the linked support ticket first; Chat closure is insufficient` (support) | Link to the ticket |
| 503 | `CS Action database setup is not installed or needs upgrade` | Show "CS Action not available on this server yet" |

## P4 Resolution labels everywhere

`GET /api/admin/cs/performance/definitions` → `data.resolutions` is the full catalog, including the server-only ones:

```json
"resolutions": {
  "sales_follow_up": [
    { "resolution": "converted", "label": "Converted", "manual": false, "managementOnly": false, "meaning": "Set automatically when a verified paid or deposit purchase is found. Not a manual choice." },
    { "resolution": "lost", "label": "Lost", "manual": true, "managementOnly": false, "meaning": "…" },
    { "resolution": "declined", "label": "Sales decline (no further contact)", "manual": true, "managementOnly": false, "meaning": "…" },
    { "resolution": "not_eligible", "label": "Not eligible", "manual": true, "managementOnly": true, "meaning": "…" }
  ],
  "review_request": [
    { "resolution": "review_received", "label": "Review received", "manual": false, … },
    { "resolution": "declined", "label": "Review declined", "manual": false, … },
    { "resolution": "not_eligible", "label": "Not eligible", "manual": true, … }
  ],
  "scope_confirmation": [ … ], "support": [ … ]
}
```

- Use it to show a completed task's `resolution` (task list, detail header, history, performance records/export).
- Look up the label by **purpose + resolution**: `declined` means "Sales decline" on a sales task but "Review declined" on a review task.
- This endpoint needs `CS_PERFORMANCE / VIEW`. For users without it (CS-only), use a local fallback map with the same labels as above.
- Show "Converted" with a small "verified purchase" hint, because staff did not choose it.

## P5 Permissions

After backend runs `sql/cs-action-permissions.sql`, the role editor (`GET /api/admin/rbac/permissions`) lists:

| Module | Actions | Portal use |
|---|---|---|
| `CS_ACTION` | `VIEW`, `UPDATE` | VIEW: CS Action menu, queue and task detail. UPDATE: take, contact, schedule, complete |
| `CS_PERFORMANCE` | `VIEW`, `UPDATE` | VIEW: performance page, records, export, other employees' targets. UPDATE: release/reopen, `not_eligible`, target editor, "Sync new orders" |

- Use exactly these strings in the permission constants (for example `MODULES.CS_ACTION`, `MODULES.CS_PERFORMANCE`).
- ADMIN/SUPER_ADMIN always have access. Users of type `CUSTOMER_SERVICE` can work tasks without a grant. Do not hide the CS Action menu from them just because no role row exists.
- The API also accepts `READ` as a synonym of `VIEW`. The portal should check `VIEW`.
- No role gets these automatically. An admin assigns them on the Roles & Permissions page.

## P6 Chat details save → CS task (after the Chat branch merge)

When `codex/telegram-customer-intake` is merged with this backend, `PATCH /api/admin/chat/conversations/:id/details` responds with the saved details **plus** `csAction`:

```json
{ "id": "…", "conversationPurpose": "SALES", "conversationOutcome": "INTERESTED_PENDING", "linkedUserId": 3526, "csVersion": 5,
  "csAction": { "ingested": true, "taskId": "42", "duplicate": false } }
```

| `csAction` | Show in the Chat panel |
|---|---|
| `{ingested:true, duplicate:false, taskId}` | Toast "Sales follow-up task created" with link to `/cs/actions/{taskId}` |
| `{ingested:true, duplicate:true, taskId}` | "Added to existing follow-up task" + link |
| `{ingested:false, skipped:true, reason:"Not a sales chat that needs follow-up"}` | Show nothing |
| `{ingested:false, skipped:true, reason:"Chat lead is not linked to a bEasy customer yet"}` | Small hint: "Link this chat to a customer to create a follow-up task" |
| any other skip (`declined`, `Verified purchase already recorded…`, permission) | Small info text with `reason`. The details are still saved. |

- A task is created only when purpose is **Sales** and (Follow-up = Yes, or outcome Interested/Pending or Booked).
- `csAction` never means the save failed. Always treat the details save as successful when the HTTP status is 2xx.
- `csAction` may be missing on servers without the CS code. Treat missing as "nothing to show".

## P7 Switching off sample data

Backend will tell you when api-dev is deployed. Then check, with a real admin login:

1. `GET /api/admin/cs/actions?page=1&limit=1` → 200 with `items`, `totalItems`, `counters`.
2. `GET /api/admin/cs/performance/definitions` → 200 with `resolutions`.
3. Role editor shows `CS_ACTION` and `CS_PERFORMANCE`.

Then switch off sample data for CS Action / Performance.

## P8 Scheduler

Automatic task generation is not on yet (waiting for approved operator, lookback and cadence). Keep the manager **"Sync new orders"** button (`POST /api/admin/cs/actions/reconcile`, `CS_PERFORMANCE / UPDATE`). When backend confirms the scheduler is live, it can be moved to a secondary/"repair" place, but not removed.

## Still waiting (no portal work yet)

- **E1 policy values:** SLA, retry timing, escalation, sales-assistance window, review timing, revenue/VAT. Keep showing `available:false` / "Not configured" and never show `null` as 0.
- **A2 production segments:** staging still has only the 2 protected test segments.

## Portal test checklist

- [ ] Sales task in progress (taken by me): Complete shows Lost, Sales decline; Not eligible only for a manager.
- [ ] Complete → Lost with note → task shows Completed / "Lost", `completeOptions` empty, list refreshed.
- [ ] Sales decline asks for confirmation; after it, creating a follow-up for the same customer shows skip reason `declined`.
- [ ] Task not taken by me: Complete → 409 message, Take button shown.
- [ ] Stale version (another tab changed it) → 409 → refetch, note kept.
- [ ] Review task: Complete shows only Not eligible (manager); received/declined via the contact form.
- [ ] Support task: Issue resolved fails with the ticket message until the ticket is RESOLVED.
- [ ] Completed "converted" task shows label "Converted" (from definitions or the fallback map).
- [ ] CS user without `CS_PERFORMANCE` does not see manager options, targets editor or Sync button.
- [ ] (after Chat merge) Saving a Sales chat with Follow-up = Yes shows "task created" with a working link; an unlinked chat shows the link hint.
