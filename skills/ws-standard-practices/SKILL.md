---
name: ws-standard-practices
description: Structure WebSocket and Socket.IO logic across backend services and React/Next.js clients. Use for connection ownership, domain event handlers/services, subscription hooks, cleanup, state synchronization, and shared Turborepo event contracts; preserve the project's chosen protocol.
---

# WebSocket Structure and Practices

Separate transport lifecycle from domain messages and UI consumption. This skill replaces `ws-be-standard-practices` and `ws-fe-standard-practices` with one end-to-end ownership model.

## Inspect first

- Read repository instructions, workspace packages, existing folder conventions, transport/plugin versions, authentication, event schemas, state ownership, deployment, and tests. Apply the backend/frontend folder-structure skills when available; do not create a competing tree in a working project.
- Identify whether the application uses native WebSocket, `ws`/`@fastify/websocket`, or Socket.IO. Preserve that choice. Socket.IO events, rooms, acknowledgements, and reconnection APIs are not native WebSocket APIs; do not mix their clients and servers.
- Identify who creates the connection, who subscribes, who removes each listener, and who closes the connection. Search existing implementations before adding another connection or event contract.
- Use [folder structures](references/folder-structures.md) when creating or moving files. Names are defaults; create only files with actual responsibilities. Do not generate empty layers.

## Backend responsibilities

| Owner | Responsibility |
|---|---|
| Application factory / feature assembly | Construct dependencies, register infrastructure before consumers, and wire domain handlers. |
| `plugins/websocket.ts` | Integrate the selected transport with Fastify, registration scope, and application shutdown. Reuse an existing equivalent. |
| `ws/connection.ts` | Connection/session lifecycle, identity context, dispatcher binding, and cleanup. No domain queries or business policy. |
| `ws/routes.ts`, when applicable | Native WebSocket upgrade endpoint and transport-level binding. Socket.IO attaches through its actual server integration; do not invent Fastify HTTP routes for each Socket.IO event. |
| `modules/<domain>/ws/index.ts` | Bind the domain's accepted events to handlers using injected dependencies. Return/provide cleanup where registration needs it. |
| `modules/<domain>/ws/handlers.ts` | Validate/interpret incoming events, authorize the operation with trusted context, call domain services, and map acknowledgements/errors. No direct persistence. |
| `modules/<domain>/ws/publisher.ts`, when needed | Map domain results to public events and authorized recipients through an injected publisher. Keep transport details out of business services. |
| Domain `service.ts` / `repository.ts` | Reusable application operations and persistence, also usable by HTTP controllers. Do not import sockets, request/reply objects, or the listening application here. |
| Domain `ws/schemas.ts`, when needed | Runtime validation of transport payloads; reuse portable shared schemas rather than duplicate them. |

Typical incoming flow:

```text
transport endpoint / connection dispatcher
  -> domain event handler -> domain service -> repository
  -> acknowledgement / public event mapping
```

HTTP and WebSocket adapters may call the same domain operation. Do not call an HTTP controller from a socket handler or reimplement business rules in both. Preserve the application's transaction and stored-procedure boundaries. Publish committed results at the established commit boundary; do not announce a successful change before it succeeds.

Authenticate the connection and authorize every requested operation/subscription against trusted actor/tenant context. A client-supplied room, channel, or identifier is not authorization. Parse external payloads as untrusted data and validate them before dispatch. Map errors at the transport boundary without exposing internal details. Remove connection-owned listeners/subscriptions/timers on close; the app closes its owned server resources on shutdown.

For `@fastify/websocket`, register the plugin before routes and attach message listeners synchronously in the route handler before asynchronous work can lose arriving messages. Handle asynchronous message failures in the message-processing boundary; do not assume HTTP error handling catches them. Preserve encapsulation and the installed plugin's lifecycle APIs.

## React client responsibilities

| Owner | Responsibility |
|---|---|
| `ws/ws.ts` | Transport factory/adapter: create, connect, disconnect, connection status, and protocol-specific send/listen behavior. Do not connect at module import. |
| `providers/WebSocketProvider.tsx` or existing session owner | Own the configured connection for its browser session or explicitly scoped workspace. Supply URL/auth/config, react to identity changes, and close the owned connection. |
| `ws/services/<domain>SocketService.ts` | Named domain operations and subscriptions over the supplied transport. No React, JSX, UI imports, or hidden connection creation. Subscriptions return an unsubscribe function. |
| `ws/hooks/use<Domain>Socket.ts` | React subscription lifecycle, UI-facing status/actions, and integration with existing query/store ownership. Consume services, not raw transport APIs. |
| `ws/types/` | Client-only adapter/subscription types; portable public payloads belong in the shared contract owner. |
| Feature components / containers | Consume hooks and render application states. No raw `new WebSocket`, `.on`, `.emit`, or `.send` in UI code. |

```text
UI -> domain hook -> domain socket service -> transport adapter -> backend
                       ^
session/provider owns configured connection and supplies it to consumers
```

- One connection per deliberate session/scope, not one per rendered component or hook. Independent backend endpoints may justify separate connections. A hook unsubscribes its listeners on cleanup; it must not disconnect a connection shared by other consumers.
- Remove the exact listener installed by this subscription. For Socket.IO, do not use a broad `off(event)`/`removeAllListeners` cleanup that deletes another consumer's listener. Do not register duplicate domain listeners on every reconnect. Shared channel join/leave ownership must support multiple subscribers; one hook unmounting must not unsubscribe the others.
- Effects must tolerate setup/cleanup/setup and dependency changes. Keep dependencies accurate and callbacks current rather than suppressing checks. Scope connections to browser app instances; do not use a server-process singleton shared across users.
- Server data stays with its established owner: update or invalidate the relevant TanStack Query cache when used. Put client UI state in its existing store. Do not duplicate every server record into a separate socket store. If an event is only a notification of change, refetch rather than guess the complete new record.
- Define reconnect and resynchronization against the actual protocol/product capabilities. Preserve supported subscriptions and refetch/replay as appropriate; never promise exactly-once delivery or lossless recovery just because a connection reopened. Do not queue/retry consequential mutations without the application's idempotency rules. Close or replace the old authenticated scope on logout/account switching and prevent late events from updating the new session.

## Next.js boundary

Use the same transport -> services -> hooks layering as React. Put the provider and hook-consuming UI behind the appropriate `'use client'` boundary; keep browser connections out of Server Components, module initialization, server actions, and server rendering. A Server Component layout may render the client provider with serializable configuration; do not pass a socket object or server secrets across that boundary. Scope the provider to the routes/session that actually need it rather than marking the entire application client-only.

The Next frontend can connect to the separate Fastify service. Do not assume a Next route handler or hosting platform supports a long-lived upgraded connection; verify the actual runtime/deployment before choosing a server host.

## Turborepo ownership

- Public event names/maps, client-to-server and server-to-client payloads, acknowledgement shapes, and portable runtime schemas belong in `packages/contracts/src/ws-types/` and its schema owner. Both apps import public package exports. Keep Fastify/Socket.IO socket instances and database models out of browser-safe contracts.
- Reusable browser transport factories and domain protocol helpers belong in `packages/websocket-client` or an existing focused package from their first implementation. Reusable React subscription adapters may live in `packages/websocket-react` when they have a distinct responsibility. UI-specific side effects remain in the app.
- Reusable server transport/plugin mechanisms belong in `packages/server/src/ws/` or the established server package. App-specific handlers, authorization policy, dependency composition, and configured lifecycle remain in `apps/api`.
- Consumers declare direct package dependencies and use public exports. No cross-app source imports, deep imports into package internals, duplicated frontend/backend payload definitions, or packages importing apps. Preserve source-export versus compiled-package strategy and verify both browser and backend runtime resolution.

## Verify changes

Check the relevant scenarios in the actual consumer project: connect/close, duplicate consumer mount/unmount, React development remount, auth scope change, reconnect/resync, malformed payload, unauthorized subscription, event-to-cache behavior, and application shutdown. Verify absence of import-time connections and that unmounting one consumer does not break another. Use the project's existing lint/typecheck/tests/build; broaden to affected packages/apps for shared contracts. Do not add mirrored tests for a documentation-only change.

Report the chosen owners, changed contract/import paths, performed checks, and remaining limitations. Type checking does not validate event payloads or prove delivery guarantees.

## Protocol and framework references

Consult the installed version's guidance when implementing behavior: [Fastify WebSocket integration](https://github.com/fastify/fastify-websocket), [React effect lifecycle](https://react.dev/reference/react/useEffect), [Next.js client boundary](https://nextjs.org/docs/app/api-reference/directives/use-client), and [Socket.IO listener ownership](https://socket.io/docs/v4/listening-to-events/). This skill defines repository placement conventions; these sources describe the underlying APIs.
