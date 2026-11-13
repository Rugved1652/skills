---
name: fastapi-folder-structure
description: Organize, extend, or refactor FastAPI Python services with explicit feature boundaries, shared schemas/types/utilities, dependency wiring, and resource lifecycles. Use for backend layout and code placement, not HTTP behavior alone.
---

# FastAPI Folder Structure

Make ownership and dependencies clear. Prefer feature-first organization for new services; adapt existing architecture without imposing a directory migration.

## Inspect before placing code

- Inspect Python/FastAPI/Pydantic versions, packaging, ASGI entrypoint, router assembly, settings, middleware, dependencies, database tooling, migrations, and tests. Trace one comparable feature end to end.
- Preserve routes, contracts, import paths, startup commands, migration discovery, and test fixtures during moves. Update actual callers; avoid unrelated tooling changes or mass renames.
- Preserve the chosen persistence approach, settings library, synchronous/asynchronous clients, and test conventions; do not force an ORM into SP-only services. Resolve material business or compatibility ambiguity before changing behavior.

## Placement map

This is a responsibility map, not a scaffolding checklist. Create only needed files; omit empty layers where the selected architecture permits. SP mode retains its specified entry flow below. Keep the project's package convention (`__init__.py` where needed; omitted below).

```text
app/
  main.py                        # ASGI application export
  application.py                 # create_app and assembly
  core/
    config.py
    lifespan.py
    security.py
    exceptions.py
    exception_handlers.py
  middleware/                    # actual cross-request middleware
  infrastructure/
    database.py                  # engine/client and session providers
    clients/                     # external-service clients when needed
  schemas/                       # shared runtime API contracts
  contracts/                     # shared internal types/protocols
  utils/                         # shared pure helpers
  modules/
    users/
      router.py
      controller.py              # HTTP adaptation; separate in SP mode
      dependencies.py
      service.py
      repository.py
      schemas.py                 # request/response models
      models.py                  # persistence models when applicable
      types.py                   # internal feature types
      utils.py
      exceptions.py
tests/
  modules/users/
  integration/
  conftest.py
migrations/                      # preserve the migration tool's convention
database/
  procedures/                    # SQL only when owned in this repository
    users/
      list_users.sql
```

| Location | Responsibility and boundary |
|---|---|
| `application.py` | Compose settings, lifespan, middleware, exception handlers, and routers; no business operations. |
| `router.py` / `controller.py` | Declare HTTP contracts/dependencies; adapt validated inputs and public outputs. Outside SP mode, simple adaptation may stay in the route. |
| `dependencies.py` | Resolve identity/resources and construct feature collaborators; no duplicate business workflows. |
| `service.py` | Application orchestration/authorization and transaction coordination; respect procedure-owned rules. No `Request`, `Response`, or `Depends` resolution. |
| `repository.py` / `models.py` | Ordinary queries outside SP mode; exactly one public API-entry SP call per operation in SP mode. Models only when needed; accept supplied resources, never choose HTTP responses. |
| `schemas/` / feature `schemas.py` | Shared transport contracts versus feature inputs/outputs; protect writable and exposed fields. |
| `contracts/` / feature `types.py` | Shared internal types/protocols versus local domain types; no duplicate model for identical semantics. |
| `utils/` / feature `utils.py` | Pure domain-neutral helpers versus feature-only helpers; no database, HTTP, or business orchestration. |

Keep shared pagination models, normalization, response envelopes, and error mapping in their appropriate boundary once. Promote feature-local code only when consumers share its meaning. Do not use `core/`, `contracts/`, or `utils/` as miscellaneous storage.

## Stored-procedure API mode

When the project requires stored procedures, every data-backed API operation
maps to exactly one approved public API-entry stored procedure through its
repository.

A repository operation must execute exactly one public entry procedure and
must not substitute direct table queries, ORM queries, ad-hoc SQL, or multiple
public procedure calls.

The entry procedure may internally reuse private database procedures,
functions, views, or helpers. These private routines are database
implementation details and must not be called directly by HTTP-facing
repositories.

Maintain the flow:

```text
route → controller → optional service → repository → one API-entry SP
```

Do not reuse one public API-entry procedure across distinct API operations.
Reuse common database logic through private functions or procedures.

Retain API validation/auth and response mapping; avoid duplicating procedure-owned rules or requiring ORM models. Keep typed SP inputs/results in feature `types.py` and SQL in the established deployment system. Operations without database access need no SP. Read [stored-procedure guidance](references/stored-procedures.md) for explicit input/output contracts, transactions, deployment, and verification.

## Dependency direction and routing

Assembly wires transport → service → persistence/infrastructure. Pass resolved identity, validated values, and collaborators into ordinary functions or classes. Directly calling a function with `Depends` defaults does not resolve dependencies.

Expose an intentional service function or protocol for cross-feature collaboration, injected through dependency wiring. Never import another feature's private repository or the ASGI app into a feature. Resolve cycles through ownership and explicit interfaces; avoid broad re-export barrels or `sys.path` hacks.

In SP mode, cross-feature application collaboration must not add persistence calls; the operation's entry SP composes private database helpers for that work.

Compose [APIRouter](https://fastapi.tiangolo.com/tutorial/bigger-applications/) instances with deliberate prefixes, tags, response models, and dependencies. Include once per intended route/version; check static routes against dynamic-path collisions. Group protected routes separately from public routes: adding a route dependency cannot remove an inherited authentication requirement. Router dependency lists enforce checks; use a parameter dependency when the handler needs the returned identity.

## Application and resource lifetimes

- `create_app` assembles independently testable applications; `main.py` exposes the configured ASGI app. Keep server launch/process signals in the existing runner. Load and validate settings centrally; support test injection and never log secrets.
- Acquire shared engines, pools, and clients through [lifespan](https://fastapi.tiangolo.com/advanced/events/), not import-time connections. Expose them through app state and typed dependency providers, not mutable module globals. Track ownership: close acquired resources, leave borrowed resources to their owner.
- Pair acquisition with cleanup immediately, including partial startup failure; release resources in reverse dependency order. Initialize per worker intentionally. Avoid migration execution in every worker. Do not mix a supplied lifespan with legacy startup/shutdown handlers and assume both execute.
- Request dependencies acquire/close request resources using supported context management. Never share mutable sessions across requests or concurrent tasks. Background work owns its own resources; it must not retain request sessions.
- Honor one transaction owner: application or procedure. Share the connection for application-owned transactions; in SP mode, wrap only the single entry call and never blindly wrap self-committing SPs. Confirm completion before success, not post-response [yield cleanup](https://fastapi.tiangolo.com/tutorial/dependencies/dependencies-with-yield/). Verify cleanup timing for streams/resources; do not swallow exceptions.
- Separate persistence models from public schemas without requiring redundant copies everywhere. Preserve the migration tool's model/metadata imports when relocating models; ensure discovery does not depend on importing routes or connecting to production.

## Request policies and execution

Use `Depends`/`Security` for authentication and declared route requirements; authorize actions/resources in operations and scoped queries/procedure contracts. Keep credentials and trusted identity out of generic utility modules.

Use middleware for request context, redacted access logs, timing, and actual cross-request policies. Verify wrapping order, early rejections, public routes, and applicable CORS/preflight. Register centralized validation/HTTP/application error handlers, preserving status, required headers, and safe response contracts. Middleware rejection should return a response rather than assume route exception handling applies. Do not buffer streams or rewrite committed responses.

Match [execution mode](https://fastapi.tiangolo.com/async/) to clients: await asynchronous I/O; use synchronous route/dependency execution or deliberate offloading for blocking work. FastAPI offloads synchronous handlers/dependencies, not ordinary helper calls made inside `async def`. Keep session/thread ownership compatible with the chosen driver; do not share an async session across concurrent operations.

## Add orders after users

1. Trace users registration, dependencies, auth, pagination, response schemas, tests, and persistence. Reuse sound policies; preserve existing contracts.
2. Add orders-specific routes/schemas and needed layers. In SP mode, define each operation's distinct entry SP and input/output contract before wiring its single-call repository; share database logic through private helpers, not public users procedure calls.
3. Reuse shared pagination and response contracts without cloning them. Centralize common error mapping; keep feature exceptions local.
4. Include the orders router once with its intended prefix/access scope. Update affected migration discovery, imports, and API documentation; verify users remains unchanged.

Use `api-standard-practices` when available for detailed HTTP behavior; this skill works independently.

## Verify and finish

Test imports and the real ASGI target, route/OpenAPI registration, protected/public access, shared pagination, and safe error responses. Exercise operation authorization, transaction rollback, startup failure cleanup, shutdown, and independent app instances where affected.

Use an isolated test app/database. Restore dependency overrides and settings caches; keep fixtures in `conftest.py`. Exercise lifespan using a context-managed `TestClient`; an async HTTP client alone does not trigger it, so use the project's explicit [lifespan test setup](https://fastapi.tiangolo.com/advanced/async-tests/). Test service behavior separately from persistence integration; avoid tests asserting folder names.

Run targeted tests and available lint/typechecks/build checks for the affected package; expand for shared changes or CI requirements. Report the placement decisions, actual verification, and remaining limitations briefly.
