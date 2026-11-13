# Writing Endpoint callback code

Load this card when a task writes or changes the App code that serves an
Endpoint or calls one: the callback, replies, subscription snapshots and
reports, liveness, and checks of an optional peer capability. Its rules are
generated from the
[Endpoint runtime rules](../reference/protocols.md#endpoint-runtime-rules)
and their neighbouring sections, which own them. Exact callback and function
signatures belong to the installed public SDK; the protocol itself follows the
[Endpoint protocol card](endpoint_protocol.md).

## Order of work

1. Branch on the callback action and keep peers and transaction IDs opaque
   (`runtime.*`).
2. Answer every Request and SubscriptionRequest with its correlated reply, in
   send order (`ordering.send-order.authoring`,
   `runtime.subscription-snapshot-response`), and start a stream only on its
   callback (`sub.server.*`).
3. Check an optional server capability before sending it
   (`conformance.optional-runtime-check`), then build and test the App.

<!-- generated:codebot-rules task=endpoint-runtime -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Quick usage reminder

Owner: [endpoint-lifecycle.md#quick-usage-reminder](../reference/endpoint-lifecycle.md#quick-usage-reminder)

- `lifecycle.subscription-access-privileges`: Each user's subscription remains subject to that user's access privileges.
- `lifecycle.sdk-owns-guarantees`: The public SDK comments own exact callback ordering, timeout, resubscription, and removal guarantees; Apps have no additional acknowledgement API.

### Request and response ordering

Owner: [protocols.md#request-and-response-ordering](../reference/protocols.md#request-and-response-ordering)

- `ordering.send-order.authoring`: Handle human-task requests and emit correlated replies in [request send order](../reference/protocols.md#request-and-response-ordering), including calculations, Error and EditRequest replies.

### Required and optional capabilities

Owner: [protocols.md#required-and-optional-capabilities](../reference/protocols.md#required-and-optional-capabilities)

- `conformance.optional-runtime-check` (judgment): Configuration matching proves only mandatory compatibility. The server does not know the client's complete declaration. A client that wants to use an optional server payload must check the configured server's runtime variant index before sending the request. In Rust use `VariantIndex` and `variant_index!`; a nested optional payload may require a chain of variant-index checks.

### Default and explicit subscriptions

Owner: [protocols.md#default-and-explicit-subscriptions](../reference/protocols.md#default-and-explicit-subscriptions)

- `sub.server.start-on-callback`: The server starts/renews its application stream state only after receiving the current SubscriptionRequest callback. Host or App restart does not authorize a server to emit data from remembered application state without a fresh routed subscription callback.

### Endpoint runtime rules

Owner: [protocols.md#endpoint-runtime-rules](../reference/protocols.md#endpoint-runtime-rules)

- `runtime.branch-on-action`: Branch on the callback action before decoding `data`. `PeerUp`, `PeerDown`, `PeerAlive`, and `PeerTimeout` are signals and carry no application payload.
- `runtime.peer-opaque`: A server-side peer value is opaque. Retain it and pass it back unchanged. Never interpret it as an App, task, user, endpoint, session, or object ID.
- `runtime.echo-transaction`: Return the exact nonzero transaction ID and peer supplied with a Request when sending its Response.
- `runtime.subscription-snapshot-response`: Complete an accepted SubscriptionRequest with a correlated Response carrying the complete current snapshot, echoing its transaction ID and peer. A complete snapshot is therefore always a `Response`. When the same variant also delivers later complete reports, declare both `Response` and `SubscriptionData` on it. A variant that delivers only partial reports, such as a [reduced](../reference/protocols.md#reduced-subscriptiondata-design-pattern) or [appended](../reference/protocols.md#subscriptiondata-array-append) report, carries `SubscriptionData` alone and need not be a `Response`. Publish later changes through the report API.
- `runtime.report-not-reply`: Sending only a report does not complete the opening transaction and can leave the client waiting indefinitely. Rejection also requires a correlated reply.
- `runtime.peer-fanout`: Omit the peer for normal common reports/liveness and let the host fan out. Target a peer only with a currently valid callback-provided handle.
- `runtime.peer-alive`: Use `PeerAlive` when a liveness cadence is required without changed data. Never fabricate a schema-level “no change” value or resend unchanged data merely to reset a watchdog.
- `runtime.peer-alive-no-state` (judgment): `PeerAlive` does not update data, cache, UI, sequence, cursor, or recovery state. Protocols that need loss detection carry sequence/revision/snapshot information in real application messages.
- `runtime.no-private-records` (judgment): Do not model private acknowledgements or host subscription records in App code.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

<!-- /generated -->
