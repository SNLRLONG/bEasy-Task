# Security checklist (OWASP Top 10)

Based on OWASP Top 10:2025 (check https://owasp.org/Top10/ for the current edition and update this list when it changes). Run it for T3 tasks and big T2 features only (see `AGENTS.md`). Write `N/A: <reason>` for items that do not apply. Frontend is never the security boundary; for each item also check what backend must enforce and file a request if missing.

## Always (every change)
- [ ] No secrets, tokens, keys or `.env` values in code, docs, logs or commits.
- [ ] No `dangerouslySetInnerHTML` / `innerHTML` / unsanitized WebView HTML with user data.
- [ ] No personal data or tokens in `console`, analytics events, URLs or error messages.

## Feature checklist
| # | Risk | Check |
|---|---|---|
| A01 | Broken access control | Every screen/route/action is guarded by role/permission; hidden UI is not protection, backend also enforces it; no IDs the user can change to see others' data (IDOR). |
| A02 | Security misconfiguration | Env/flavor config correct (dev vs prod URLs); no debug flags, verbose errors or open CORS/hosting rules shipped; vConsole off in prod. |
| A03 | Software supply chain | New dependency is needed, maintained, pinned in the lockfile, and checked (`yarn audit` / `pub outdated`); no unknown scripts or CDN code without integrity. |
| A04 | Cryptographic failures | Sensitive data only over HTTPS; tokens in secure storage/cookie flags, not localStorage where avoidable; no custom crypto; no sensitive data cached or exported unnecessarily. |
| A05 | Injection | User input is validated/encoded before use in URLs, queries, HTML, deep links, file names, CSV/Excel export (formula injection `= + - @`). |
| A06 | Insecure design | Abuse cases considered: replay, double-submit, skipping steps, price/amount tampering, enumeration; money and status rules enforced by backend. |
| A07 | Authentication failures | Login/OTP/session handling follows the contract; expired/invalid token logs out cleanly; no auth state trusted from the client alone; no credentials in URLs. |
| A08 | Software/data integrity | Data from API/deep link/WebView bridge is validated before use; deploy and OTA (Shorebird/Firebase) done only through the release plan. |
| A09 | Logging and alerting failures | Security-relevant actions (login, permission change, export, refund) are logged by backend; client reports errors without leaking data. |
| A10 | Mishandling exceptional conditions | Failures (network, 401/403/5xx, empty, malformed data) fail closed with a safe message; no crash exposing data; retries are bounded. |
