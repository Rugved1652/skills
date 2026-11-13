---
name: api-standard-practices
description: Design, implement, or review backend REST/HTTP endpoints with consistent CRUD, search, response contracts, pagination, middleware, and authorization. Use for server-side API changes, not frontend API consumption alone.
---

# API Standard Practices

Implement the smallest complete change. Infer engineering essentials; do not invent business rules or expand scope.

## Inspect and reuse

- Inspect comparable endpoints, consumers, schemas, auth/error handling, database access, and tests.
- Explicit requirements and established contracts outrank these defaults. Treat routes, methods, statuses, headers, fields/types/nullability, defaults, filtering/sorting, pagination, and stable error codes as contract. Flag correctness/security defects; neither copy them nor silently break compatibility.
- Before a required breaking change, identify affected consumers, schemas, docs, tests, generated clients, and migration/versioning needs. Keep existing API specifications synchronized.
- Reuse shared policies/helpers across endpoints; extract common behavior while keeping queries and permissions explicit. Avoid speculative CRUD frameworks or infrastructure. Ask only about material business, permission, or compatibility ambiguity.

## Middleware and request lifecycle

Reuse native middleware/hooks, dependencies, and exception handlers at app/route-group scope. Verify order and early exits; avoid per-handler duplication.

| Concern | Placement and guardrail |
|---|---|
| Authentication | Verify credentials once; attach trusted identity. Protect route groups with explicit public exceptions. Stop on rejection; authorize resources/actions at the operation/data boundary. |
| Errors/recovery | Centralize validation, auth, and unexpected failures into the shared contract; preserve status/headers and log safely once. Never rewrite a committed response. |
| Observability | Establish request IDs early; record redacted access logs and timing, including rejected requests. |
| Resource limits | Enforce body limits before parsing and appropriate throttling before expensive work; propagate deadlines/cancellation. Trust forwarded client IPs only through configured proxies. |
| Browser protections | Configure CORS when needed, covering preflight and error responses. CORS is not authorization; protect cookie-authenticated mutations against CSRF. |

Keep parsing/validation in supported lifecycle stages. Preserve raw bytes for webhook verification; never blindly buffer streams or wrap every response in JSON.

## Methods and outcomes

Use these defaults for new contracts; select routes consistent with the project.

| Operation | Method / target | Outcome |
|---|---|---|
| List or ordinary search | GET collection | 200, including empty results |
| Read one | GET resource | 200; absent resource: 404 |
| Create | POST collection | 201; include Location for an addressable created resource |
| Replace | PUT resource | 200 with representation or 204; 201 only for supported creation |
| Partial update | PATCH resource | 200 with representation or 204 |
| Delete | DELETE resource | 204, or 200 with a result representation |
| Async operation | Operation-appropriate method | 202: accepted, not completed; return an operation/status resource or monitor URL |

GET/HEAD must not request business mutations. PUT replaces the client-writable representation. Idempotency concerns effects, not identical responses; POST/PATCH are not automatically retry-safe. Avoid DELETE bodies unless an established contract supports them; use the project's batch/operation endpoint for structured deletion. Use 202 only when the requested result is deferred, not merely because an internal queue exists. See [HTTP semantics](https://httpwg.org/specs/rfc9110.html).

[PATCH](https://www.rfc-editor.org/rfc/rfc5789.html) must define its document format/media type and validate accordingly. Partial JSON, JSON Merge Patch, and JSON Patch have different null/omission, array, and operation semantics. Apply all changes atomically or none; reject unsupported media types with 415. Advertise accepted formats through Accept-Patch where supported.

| Failure | Status / guardrail |
|---|---|
| Malformed syntax/framing, unparsable content, or malformed path/query shape | 400 |
| Well-formed, supported content failing schema/semantic validation | 422 Unprocessable Content, or established 400 convention |
| Missing/invalid authentication | 401; preserve WWW-Authenticate challenge |
| Forbidden action | 403; use 404 where disclosure policy conceals existence |
| Missing resource | 404; do not disguise database outages as absence |
| Unsupported method | 405 with Allow |
| Conflict with current resource/domain state | 409; prefer over 422 for uniqueness/version conflicts |
| Failed write precondition, such as If-Match | 412; do not collapse into 409 or 422 |
| Excessive payload / unsupported media type | 413 / 415 respectively |
| Rate limit / temporary service unavailability | 429 / 503; Retry-After when known |
| Unexpected server failure | 500; safe message, internal diagnostics |

## Shared JSON response

This envelope is an application convention, not an HTTP requirement. For new contracts without an established format, define shared types/models and schemas once:

```text
ApiResponse<T> = {
  status: number,
  message: string,
  data: T | null,
  errors?: [{ code: string, message: string, field?: string }]
}
```

- Reuse response/error helpers; derive transport and body `status` from one value, never HTTP 200 with body status 500.
- Success: public result in `data`, no `errors`. Read/create/update return the resource directly; lists use `data: { items, pagination }`. Empty lists contain `items: []`, not null.
- Failure: `data: null`; optional `errors` carries stable codes and safe details. Keep messages brief; clients must not parse prose. Strip raw inputs/secrets/internals. Document one `field` path syntax for nested objects/arrays, identifying the request source.
- Preserve established formats, including resource-native responses and RFC 9457 errors; never mix/migrate shapes incidentally. Exempt downloads, streams, mandated payloads, and bodyless responses (204, 205, 304, HEAD).

## CRUD and write safety

- Validate path/query/body at runtime. Separate create/update inputs from public outputs; allowlist writable fields, protect immutable fields, and handle unknown fields consistently. Return persisted public data, not echoed input or raw rows.
- Apply trusted tenant/ownership scope to reads, counts, mutation predicates, and referenced IDs; authentication alone cannot authorize resources/actions.
- Distinguish absent resources from empty collections. Honor patch-format null/omission semantics; never invent upserts, cascades, or retention/soft-delete changes.
- Enforce invariants through constraints/atomic operations; use transactions for coupled writes. Read-then-write checks can race. Reuse established ETag/If-Match support against stale writes; failed write preconditions return 412. Require new write preconditions only for demonstrated lost-update risk and a compatible contract.
- Reuse idempotency protection when retries could duplicate consequential effects. When adding it, scope keys to caller/operation, bind request content, reject conflicting reuse, coordinate concurrent claims with effects, persist outcomes, reauthorize replays, and cover the retry window with retention. Database transactions do not cover external services.

## Search and pagination

- Use GET parameters for ordinary search; documented POST for justified complex input, not GET bodies. Establish searched fields, matching, and empty-query behavior from requirements/conventions.
- Bound query length/work; allowlist fields/operators/sorts, parameterize values, and escape wildcards for literal matching. Avoid arbitrary user regex or unsolicited search infrastructure. Authorize/filter before database pagination/counting; never fetch everything then slice.
- Paginate growing collections; small, bounded lookup lists may remain unpaginated. Reuse parameters, parsing, finite defaults/maximums, validation, and metadata across endpoints.
- Without a convention, choose page/offset for page navigation or cursors for scale/sequential traversal. Use deterministic ordering with a unique tie-breaker; stable ordering is not snapshot consistency.
- Treat cursors as opaque to clients; validate structure, any version/integrity mechanism, and sort/filter scope server-side. Reject malformed values rather than restart. Derive access scope from authenticated context, never cursor-contained identity.
- Derive continuation with an extra-row probe or equivalent; a full page proves nothing about the next. Include totals only when required, using identical authorization/filter scope.

## Framework examples

For users and orders endpoints, share policy while retaining separate queries and permissions:

- **Fastify:** share pagination/response schemas. Scope auth [hooks](https://fastify.dev/docs/latest/Reference/Hooks/): `onRequest` for credential-only checks, `preHandler` when validated input is needed; bodies are unavailable in `onRequest`. Use `setErrorHandler` for error responses, `setNotFoundHandler` for routing 404s; use `onError` only for observation.
- **FastAPI (Python):** share request/response models and pagination dependencies; use `Depends`/`Security` for route auth, middleware for request context/logging, and [exception handlers](https://fastapi.tiangolo.com/tutorial/handling-errors/) for request-validation/HTTP errors. Preserve status/headers; middleware rejections must return a response, not assume route exception handling applies.
- **Go:** compose `http.Handler` middleware for auth/context, centralize error responses, and recover panics at the request boundary; send errors only before commitment. Share pagination parsing and response structs; preserve cancellation, required `data: null`, and empty `items: []` rather than nil slices.

## Load advanced guidance only when needed

- Batch mutations: [bulk](references/bulk.md).
- File ingestion/direct uploads: [uploads](references/uploads.md).
- Deferred processing/status endpoints: [async jobs](references/async-jobs.md).
- Receiving or delivering events over HTTP: [webhooks](references/webhooks.md).
- HTTP or application cache behavior: [caching](references/caching.md).

Do not add these capabilities to ordinary CRUD without a requirement.

## Verify and finish

Test affected behavior: HTTP/envelope status agreement, safe errors, bodyless exceptions, CRUD outcomes, omitted/null updates, search/access scope, pagination boundaries, and consistent defaults. Check protected/public routes, early middleware rejection, relevant CORS/preflight, and no second response or error rewrite after response commitment. Exercise relevant conflicts, rollback, and retries; avoid tests that merely mirror implementation.

Run targeted tests first, then available lint/typecheck/build for the affected package/service. Expand for shared-code changes, repository instructions, or CI requirements. Fix introduced failures; distinguish pre-existing/environment failures. Briefly report commands actually run, results, and material limitations; never claim unexecuted checks passed.
