# Turborepo Package Structure

Use this layout for a new Turborepo containing React/Next.js and Fastify. Preserve existing package names, package manager, framework conventions, and working boundaries. This is an ownership map, not a requirement to create empty packages or a second application.

```text
repo/
  package.json                    # private workspace root, packageManager, task scripts
  turbo.json                      # tasks, dependency ordering, cache outputs
  pnpm-workspace.yaml             # pnpm only; otherwise package-manager workspace config
  apps/
    web/
      package.json
      tsconfig.json
      src/
        app/                      # Next App Router; use framework routes for other React apps
        features/                 # app-specific screens, workflows, API hooks, UI state
        providers/                # app-owned context and resource composition
        ws/                       # app config/transport/subscriptions; services/<capability>, hooks/<capability>
        lib/                      # configured clients, environment and framework adapters
      public/
    api/
      package.json
      tsconfig.json
      src/
        app.ts                    # compose Fastify plugins, resources, feature routes
        server.ts                 # listen and shutdown
        config/                   # environment validation
        plugins/                  # instantiate/register app-owned infrastructure
        hooks/                    # app-specific request behavior
        ws/                       # connection config/lifecycle and transport dispatch
        http/                     # transport adapters, response builders
        types/                    # Fastify augmentation and server-internal types
        database/                 # migrations/procedures and app-owned DB setup
        modules/
          users/
            index.ts
            routes.ts
            controller.ts
            service.ts
            repository.ts
            schemas.ts            # server schema wiring; reuse shared schemas when owned there
            types.ts              # internal service, repository and SP types
      test/
  packages/
    ui/
      package.json
      tsconfig.json
      src/
        components/
          button.tsx
          dialog.tsx
        hooks/                    # hooks owned by these primitives
        utils/                    # private UI helpers
        styles/                   # shared tokens/theme/styles, if owned here
    contracts/
      package.json
      tsconfig.json
      src/
        common/
          generic.types.ts        # APIResponse<T>, pagination
        api-types/
          users.types.ts          # public request, params, query and response contracts
        schemas/                  # optional portable runtime transport schemas
    hooks/                        # independent reusable React hooks, when needed
      package.json
      tsconfig.json
      src/
        use-debounce.ts
    utils/                        # environment-neutral pure utilities, when needed
      package.json
      tsconfig.json
      src/
        date.ts
    server/                       # reusable server-only capabilities, when needed
      package.json
      tsconfig.json
      src/
        plugins/                  # reusable Fastify plugin implementations
        auth/                     # auth mechanisms with explicit dependencies
        database/                 # factories/adapters; no process-owned singleton
    typescript-config/            # shared compiler configuration, not application DTOs
      package.json
      base.json
    eslint-config/                # shared lint configuration, when used
      package.json
```

Use `apps/*` and `packages/*` as workspace locations. Every actual app/package needs its own `package.json`; TypeScript code packages also need package-specific compiler configuration, typically extending the shared config. Do not nest a workspace package inside another workspace package. Keep root files focused on workspace tooling.

## Choose ownership before writing code

**One current consumer is enough.** A reusable capability belongs in its owning package from the first implementation. Do not wait for a second app or a second use to extract a `Button`, `Dialog`, portable helper, shared API contract, or reusable backend mechanism. Reusability comes from a stable responsibility and clean dependencies, not an observed import count.

Before creating anything, search workspace package exports, implementations, installed UI libraries, and generator configuration. Reuse or extend the existing public API. If an app already contains the reusable implementation needed by this task, move that implementation into the appropriate package and update its consumers; do not copy it and leave two authoritative versions. Preserve unrelated code and behavior during the move.

| Responsibility | Owner |
|---|---|
| Reusable UI primitive or presentation component with a stable prop API | `packages/ui`, even when only `apps/web` exists |
| Hook/helper used internally by a UI primitive | Inside `packages/ui`; export only if independently useful |
| Independent portable React hook | `packages/hooks`, if it warrants a separate public capability |
| Pure environment-neutral utility | `packages/utils` or an existing focused utility package |
| Public HTTP/WS request, response, event payload, pagination or envelope | `packages/contracts` |
| Portable runtime transport schema shared with consumers | `packages/contracts/src/schemas`, if compatible with both runtimes |
| Reusable Fastify plugin, server auth mechanism or DB factory | `packages/server` or an existing focused server-only package |
| App routes, concrete forms, page sections, redirects, permissions policy, workflow orchestration | Owning app/feature |
| Configured query/socket client, environment reads, listening server and lifecycle ownership | Owning app composition |
| Fastify decorations, database rows, SP inputs/results, service internals, UI-only view models | Their owning app/module; not public API contracts |

A component such as `DeleteInvoiceDialog` remains app-owned when it coordinates that app's API mutation, routing, and permissions. It imports the shared `Dialog` and `Button`. A reusable domain presentation component can have its own focused domain package when its domain API is stable; do not remove useful domain semantics merely to place it in `ui`.

Do not invent future APIs, option flags, consumers, or generic frameworks to justify extraction. Package the concrete reusable capability needed now. Do not move every file into packages: deployment-specific composition and feature workflows still have an app owner.

## Shared API ownership

Both Fastify and React/Next import the same public contracts from `@repo/contracts`; the backend implements the API and the frontend calls it. Neither app imports the other app's source. Preserve existing generated contracts/client workflows instead of introducing a competing handwritten contract package.

- Move or derive public request/response contracts and their common envelope once. Remove competing app-local definitions and update affected imports. App-local compatibility re-exports may temporarily point to the authoritative package during an incremental migration.
- Infer from the existing runtime schema tooling when appropriate; distinguish raw input, validated output, and serialized response. If schema tooling is server-only, keep it on the server and use the existing generation or explicit conformance checks for the portable contract.
- Shared DTOs describe the wire representation: for example a JSON timestamp is a string, not a database `Date`. Use explicit public fields rather than exposing ORM models or SP results.
- Keep Fastify request/reply objects, decorators, Node-only dependencies, credentials, and database drivers out of browser-importable contracts.
- Type sharing does not perform runtime validation. Fastify route schemas and response serialization must match the public contract. Validate external input at the appropriate trust boundaries.

```ts
// apps/api/src/modules/users/routes.ts AND apps/web feature API service:
import type {
  CreateUserAPIRequest,
  CreateUserAPIResponse,
} from '@repo/contracts/users';

// React/Next app UI:
import { Button } from '@repo/ui/button';
```

These are example public entry points: declare them in each package's `exports` before use. Match naming to the repository rather than assuming the directory path is an exported import path.

## Dependency and runtime boundaries

- Apps depend on packages. Packages never depend on `apps/*`. Packages may depend on other packages through declared, acyclic dependencies. There must be no frontend-to-backend app source imports.
- Declare every direct dependency in the consuming package's `package.json`, using the current package manager's workspace syntax. Type-only imports still require a declared dependency; use the appropriate dependency category for its build/runtime/public declaration usage. Root dependency hoisting or `tsconfig.paths` is not a substitute.
- Define focused public entry points in `package.json` `exports`; use `import type` / `export type` for type-only symbols. Do not import `@repo/ui/src/...`, relative sibling app/package files, or bypass exports through aliases.
- Keep React runtime compatibility and peer dependencies consistent with workspace policy. Put Next client directives at interactive entry points; avoid marking the entire UI package as client-only. Do not make a reusable primitive depend on app providers, router state, configured API clients, or environment variables. Pass dependencies or wrap it in the app.
- Server packages accept dependencies/configuration and export factories or plugins. The consuming app creates/registers resources and owns cleanup. Importing a package must not start listening, connect to databases, or read app secrets. Preserve Fastify encapsulation and stored-procedure call boundaries when extracting code.
- Preserve the existing source-export versus compiled-package strategy. Source exports work only if every relevant consumer/tool can process them. For compiled packages, expose matching JS/type outputs and ensure dependency builds precede consumers. Never assume a Next bundler's TS support proves a deployed Fastify process can load the same runtime exports.
- Keep browser-safe and server-only entry points separate. A type-only import does not make an unsafe runtime import elsewhere acceptable.

## WebSocket capability placement

Use stable capability names such as `chat`, `notifications`, or `collaboration` for frontend `ws/services/<capability>/` and `ws/hooks/<capability>/`, and backend `modules/<capability>/ws/handlers/` / `publishers/`. Each operation/event gets its own file. `group` in an example is a group-chat capability placeholder; runtime group/room/tenant IDs are arguments, never source folders or dynamically renamed directories. Adding another group reuses the existing implementation.

Keep `ws/ws.config.ts` separate from transport and event code. App session/composition owns configured connections; subscription infrastructure tracks concurrent groups and consumer claims. Portable client code can live in `packages/websocket-client/src/ws/`, reusable React hooks in `packages/websocket-react/src/ws/hooks/`, shared event contracts in `packages/contracts/src/ws-types/`, and server mechanisms in `packages/server/src/ws/`. Keep app policy, credentials, lifecycle ownership, and cache/draft integration with their app owners. Declare package dependencies and use public exports; do not duplicate the code into a second app.

The owning [folder skill](../SKILL.md) contains its concrete frontend or backend layout. The optional `ws-standard-practices` skill covers the complete event lifecycle. Create only the layers/capabilities required by the task.

## Move and verify

1. Inspect workspace configuration, package exports, existing code and consumers. Choose the owner before generating the implementation, including before running component generators.
2. Move reusable implementation and its owned types/styles/helpers/tests together. Use stable props/dependency inputs; keep app configuration and workflow wrappers in the app. Update all consumers and remove duplicate implementations within the affected scope.
3. Update manifests, public exports, workspace lockfile, framework transpilation settings and Turbo tasks only where needed. Configure actual build outputs and upstream task dependencies for the chosen package strategy.
4. Run available package and consumer lint/typechecks/tests/builds through existing workspace tasks. For moved UI, verify rendering, interaction, styles and client/server boundaries. For backend extraction, verify registration, runtime resolution and resource cleanup. For contracts, check both producer and consumer plus schema/serialization compatibility.
5. When adding a later app, search and consume these packages first. A new app must reuse `@repo/ui/button` rather than generate another `Button` merely because its own component folder is empty.

This repository's package ownership conventions build on [Turborepo workspace structure](https://turborepo.dev/docs/crafting-your-repository/structuring-a-repository), [internal packages](https://turborepo.dev/docs/core-concepts/internal-packages), and [package creation](https://turborepo.dev/docs/crafting-your-repository/creating-an-internal-package).
