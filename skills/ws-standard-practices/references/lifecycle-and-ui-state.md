# Subscription Lifecycle and Stable UI State

Use this reference for shared group/event subscriptions, delivery reconciliation, and input resets during incoming activity. Retain one hook and one service file per operation/event under `ws/`; lifecycle infrastructure does not replace them with an all-events hook. The names below are application conventions, not library-provided APIs.

## Shared ownership without duplicate cache writes

```text
session provider
  -> one configured connection
  -> groupSubscription manager: join/leave leases per authorized group
  -> event subscription entries: one processing owner per event + scope
       -> per-event receive service registers through the manager/dispatcher
       -> authoritative cache reconciliation once per received event
       -> optional independent consumer callbacks/status subscribers

useGroupMessageReceived(groupId) may mount in multiple components
  -> each acquires/releases its own claim
  -> all share the same scoped event processing and group membership
```

The connection/session owner constructs the managers and supplies an app-owned cache reconciliation callback. Services and shared packages do not import an app's QueryClient singleton, provider, or store. Keep transport dispatch, group membership, and event processing responsibilities distinct even if a small implementation keeps infrastructure in one module. Put reusable subscription infrastructure under `ws/subscriptions/`, with one entry per logical event/scope and explicit ownership of cleanup.

Key scope by the connection/session generation and the relevant tenant/group identity, not by an unstable render object or callback. For a native transport, the connection dispatcher may already own the physical message listener; share event bindings through it rather than installing another listener per component. For Socket.IO, share the logical event subscription using its real event API.

- The first group claim starts an authorized join; the final release initiates the corresponding leave when applicable. Different received-event entries, such as messages and typing, can hold claims on the same group.
- Bind event processing before the join can start delivery. Components registering callbacks get their own unsubscribe handle; releasing one must not remove the shared processing entry while other claims remain.
- Process a valid event once through the authoritative cache owner, then notify independent consumers as needed. Do not let every hook callback append the same record. Isolate a consumer callback failure from the rest of the fan-out and keep error handling observable.
- Share event processing and also reconcile by authoritative message identity/version: duplicate network delivery, acknowledgement, optimistic state, and replay are a separate problem from duplicate mounted consumers. Use immutable or functional cache updates according to the existing state library; do not mutate cached arrays in place.
- Handle a final release while join is still pending. A late join acknowledgement must not activate an abandoned group; release/compensate only for its own generation. Switching tenants, logout, reconnect, and a new acquisition must not let old cleanup close a new connection or membership lease.

## Connection is not channel readiness

Use existing status types when available, but distinguish these meanings:

| Scope | States and meaning |
|---|---|
| Transport | Disconnected, connecting, connected, reconnecting, or the library's equivalent. |
| Group subscription | Idle, joining, subscribed, failed, and leaving when relevant to the UI. `subscribed` requires actual protocol confirmation, not merely a connected transport. |
| Send operation | Pending; acknowledged with authoritative result; explicitly rejected; or uncertain because confirmation was lost/timed out. |

A join rejection should not appear as a global connection failure. During reconnect, do not continue to label the old group as currently subscribed. Preserve the distinction between having cached history and receiving live updates. If the protocol has no join confirmation, expose its actual readiness limits rather than fabricating a guarantee.

## Delivery and recovery

Define the reconciliation strategy against the current API/protocol before implementing a new one:

- If replay is supported, use its actual event identity/cursor and resume boundary. Detect gaps/duplicates according to that contract; timestamps alone are not a reliable universal ordering key.
- If authoritative history/refetch is the recovery mechanism, coordinate it with the live subscription so a late response cannot overwrite newer live records. For example, buffer relevant live events during a scoped fetch and merge them by stable identity/version, or use an existing server snapshot/cursor protocol. Merely "fetch, then listen" leaves a gap.
- Clear temporary typing/presence indications according to their real expiry policy. Do not persist or buffer them for replay as if they were messages. Transient activity should not become a backlog of stale notifications.
- On uncertain sends, retain reconciliation context and show the supported retry/check path. Do not report rejection, roll back committed data, or automatically create a duplicate write merely because the acknowledgement did not arrive.
- Choose stronger publication durability only when the product requires it. A commit followed by broadcast can still lose the notification if the process fails between them. Use the established replay/refetch approach or a supported durable publication mechanism where necessary; do not add infrastructure unconditionally.

Socket.IO describes default delivery limits in [delivery guarantees](https://socket.io/docs/v4/delivery-guarantees/) and explicitly notes recovery can fail in [connection-state recovery](https://socket.io/docs/v4/connection-state-recovery/). These application ownership conventions do not change those guarantees.

## Input resets: find the actual cause

First reproduce with a partially typed draft and one incoming event. Inspect component identity, state updates, form behavior, and the rendered tree. A render log alone does not show a remount. Use the existing React debugging tools to inspect mounts/keys and trace the code that changes the field value or resets the form.

| Cause to inspect | Correct ownership/fix |
|---|---|
| Composer or ancestor keyed by message count, newest message ID, random value, or timestamp | Keep stable identity for the same conversation; never tie composer identity to message-list changes. |
| Component function declared inside a frequently rendering parent | Declare the component at module scope so its type remains stable. |
| Loading/reconnect branch replaces the whole conversation with a spinner | Keep the composer and existing content mounted; render the relevant status separately. |
| Draft value is derived from received messages or overwritten by a socket/cache effect | Keep draft/form state independent of server message data. Update history without assigning to the draft. |
| An effect resets form values whenever query data or a fresh object changes | Initialize deliberately and reconcile server changes without overwriting dirty user edits. |
| Acknowledgement clears whichever draft happens to be current | Associate clearing with the submitted conversation and draft revision. Do not clear text entered after submission. |
| Input alternates between controlled/uncontrolled values or transforms text during entry | Use a consistent input contract and synchronous input updates; preserve composition and selection behavior. |

For React Hook Form, inspect `reset`, `resetField`, reactive `values`, unregister-on-unmount behavior, and initialization effects. Ordinary rerenders do not by themselves mean cached `defaultValues` reset. If server-fed values change, adopt an explicit dirty-field/conflict policy using the installed library's supported options; do not repeatedly reset the entire form on each incoming event. A newly allocated `defaultValues` object alone is not proof of the cause.

## Composer and received data are separate owners

```text
ChatScreen (stable conversation identity)
  ChatMessageList
    -> useGroupMessageReceived(groupId)
    -> existing query/cache for received message data
  ChatComposer
    -> local form/draft state (or the app's scoped draft store)
    -> useSendGroupMessage(groupId)
```

This separation limits unnecessary subscriptions, but correctness still depends on stable component identity and explicit state ownership. An ordinary parent rerender remains valid; blanket memoization or suppressing effect dependencies is not the fix for lost input. Profile actual rendering cost before adding performance optimizations. Keep high-frequency message data out of a connection-only context where possible; consumers should select only the data/status they need.

Draft lifetime follows product intent. If changing conversation intentionally resets the composer, key only by that stable conversation identity and apply the documented policy. If drafts must survive switching away and back, keep them in an app-owned draft map keyed by account/tenant/conversation. Do not automatically persist sensitive drafts to browser storage or retain them across logout.

When sending, capture the submitted scope and a draft revision/token alongside the content. Only clear that submitted revision on the actual successful outcome required by the product. If the composer clears immediately by explicit optimistic policy, keep recovery for that submitted text separate; failure must not overwrite a newer draft with the old text. An acknowledgement from group A must never clear the current draft for group B, including when two drafts have identical text.

See React's [state identity guidance](https://react.dev/learn/preserving-and-resetting-state), [input contract](https://react.dev/reference/react-dom/components/input), and the installed [React Hook Form API](https://react-hook-form.com/docs/useform) for the underlying framework behavior. The draft revision, subscription, and recovery ownership rules above are application design guidance.

## Regression scenarios

Use the consumer project's existing test setup for actual implementation changes. These are behavioral acceptance cases, not tests that simply match code structure:

1. Type part of a message; receive messages and typing events. The draft value, focus, selection, and active input composition remain intact while history updates.
2. Mount two message consumers and a typing consumer for the same group. One delivered message produces one cache reconciliation; unmounting one consumer leaves the others subscribed. Final cleanup releases its resources.
3. Join a group slowly, then switch group or log out before confirmation. Late acknowledgement/event/cleanup cannot activate the abandoned scope or affect the replacement.
4. Send draft A, then type draft B before A is acknowledged. A's result reconciles its message but does not clear or replace B. Repeat while switching groups and with identical text in different draft revisions.
5. Lose the acknowledgement after server commit, then receive an echo or replay. The message reconciles once; no blind resend or false rejection is reported.
6. Reconnect while fetching history and receiving live messages. No supported message is lost or appended twice; a late fetch cannot overwrite newer live state. Temporary typing expires instead of replaying.
7. Trigger background refetch/reconnect without changing conversation. The composer is not remounted or reset, and form dirty values survive. Explicit discard and the chosen conversation-switch policy still work.
8. Exercise React development setup/cleanup cycles and auth/tenant changes. Listener/membership counts return to the expected baseline, and stale-session events cannot update current data or drafts.

For a skill-only edit, validate metadata and local references and review these scenarios for coherent ownership. Do not claim a consumer application's draft-loss bug is fixed without inspecting and testing that application.
