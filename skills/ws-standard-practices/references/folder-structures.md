# WebSocket Folder Structures

Preserve existing source-root and naming conventions. These examples separate responsibilities; optional files/packages should exist only when needed. Keep one event/operation per handler, service, and hook file. Group files by channel/capability; never consolidate unrelated events into a single domain-wide hook or service.

## Name folders by capability

Use the project's actual stable capability names. For example:

```text
ws/
  services/
    chat/
      sendMessage.ts
      subscribeMessages.ts
    notifications/
      subscribeNotifications.ts
  hooks/
    chat/
      useSendMessage.ts
      useMessageReceived.ts
    notifications/
      useNotificationReceived.ts
```

The later `group/` examples mean a group-chat capability; `chat/` is an equally valid name. Replace the example with the actual capability name consistently. `groupId`, `roomId`, and `documentId` remain runtime arguments. Never create folders per actual group or dynamically rename a source folder when joining/leaving. Backend equivalents are `modules/chat/ws/` and `modules/notifications/ws/`, with per-event handler/publisher files. Different capabilities may share connection infrastructure without sharing an omnibus service or hook.

## Fastify backend

```text
src/
  app.ts                             # dependency composition and plugin registration
  server.ts                          # listen, signals, shutdown
  plugins/
    websocket.ts                     # selected transport integration and cleanup
  ws/
    ws.config.ts                     # transport options from validated app configuration
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
        handlers/
          join-group.ts              # authorize and establish group subscription
          leave-group.ts
          send-group-message.ts      # validate/auth/dispatch this operation
        publishers/
          group-message-created.ts   # map committed result to authorized recipients
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
    ws.config.ts                     # browser-safe endpoint and transport options
    ws.ts                            # transport factory/adapter; no import-time connect
    subscriptions/
      groupSubscription.ts           # shared join/leave ownership for groupId
      groupMessageSubscription.ts    # shared message processing/cache-owner binding
    services/
      group/
        joinGroup.ts
        leaveGroup.ts
        sendGroupMessage.ts          # outgoing command and acknowledgement
        subscribeGroupMessages.ts    # incoming event; returns exact unsubscribe
        sendGroupTyping.ts           # only if supported
        subscribeGroupTyping.ts
    hooks/
      group/
        useSendGroupMessage.ts       # send action/pending/error state
        useGroupMessageReceived.ts   # subscribe and reconcile authoritative data
        useSendGroupTyping.ts
        useGroupTypingReceived.ts
    types/
      connection.types.ts            # client-only transport/status types
  features/
    chat/
      components/
        ChatPanel.tsx                # composes the required event hooks
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
    ws.config.ts
    ws.ts
    subscriptions/
      groupSubscription.ts
      groupMessageSubscription.ts
    services/
      group/
        joinGroup.ts
        leaveGroup.ts
        sendGroupMessage.ts
        subscribeGroupMessages.ts
    hooks/
      group/
        useSendGroupMessage.ts
        useGroupMessageReceived.ts
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
  websocket-client/src/ws/
    client.ts                        # reusable browser transport factory
    subscriptions/
      groupSubscription.ts           # reusable ownership manager, if portable
    services/group/
      sendGroupMessage.ts
      subscribeGroupMessages.ts
  websocket-react/src/ws/hooks/group/ # optional reusable React event hooks
  server/src/ws/                     # reusable server-only mechanisms, when needed
```

Package reusable capabilities from their first implementation even if only one frontend/backend currently exists; do not wait for a second app to duplicate the transport. Keep configured URL, credentials, active tenant, connection ownership, and app query/store integration in the app. If a domain service/hook is app-specific, keep it there rather than force it into a generic package. Package code accepts dependencies and never imports apps.

Even inside packages, keep event services and hooks under the package's `ws/` subtree. Keep configured `ws.config.ts` in the app; a package accepts options rather than importing app environment or credentials.

Example public import boundaries, configured in package exports:

```ts
import type { ChatServerEvents } from '@repo/contracts/ws/chat';
import { createWebSocketClient } from '@repo/websocket-client';
```

These names describe example project APIs, not exports supplied by this skill or APIs guaranteed by a transport library. Native WebSocket event envelopes and Socket.IO event maps should match the selected protocol rather than forcing one identical wire shape on both. For a standalone app, keep public contracts in its existing contract/types owner; do not create a Turborepo just to share a file.

## Group message flow

`group` is a capability folder. Group IDs are runtime arguments: use the same `useGroupMessageReceived(groupId)` for different groups rather than creating a hook or folder for each group ID. Example names below are proposed application event contracts, not built-in Socket.IO/native WebSocket events.

```text
Send:
ChatComposer
  -> useSendGroupMessage
  -> ws/services/group/sendGroupMessage
  -> shared connection: group:message:send
  -> backend ws/handlers/send-group-message
  -> domain service -> repository -> successful commit
  -> acknowledgement + ws/publishers/group-message-created

Receive:
authorized group broadcast: group:message:created
  -> ws/services/group/subscribeGroupMessages via shared event entry
  -> one app-owned message cache reconciliation
  -> useGroupMessageReceived(groupId) consumers observe updated data
  -> ChatMessageList re-renders
```

The outgoing command carries only the fields the application accepts, such as `groupId` and message content. Trusted sender/tenant identity comes from the authenticated session. Use a client message ID only if the application's idempotency protocol supports it. The published event contains its authoritative public message identity and group context, not raw database rows. Keep command, acknowledgement, and published-event types distinct in the contract package.

A receiving hook acquires the shared per-event entry through its service. That entry installs its processing binding before acquiring the shared group subscription; mounting another hook reuses the entry rather than installing another cache writer. The subscription manager joins on the first relevant consumer and leaves after the final consumer releases it. Group changes release the old scope and acquire the new one; late callbacks must not write to the new group's state. The sender can use the existing active subscription or receive its authoritative result through acknowledgement according to the chosen echo policy.

Send and receive remain separate even when a component composes both. Receiving is event-driven, not a polling hook or repeated receive command. A subscriber's cleanup removes only its own callback and releases its membership claim; it never disconnects the app-wide connection or calls a broad listener removal. Reconnection reacquires authorized scope and reconciles missed data through supported refetch/replay behavior.

Choose sender inclusion explicitly and reconcile by stable message identity, because acknowledgement, optimistic insertion, and broadcast can represent the same message. A lost acknowledgement after a committed send is an uncertain result, not proof of failure; do not automatically resend without the protocol's idempotency behavior.

## Stable UI and draft ownership

Keep the message-list subscription/cache separate from the composer's draft/form state. Incoming messages and background refetch must not reset or remount the composer. Use stable conversation identity, and clear only the submitted draft revision on the appropriate confirmed send outcome. See [lifecycle and UI-state rules](lifecycle-and-ui-state.md) for shared cache ownership, readiness states, recovery, and input-reset diagnosis.

## Existing installation migration

Install `ws-standard-practices` and remove the old `ws-be-standard-practices` / `ws-fe-standard-practices` installations using the consumer's skill manager. Removing the old source folders from this repository does not automatically remove copies already installed in another project. The merge itself does not authorize reorganizing a consumer application without a corresponding task.
