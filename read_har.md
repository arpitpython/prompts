You are helping an automation engineer fully understand a client portal from a HAR file — covering tech stack, authentication, security, API structure, and dynamic data flows — so we can reliably automate report downloads with no hardcoding.

HAR file path: $ARGUMENTS

---

## Core Principles

- **Do NOT guess or assume anything** — base everything strictly on HAR data
- **Do NOT conclude early** — a finding (e.g. auth token in body) does NOT rule out other mechanisms; always verify exhaustively
- Accuracy over speed — but skip static assets (images, fonts, CSS, JS bundles)
- Focus only on API / XHR / fetch / auth-related requests

---

## IMPORTANT — Large File Strategy

HAR files are often 30–60 MB. Never read the whole file. Follow this strictly:

1. Use **Grep** for specific patterns — fast and precise
2. Use **Read with offset + limit** only for small targeted windows around a grep match (20–50 lines)
3. Work through each step below sequentially using Grep
4. Never skip a step because a prior step gave a "good enough" answer

---

## Step 1 — Identify Domain and Tech Stack

Grep for `"url"` — limit to first 30 results. Note:
- The portal domain(s)
- Any hints in response headers: `Server`, `X-Powered-By`, `x-aspnet-version`, `x-aspnetmvc-version`
- API patterns: REST, GraphQL, RPC-style endpoints

Grep for `"Server"` and `"x-powered-by"` in the HAR to identify backend tech.

---

## Step 2 — Authentication — Login Request

Grep for each one at a time (stop when found):
- `login`
- `signin`
- `/auth`
- `/token`
- `authenticate`

For the matching entry, read surrounding lines to extract:
- Full URL and HTTP method
- Complete request payload field names
- All request headers sent with the login call (list every single header — do not skip)
- Response body: token field names, cookie names, any encrypted IDs returned

---

## Step 3 — Token Tracing (CRITICAL — do NOT skip or shortcut)

This step is mandatory regardless of what Step 2 found.

**3a — Extract the token value** from the login response (first 30 characters of the JWT or session token is enough).

**3b — Grep for that token value substring** across the entire HAR. This reveals EVERY place the token appears — request headers, request bodies, query params, cookies. Document all locations found.

**3c — For each post-login API request found**, read its raw header array in full. List every header name present. Do NOT rely only on searching for header names by pattern — read the actual headers array. Specifically confirm presence or absence of:
- `authorization` / `Authorization`
- `cookie` / `Cookie`
- `x-auth-token` or any custom auth header
- Any header whose value starts with `ey` (JWT prefix) or is a long opaque string

**3d — Document the auth mechanism exhaustively:**
- Is the token sent in a header? Which header? What format (raw JWT, `Bearer <JWT>`, custom)?
- Is the token sent in the request body? Which field?
- Is it sent in BOTH places? Note both.
- Is there a cookie carrying a session ID alongside the token?

**3e — Check token lifecycle:**
- Is there a refresh endpoint? (grep for `refresh`, `renew`, `extend`)
- What is the token expiry field name and format?
- Is there a logout/signout call?

---

## Step 4 — Portal Context and Dynamic IDs

Grep for each one at a time:
- `companyID`, `company_id`, `companyId`
- `locationID`, `location_id`, `locationId`
- `serverID`, `server_id`
- `userID`, `user_id`, `userId`
- `userRoleID`, `roleId`
- `org_id`, `organization`, `account_id`, `store_id`

For each match: note the URL that returns it, the JSON field name, and whether it's an encrypted/encoded value or a plain integer.

**Dynamic data priority:** Identify which endpoints return the location list, report list, or filter options — these must be called dynamically at runtime, not hardcoded.

---

## Step 5 — Full Login Sequence Mapping

After Steps 2–4, document the complete ordered sequence of calls made between login and the first real API call. Include:
- Every intermediate call (role activation, user validation, session setup, etc.)
- For each call: URL, method, what it sends, what it returns
- Whether each call is strictly required or optional

---

## Step 6 — Report / Data Download Flow

Grep for each one at a time:
- `export`, `generate`, `/report`, `/download`, `/queue`, `/run`, `job_id`, `task_id`
- Also grep for the specific report name if known

For the matching entry extract:
- Full URL pattern and method
- All payload fields — especially date range, location/filter fields, format/type
- Response: direct file (synchronous) or job_id (async)?

---

## Step 7 — Parameter Valid Values (do not skip)

Many portals reject calls if parameter values don't match an allowed set.

Grep for: `validValue`, `allowedValue`, `"options"`, `"enum"`, `validate`, `resolve`

For every match: which parameter, what are the allowed values, what is the default.

---

## Step 8 — Async Polling (if applicable)

If Step 6 found a job_id, grep for `status`, `poll`, `progress`, and the job_id value itself.

Extract: polling URL, how job_id is passed, completion field and value, polling interval.

---

## Step 9 — Security Mechanisms

Grep for and document:
- `csrf`, `xsrf`, `_token` — CSRF protection
- `nonce`, `_=`, `timestamp` in URLs — anti-replay / cache-busting
- `302` or `Location` in responses — redirects
- `500` in response status — read its body to identify rejected parameter values
- `logout`, `signout` — whether a session teardown call is expected
- Rate limiting patterns (repeated 429 responses)
- Any unusual or non-standard headers

---

## Step 10 — Save to har_analysis.md

Save all findings to `har_analysis.md` in the current working directory with this structure:

```
# HAR Analysis — [portal domain]

## Tech Stack
- Backend: (detected from Server / X-Powered-By headers)
- API pattern: (REST / GraphQL / RPC)
- Hosting hints:

## Authentication Flow (step-by-step)
| Step | Method | URL | Sends | Returns |
|---|---|---|---|---|

## Token Usage — WHERE and HOW the token is sent
- Header name: (exact name and format, e.g. `authorization: <raw JWT>` or `Authorization: Bearer <JWT>`)
- Body field: (e.g. `authKey` inside `CommonCompanySettings`)
- Cookie: (name if any)
- NOTE: if token appears in multiple places, document ALL of them

## Required Headers (every post-login request)
- header-name: value or format

## CommonCompanySettings / Equivalent Auth Body Block
(Exact field names and where each value comes from)

## Portal IDs Needed
- Field name | Source endpoint | JSON key | Format (plain int / encrypted string)

## Dynamic Data Endpoints
- Locations: URL + response field
- Reports/filters: URL + response field
- Other runtime params: URL + response field

## Report Download Flow
- Exact ordered steps from login to downloaded data

## Parameter Valid Values
- field_name: allowed values

## Async Polling (if applicable)
- Poll URL:
- Completion field and value:

## Security Mechanisms
- CSRF: (present / absent / how handled)
- Anti-replay: (nonce / timestamp in URLs?)
- Redirects: (any 302s in the flow?)
- Logout required: (yes/no)
- Rate limiting: (observed?)

## Edge Cases / Important Notes
- Anything unusual, non-obvious, or that a developer would likely get wrong
```

---

## Step 11 — Plain English Summary

Give the user:
1. Tech stack overview
2. Auth mechanism — exactly how the token is transmitted (header vs body vs both) — be explicit
3. Full login sequence (all steps, not just login call)
4. Dynamic vs hardcoded data — what must be fetched at runtime
5. Report download flow — synchronous or async
6. Parameters with constrained valid values
7. Security mechanisms to handle
8. Any tricky parts a developer would likely get wrong

Do not write any Python code. Analysis and saving findings is the only goal.
