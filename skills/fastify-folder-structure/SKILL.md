---
name: fastify-folder-structure
description: Structure, extend, or refactor Fastify services with feature-owned layers, scoped plugins/hooks, shared types/utilities, and explicit resource lifecycles. Use for application wiring and code placement; not HTTP contract design alone.
---

# Fastify Folder Structure

Build the smallest complete structure with clear ownership. Default to feature-first TypeScript for new services; preserve existing JavaScript and working repository conventions. Folder names are defaults, not reasons to reorganize unrelated code.

## Inspect before changing structure

- Inspect installed Fastify/plugin versions, entrypoints, module resolution, registration/autoload rules, schema tooling, database ownership, tests, and comparable features. Preserve ESM/CommonJS and build conventions.
- Explicit requirements and established contracts outrank this layout. During moves, preserve routes/prefixes, auth scope, imports, startup commands, migration discovery, and generated clients; identify affected consumers before breaking changes.
- Reuse existing infrastructure. Do not introduce an ORM, DI container, autoloader, or mandatory layers merely to fill the tree. The optional `api-standard-practices` skill covers HTTP behavior; this skill works independently.

## Placement map

Create only files with a real responsibility. Example helpers, seeds, and test names are not scaffold requirements.

```text
src/
  app.ts                         # application factory and composition
  server.ts                      # listen, signals, graceful shutdown
  config/
    index.ts                     # load and validate configuration
    schema.ts
  plugins/
    database.ts                  # expose database access and own cleanup
    authentication.ts            # auth dependencies and scoped setup
    errors.ts                    # install error and not-found handlers
  hooks/
    authenticate.ts              # reusable lifecycle hook
    request-context.ts
  http/
    response.ts                  # shared HTTP response builders/adapters
  errors/
    application-error.ts          # shared failure categories
    http-error-mapper.ts          # transport mapping
  schemas/
    pagination.ts                # shared runtime request schema
    api-response.ts
  types/
    fastify.d.ts                 # declaration augmentation
    request-context.ts
    api-response.ts              # derive from schema when supported
  utils/
    pagination.ts                # pure normalization/calculation
    date.ts                      # only genuinely shared helpers
  database/
    client.ts                    # client construction
    migrations/
    procedures/                  # domain-grouped SQL when owned here
      users/
        list_users.sql
    seeds/                       # only when needed
  modules/
    users/
      index.ts                   # feature plugin, prefix, dependency wiring
      routes.ts
      controller.ts
      service.ts
      repository.ts
      schemas.ts
      types.ts
      utils.ts
      errors.ts
      tests/
        routes.test.ts
        service.test.ts
        repository.test.ts
test/                            # root-level, alongside src
  integration/
  helpers/
    build-app.ts
    fixtures.ts
```

## Responsibility and dependency boundaries

| Location | Owns / must not absorb |
|---|---|
| Feature `index.ts` | Wires dependencies and registers routes; exports deliberate capabilities, not private repositories. |
| `routes.ts` | Methods, paths, runtime schemas, hooks, and controller binding; no business orchestration. |
| `controller.ts` | Validated input/identity extraction and HTTP output; no database queries or business rules. |
| `service.ts` | Application orchestration/authorization and transaction coordination; respect procedure-owned rules. Accept values/dependencies, not request/reply objects. |
| `repository.ts` | Ordinary queries outside SP mode; exactly one public API-entry SP call per operation in SP mode. Use injected resources/trusted scope; no HTTP responses or hidden transaction changes. |
| Feature schemas/types | Runtime transport contracts versus internal domain types; do not maintain competing handwritten DTOs. |
| Global schemas/types | Actually shared contracts; named focused files, not one application-wide type dump. |
| Global/feature utils | Pure helpers at their narrowest useful scope; no HTTP, configuration reads, database access, or business policy. |
| `http/response.ts` | Shared response builders/adapters, including pure envelope builders; HTTP-contract helpers do not belong in generic utils. |
| Database/plugins | Client construction versus registration, visibility, and cleanup; queries/procedure adapters remain feature-owned. |

For substantial features, use route → controller → service → repository. Omit empty layers where the selected architecture permits; SP mode retains its specified entry flow below. Inject dependencies through feature assembly; lower layers never import `app.ts` or `server.ts`. Cross-feature calls use explicit exported operations wired by composition, not private file imports. Avoid circular dependencies; keep domain-specific collaborators in their owning feature rather than promoting them to global utilities.

In SP mode, cross-feature application collaboration must not add persistence calls; the operation's entry SP composes private database helpers for that work.

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

Retain API validation/auth and response mapping; avoid duplicating procedure-owned rules or requiring ORM models. Keep typed SP inputs/results in feature `types.ts` and SQL in the established deployment system. Operations without database access need no SP. Read [stored-procedure guidance](references/stored-procedures.md) for explicit input/output contracts, transactions, deployment, and verification.

## Plugin graph and request lifecycle

- Register providers before consumers in an inherited scope. Children inherit parent decorations; sibling plugins do not automatically share them. Use [fastify-plugin](https://github.com/fastify/fastify-plugin) deliberately for infrastructure that must be exposed; dependency metadata does not replace correct wiring/order.
- Keep route plugins encapsulated. Wrapping them in an encapsulation-breaking plugin can change prefixes and hook scope. Preserve existing autoload behavior; otherwise use explicit registration. Inspect the actual [plugin graph](https://fastify.dev/docs/latest/Reference/Plugins/), not just file order.
- Register reused schema IDs before consumers in their common ancestor. Match types to runtime schemas through the repository's schema/type-provider integration; types alone cannot validate requests.
- Plugins install policy; `hooks/` implements reusable request behavior. Use native hooks by default; create `middleware/` only for existing compatible adapters, never add one merely for naming symmetry.
- Install authentication in protected scopes with explicit public exceptions. Credential-only checks can run in `onRequest`; parsed bodies are unavailable there. Use appropriate later stages for validated-input checks. Stop after rejection; never mix callback and async completion. Enforce resource/tenant scope through operations and their queries/procedures.
- Centralize `setErrorHandler` and `setNotFoundHandler` at the intended scope; preserve status/headers and the established response contract. `onError` observes errors; it does not send responses. Never rewrite a committed response. See [hooks](https://fastify.dev/docs/latest/Reference/Hooks/).
- Declare request decorations before use, then allocate mutable values per request. Never share identity objects across requests. `types/fastify.d.ts` describes actual [decorations](https://fastify.dev/docs/latest/Reference/Decorators/); it neither installs plugins nor makes unavailable resources safe.

## Configuration, resources, and shutdown

- `server.ts` loads validated configuration, calls the application factory, listens, and owns process signals. The factory accepts configuration/dependency overrides and returns an independent instance; importing modules must not listen or connect to external systems.
- Acquire shared clients during plugin initialization, not per request or in `onListen`, which injection tests skip. Resource-owning plugins close owned clients through `onClose`; externally injected clients retain their documented owner. Release partially acquired resources if startup fails.
- Follow the documented transaction owner. Outside SP mode, coupled repositories share the application's transaction context; in SP mode, any application transaction wraps only the single entry call. Do not blindly wrap self-committing SPs. Deploy migrations/procedures/seeds explicitly, not per request or uncontrolled worker startup.
- Await `app.close()` during shutdown and respect deployment deadlines; avoid immediate process exit that skips cleanup. Keep configuration reads and secret redaction centralized.

## Add orders after users

1. Inspect users' registration, auth scope, schemas, dependency wiring, and tests; reuse shared pagination/response contracts, hooks, and database resources.
2. Add orders-specific schemas, types, operations, and HTTP adaptation. In SP mode, define each operation's distinct entry SP and input/output contract before wiring its single-call repository; share database logic through private helpers, not other features' API-entry calls.
3. Register orders once in the correct scope, with its prefix; verify providers and schema references resolve. Update imports/specifications affected by the addition.
4. Test route availability, protected/public behavior, tenant isolation, shared pagination behavior, and feature-specific failures without changing users' contract.

## Verify and finish

Use [injection tests](https://fastify.dev/docs/latest/Guides/Testing/) with independently built applications and always close instances; do not import the listening server. Check startup failure/cleanup, provider visibility, prefixes, schemas, request-state isolation, and owned versus injected resources. Keep unit tests for meaningful business behavior and integration tests for real query/transaction boundaries; do not test empty forwarding layers.

Run targeted tests, then available lint/typecheck/build for the affected service; broaden for shared changes or repository requirements. Report commands/results and unresolved limitations briefly, distinguishing introduced failures from existing/environment failures.
