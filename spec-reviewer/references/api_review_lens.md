# API Review Lens

**Load condition:** Spec contains HTTP endpoints, API contracts, RPC, REST, or GraphQL schemas.

**Purpose:** API specs have failure modes that general specs don't—idempotency bugs, pagination traps, versioning breakage, error inconsistency. This lens flags API-specific anti-patterns using patterns from Google AIP, Microsoft API Guidelines, and API documentation best practices.

---

## Resource Design

**Check items:**

- [ ] Resources are **nouns**, not verbs (e.g., `/users`, not `/getUsers`)
- [ ] Singular vs. plural consistent throughout (e.g., `/users/{id}/orders`, not `/user/{id}/order`)
- [ ] Case style uniform: one of snake_case, camelCase, kebab-case, documented
- [ ] Hierarchy depth ≤3 (flag `/a/{id}/b/{id}/c/{id}/d/{id}` as complexity smell)
- [ ] Standard CRUD resources use `/resource` (list), `/resource/{id}` (get), `POST /resource` (create), `PUT/PATCH /resource/{id}` (update), `DELETE /resource/{id}` (delete)

---

## Methods & Operations

**Check items:**

- [ ] GET is idempotent, read-only, no side effects (never triggers writes, emails, webhooks)
- [ ] POST creates (idempotent **if** idempotency key supported); idempotent key documented
- [ ] PUT replaces entire resource, is idempotent (safe to retry)
- [ ] PATCH updates partial; document merge semantics (RFC 6902, RFC 7386, or custom—if custom, specify)
- [ ] DELETE removes; is idempotent (DELETE twice = same result as DELETE once)
- [ ] Long-running ops explicitly use: async polling (status endpoint + 202 Accepted) OR webhooks; never blocking sync
- [ ] Bulk operations state atomicity: all-or-nothing, partial success + error list, or per-item success/fail?

---

## Contracts (Request & Response Schemas)

**Check items:**

- [ ] Every endpoint documents: HTTP method, path, query/header/body parameters, request body schema
- [ ] Every parameter labeled: required vs. optional; type; constraints (min/max length, regex, allowed values)
- [ ] Every endpoint documents: success response (HTTP code, body schema, example)
- [ ] Every endpoint documents: error responses (all codes, error response structure, example)
- [ ] Error responses **never** return 200 OK with `"error": true` (violates HTTP semantics; use 4xx/5xx)
- [ ] Error structure consistent: always `{code, message, details}` or `{type, status, detail}`; never `{err, error, errorMsg}` mixed
- [ ] Error codes map to HTTP semantics:
  - 400 Bad Request (client input invalid)
  - 401 Unauthorized (auth missing/invalid)
  - 403 Forbidden (auth valid, action not allowed)
  - 404 Not Found (resource missing)
  - 409 Conflict (idempotent retry rejected, state mismatch)
  - 429 Too Many Requests (rate limit hit)
  - 500 Internal Server Error (server-side fault)

---

## Pagination & Filtering

**Check items:**

- [ ] All list endpoints have pagination (flag unbounded lists returning 1000+ items)
- [ ] Pagination documented: cursor-based, offset-based, or keyset? Default page size? Max allowed?
- [ ] Page size defaults to reasonable limit (e.g., 20, 50); client can request smaller, not larger than cap
- [ ] Cursor-based pagination preferred for large datasets (stable across page changes)
- [ ] Filtering parameters documented: exact match, range, prefix? Query syntax (e.g., `filter=status:active`)?
- [ ] Sorting documented: fields supported, ascending/descending, default order

---

## Versioning

**Check items:**

- [ ] Versioning strategy explicit: URL path (`/v1/`), Accept header (`application/vnd.api+json;version=1`), or query param (`?version=1`)
- [ ] Breaking change policy defined: what counts as breaking? How long does old version stay live?
- [ ] Deprecation timeline: deprecated endpoint flags + sunset date documented (HTTP `Sunset` header or changelog)
- [ ] New fields in responses: backward-compatible (clients ignore unknown fields); no removal without major version
- [ ] Field renames or type changes: major version bump only

---

## Auth & Rate Limiting

**Check items:**

- [ ] Auth mechanism per endpoint (or global + exceptions): OAuth2, API Key, JWT, mTLS, Basic Auth?
- [ ] Auth documented: where to pass (header, query, body), format, examples
- [ ] Scopes or permissions documented: what each scope/role grants, what endpoints require what scope
- [ ] Rate limits stated: per-endpoint or global? How measured (requests/min, tokens/day)? Limit values? Retry-After header on 429?
- [ ] Rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` or equivalent, documented

---

## Anti-Patterns to Flag

**Pattern 1: RPC-style endpoints disguised as REST**

❌ *Example:* `POST /getUser` (action verb in endpoint)
✓ *Correct:* `GET /users/{id}` (resource noun + HTTP method for action)

**Pattern 2: Mixed naming conventions**

❌ *Example:* `GET /users/{id}/ordersHistory` (camelCase) mixed with `GET /products/{id}/stock_level` (snake_case)
✓ *Correct:* Uniform throughout (`/users/{id}/orders_history` OR `/users/{id}/ordersHistory` — pick one)

**Pattern 3: Undocumented side effects**

❌ *Example:* `POST /reports` (spec says "creates report") but silently sends email to owner; no mention in spec
✓ *Correct:* Document: "Creating a report triggers email notification to resource owner"

**Pattern 4: Pagination absent on high-cardinality lists**

❌ *Example:* `GET /logs` with no pagination; can return 50,000 records
✓ *Correct:* Add `?limit=50&offset=0` or cursor pagination; document default/max page size

**Pattern 5: 200 OK with error payload**

❌ *Example:* `HTTP 200 {"error": true, "message": "Invalid email"}`
✓ *Correct:* `HTTP 400 {"code": "INVALID_EMAIL", "message": "..."}`

**Pattern 6: Idempotency not addressed for mutations**

❌ *Example:* `POST /payments` can charge twice if client retries (no idempotency key)
✓ *Correct:* Support `Idempotency-Key` header; guarantee deduplication by key over time window

---

## Sources

- [Google AIP (API Improvement Proposals)](https://aip.dev) — Concise, opinionated, widely adopted
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines) — Enterprise-grade recommendations
- [Tom Johnson: API Quality Checklist](https://idratherbewriting.com/learnapidoc/docapis_quality_checklist.html) — Pragmatic docs focus
