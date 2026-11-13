# WebSocket Folder Structures

Preserve existing source-root and naming conventions. These examples separate responsibilities; optional files/packages should exist only when needed. A small domain can keep a few handlers together and split by operation as it grows.

## Fastify backend

```text
src/
  app.ts                             # dependency composition and plugin registration
  server.ts                          # listen, signals, shutdown
  plugins/
    websocket.ts                     # selected transport integration and cleanup
  ws/
    connection.ts                    # session lifecycle and dispatch infrastructure
    routes.ts                        # native WS upgrade endpoint, only if applicable
    types.ts                         # server-internal connection/session context
  modules/
    chat/
      index.ts                       # feature assembly; wires HTTP and WS adapters
      routes.ts                      # HTTP routes, if this feature has them
      controller.ts                  # HTTP adapter, if needed
      service.ts                     # domain operations, reused by HTTP and WS
      repository.ts                  # persistence
      types.ts                       # internal domain/persistence types
      ws/
        index.ts                     # domain event bindings
        handlers.ts                  # validation/auth/dispatch to domain operations
        publisher.ts                 # outgoing payload/recipient mapping, if needed
        schemas.ts                   # schema wiring or server-only event validation
      tests/
        ws.test.ts                   # actual event behavior/authorization/lifecycle
```

Keep common connection infrastructure in root `ws/`; keep chat policy and event handling under `modules/chat/ws/`. Add `notifications/ws/` alongside it when needed, rather than growing one global message handler. Composition injects domain bindings into the connection dispatcher. A shared native endpoint may route validated message envelopes to these bindings; Socket.IO binds its named events through its own server API.

No parallel `ws/service.ts` should duplicate `modules/chat/service.ts`. Handlers adapt messages; domain services own business operations. A publisher can accept a recipient abstraction so business logic does not depend on a transport instance. Authentication mechanisms can be shared; resource-level authorization still applies to each operation.

## React frontend

```text
src/
  providers/
    WebSocketProvider.tsx            # configured connection/session owner
  ws/
    ws.ts                            # transport factory/adapter; no import-time connect
    services/
      chatSocketService.ts           # sendMessage, subscribeMessages, unsubscribe
      notificationSocketService.ts   # domain operations, when needed
    hooks/
      useChatSocket.ts               # effect lifecycle and UI-facing domain API
      useNotificationsSocket.ts
    types/
      connection.types.ts            # client-only transport/status types
  features/
    chat/
      components/
        ChatPanel.tsx                # consumes useChatSocket
      types/                         # view-only types, when needed
```

The provider creates/owns the connection; hooks borrow it. Services take a transport dependency, so they are testable without React or an application singleton. Hooks own their exact listeners and UI integration. Do not put WebSocket hooks among HTTP query hooks or introduce a second API client in UI components.

If the project already colocates integration code under `features/chat/ws/`, preserve that organization: keep the same services/hooks separation and the shared session owner. Do not create both root and feature copies. In a `containers/`-based project, the chat container replaces `features/chat/components/`; the WebSocket layers are unchanged.

## Next.js App Router frontend

```text
src/
  app/
    layout.tsx                       # server layout by default
    (workspace)/
      layout.tsx                     # wraps required routes with the client provider
      chat/
        page.tsx                     # route composition
  providers/
    WebSocketProvider.tsx            # 'use client'; owns browser connection
  ws/
    ws.ts
    services/
      chatSocketService.ts
    hooks/
      useChatSocket.ts
    types/
      connection.types.ts
  features/
    chat/
      components/
        ChatPanel.tsx                # client entry when imported from server UI
```

Client boundaries apply to the provider and interactive consumers, not every source file. The transport/service modules remain framework-independent where possible and safe to import without opening a connection. In Next Pages Router, compose the equivalent provider through `_app.tsx` or the existing authenticated layout, not a second App Router tree. For projects without `src/`, use the same owners under the existing root.

## Turborepo

```text
apps/
  api/src/                           # configured backend and app-specific domain handlers
  web/src/                           # React/Next composition, provider, feature UI
packages/
  contracts/src/
    ws-types/
      chat.types.ts                  # public event maps/payloads/acknowledgements
      notifications.types.ts
    schemas/                         # portable runtime schemas, if supported
  websocket-client/src/
    client.ts                        # reusable browser transport factory
    services/
      chatSocketService.ts           # portable protocol helper, if reusable
  websocket-react/src/               # optional shared React adapters/hooks
  server/src/ws/                     # reusable server-only mechanisms, when needed
```

Package reusable capabilities from their first implementation even if only one frontend/backend currently exists; do not wait for a second app to duplicate the transport. Keep configured URL, credentials, active tenant, connection ownership, and app query/store integration in the app. If a domain service/hook is app-specific, keep it there rather than force it into a generic package. Package code accepts dependencies and never imports apps.

Example public import boundaries, configured in package exports:

```ts
import type { ChatServerEvents } from '@repo/contracts/ws/chat';
import { createWebSocketClient } from '@repo/websocket-client';
```

These names describe example project APIs, not exports supplied by this skill or APIs guaranteed by a transport library. Native WebSocket event envelopes and Socket.IO event maps should match the selected protocol rather than forcing one identical wire shape on both. For a standalone app, keep public contracts in its existing contract/types owner; do not create a Turborepo just to share a file.

## Existing installation migration

Install `ws-standard-practices` and remove the old `ws-be-standard-practices` / `ws-fe-standard-practices` installations using the consumer's skill manager. Removing the old source folders from this repository does not automatically remove copies already installed in another project. The merge itself does not authorize reorganizing a consumer application without a corresponding task.
