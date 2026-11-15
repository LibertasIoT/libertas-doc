# Designing subscriptions

Load this card when a task adds or changes a `SubscriptionRequest` or
`SubscriptionData` variant: default and explicit subscriptions, the complete
initial snapshot, compatible updates, reduced reports, array append or live
charts. Its rules are generated from the
[compatible SubscriptionData contract](../reference/protocols.md#compatible-subscriptiondata)
and the neighbouring subscription sections, which own them. Copy paths follow
the [shared copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules);
the callback code that answers and publishes follows the
[Endpoint callback card](endpoint_runtime.md).

## Order of work

1. Choose the default and explicit subscriptions (`sub.default.*`) and make
   every subscription return a complete snapshot (`sub.snapshot.*`).
2. Choose the update shape: complete snapshots only, compatible updates
   (`sub.compatible.*`), reduced reports (`sub.reduced.*`) or appends
   (`sub.append.*`, `sub.chart.*`).
3. Design the App's consistency across every mutation path
   (`sub.consistency.*`, `sub.partial-update.*`), then compile with Studio
   and fix every reported rule.

<!-- generated:codebot-rules task=subscriptions -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Choosing a role

Owner: [protocols.md#choosing-a-role](../reference/protocols.md#choosing-a-role)

- `protocol.choice.list` (judgment): **Choosing the role for a need.** Each need has one role; the linked section owns its rules. Never substitute one role for another.
- `protocol.choice.live-data` (judgment): **Data that changes while watched:** `SubscriptionRequest` with `SubscriptionData`; the first Nil SubscriptionRequest is the default ([subscriptions](../reference/protocols.md#default-and-explicit-subscriptions), [compatible data](../reference/protocols.md#compatible-subscriptiondata)).

### Compatible SubscriptionData

Owner: [protocols.md#compatible-subscriptiondata](../reference/protocols.md#compatible-subscriptiondata)

- `sub.compatible.name-merge` (judgment): On an incoming compatible SubscriptionData report, retain A as the working type and screen, including its presentation attributes and NextRequest actions. A report with explicit CopyTo uses the [reduced subscription design pattern](../reference/protocols.md#reduced-subscriptiondata-design-pattern). For the name-matched form without CopyTo, match fields by exact native name at the struct root. Replace each matching field's complete value; do not recursively patch nested records or merge array elements. Preserve fields present only in A and ignore fields present only in the incoming report.
- `sub.compatible.null-and-hidden` (judgment): Valid null is a value and clears a matching nullable field; undefined is never transmitted. Hidden fields participate in this merge like visible fields.
- `sub.compatible.atomic-merge.authoring`: Ensure compatible updates preserve the [complete working value’s constraints and meaning](../reference/protocols.md#compatible-subscriptiondata).
- `sub.compatible.reference-context` (judgment): For actions originating from that subscription screen, the merged A supplies the SubscriptionData object in the [reference context](../reference/protocols.md#interaction-reference-context), which also retains the latest successful correlated Response received in that subscription. A report does not replace the Response object or the context of another subscription or displayed value. A compatible update does not navigate, submit a request, or execute the incoming variant's actions. Its delivery scope follows the [shared role rule](../reference/protocols.md#protocol-role-reuse).
- `sub.compatible.app-choice` (judgment): Sending compatible reports is entirely the App's choice and responsibility. Declaring compatibility never instructs the Hub or clients to generate, request, or synthesize such reports. Apps may continue sending complete snapshots only.
- `sub.compatible.complete-initial-snapshot` (judgment): Every SubscriptionRequest, including a repeated default subscription, must return the current complete initial snapshot. The App owns snapshot/update ordering and must not rely on a new subscriber having seen earlier reports.

### Relationship to compatible SubscriptionData

Owner: [protocols.md#relationship-to-compatible-subscriptiondata](../reference/protocols.md#relationship-to-compatible-subscriptiondata)

- `sub.append.no-delivery-permission` (judgment): AppendTo does not grant permission to deliver a report to a screen. A compatible append report arriving before its required initial snapshot is discarded, never replayed later.

### Attribute declarations

Owner: [protocols.md#attribute-declarations](../reference/protocols.md#attribute-declarations)

- `subscription.append.destination-root-array`: Its value is exactly one `Variant.field` destination, using native names in the same protocol. The named variant is a working SubscriptionData that admits this report through NextResponse. Its named field must be a root array. Nested paths and per-item targets are out of scope.
- `sub.append.index-length`: That field must be a non-nullable integer Number, carrying the expected destination length before the append. Its value must be non-negative.
- `sub.append.index-hidden` (judgment): The index field should be Hidden; visibility has no effect on validation.

### Studio validation and App responsibilities

Owner: [protocols.md#studio-validation-and-app-responsibilities](../reference/protocols.md#studio-validation-and-app-responsibilities)

- `sub.append.app-owns-ordering` (judgment): The App owns ordering, duplicate-report prevention, retention, and complete initial snapshots for new subscriptions. If one append report serves multiple working screens, the App must ensure each intended recipient has the required array prefix. Under the ordered-delivery contract, an index mismatch exposes a breach of that expected append sequence. Matching lengths do not prove equal contents; AppendIndex is a position-consistency check, not a content checksum.

### Combined copy and append

Owner: [protocols.md#combined-copy-and-append](../reference/protocols.md#combined-copy-and-append)

- `sub.append.combined-copy.authoring`: Combine CopyTo with AppendTo only with unique copy destinations, excluding append and index control fields from the [ordinary copy projection](../reference/protocols.md#combined-copy-and-append).

### Live chart use

Owner: [protocols.md#live-chart-use](../reference/protocols.md#live-chart-use)

- `sub.chart.app-responsibilities` (judgment): Ordering, retention, and fresh-subscriber snapshots remain App responsibilities.
- `sub.chart.complete-initial-snapshots`: Sending only new elements and update metadata reduces update-message size compared with retransmitting the complete accumulated array; initial snapshots remain complete. This is a use of the shared SubscriptionData mechanism, not a separate chart patch format or an automatic scrolling requirement.

### Reduced SubscriptionData design pattern

Owner: [protocols.md#reduced-subscriptiondata-design-pattern](../reference/protocols.md#reduced-subscriptiondata-design-pattern)

- `sub.reduced.optional` (judgment): **Reduced SubscriptionData** is an optional design pattern an App can adopt using Libertas subscriptions, `NextResponse`, `CopyTo`, and typed copying. It introduces no new protocol message role or schema attribute. An App may instead publish complete snapshots for every update.
- `sub.reduced.copyto`: Each reduced Protocol Variant declares `CopyTo` with one or more comma-delimited Libertas Schema Paths identifying the stable values or subtrees it updates.
- `sub.reduced.native-name-vocabulary` (judgment): The pattern also reuses source-code field identifiers as a shared semantic vocabulary. An App can give a Native Name one consistent documented meaning across full state, partial requests, and reduced reports. Compatible types and explicit paths establish valid field correspondence and select the intended occurrences; the same identifier can appear in several structures. The App authors these semantic relationships. See [static and runtime validation](../reference/schema.md#9-static-and-runtime-validation) for the shared checks on declarations and the separate validation of values.

### Acceptance and destination are separate

Owner: [protocols.md#acceptance-and-destination-are-separate](../reference/protocols.md#acceptance-and-destination-are-separate)

- `sub.reduced.acceptance-separate`: The SubscriptionRequest's `NextResponse` selects its complete initial snapshot. The stable SubscriptionData's `NextResponse` admits later reduced reports. The reduced SubscriptionData's `CopyTo` specifies where its payload is copied. CopyTo does not grant delivery permission or infer an acceptance relationship. A reduced report must satisfy both the existing subscription/ownership checks and the stable variant's explicit acceptance declaration.

### Per-destination resolution and typed copying

Owner: [protocols.md#per-destination-resolution-and-typed-copying](../reference/protocols.md#per-destination-resolution-and-typed-copying)

- `sub.reduced.per-recipient` (judgment): Resolve CopyTo against each eligible **stable working value** under the [shared mapping and destination-uniqueness rules](../reference/schema.md#shared-copyfrom-and-copyto-rules). One publication may reach both a ZoneState screen and a ZoneSettings screen; each independently resolves one destination while retaining its type and actions. Each recipient retains its own Endpoint, schema, subscription identity/parameters, and lifetime. There is no global atomic transaction across different recipients.
- `sub.reduced.whole-array-replace`: A field carrying a complete array with the destination's element type replaces that whole array, including removals and an empty array. This differs from a reduced-element array projection, which retains the correspondence required by its copy contract. CopyTo does not imply AppendTo, insertion, or arbitrary recursive JSON patch semantics.

### Snapshot, mutation, and publication consistency

Owner: [protocols.md#snapshot-mutation-and-publication-consistency](../reference/protocols.md#snapshot-mutation-and-publication-consistency)

- `sub.snapshot.fresh-each-subscription`: Every SubscriptionRequest, including repeated subscription and resubscription, receives a fresh complete current stable snapshot. A reduced report arriving before its required baseline is discarded, not saved for later replay.
- `sub.snapshot.capture-no-loss` (judgment): Snapshot capture and the transition to subsequent reports must not lose a committed change or reorder publication. Renew the baseline through a subscription request when the object, filter, or subscription context changes.
- `sub.partial-update.acknowledgement` (judgment): A partial-update Request changes authoritative App state after validation and recomputes affected derived values. Its correlated Response acknowledges that request; subscription publication is a separate operation. Use an ordinary non-transition success acknowledgement when the working screen should remain, and Error for a rejected request. A rejection must not publish the proposed values as accepted state.
- `sub.consistency.app-owned` (judgment): The App is responsible for designing its protocol and transactions to preserve data consistency across every permitted interaction and state-transition path. This includes partial/full requests, concurrent clients, devices, timers, background work, restored state, subscriptions, and restart recovery. The App coordinates authoritative state changes, affected derived or hidden values, and publication order; fields that must change together belong in one atomic report, or the protocol must explicitly coordinate their transition. NextResponse and CopyTo describe acceptance and typed placement. Schema and runtime checks enforce their declared structural constraints, but do not prove application-level consistency across all paths or supply missing transaction logic.
- `sub.consistency.reports-equal-snapshot` (judgment): For the same committed state and subscription context, the ordered reports must produce the complete working value that a fresh snapshot would describe. App verification must cover the snapshot boundary, concurrent changes, rejected requests, other mutation paths, changed subscription contexts, and restart/resubscription. The Hub routes reports without retaining their payloads or merging them.

### Default and explicit subscriptions

Owner: [protocols.md#default-and-explicit-subscriptions](../reference/protocols.md#default-and-explicit-subscriptions)

- `sub.default.first-nil`: If a protocol declares multiple SubscriptionRequest variants with effective Nil payloads, only the first one in Union declaration order is the automatic default subscription. Declaration order therefore matters.
- `sub.default.per-user-access` (judgment): Other subscriptions, including later Nil variants and every argument-bearing variant, require explicit invocation. The Hub does not retain default-subscription payloads or merge compatible reports. Each Control tracking admission or reconnect obtains a fresh initial snapshot from the App through the default SubscriptionRequest. The default may share one underlying stream, but each user is independently checked against that variant's `AccessPrivilege` before receiving current or subsequent data. A privileged user cannot authorize another user's subscription. Apps must not infer peer identity from how that stream is shared.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Compatible SubscriptionData

Owner: [protocols.md#compatible-subscriptiondata](../reference/protocols.md#compatible-subscriptiondata)

- `sub.compatible.declaration`: A SubscriptionData NextResponse names compatible SubscriptionData variants of the same protocol; compatibility is directional
- `sub.compatible.table-payloads`: Every compatible group member has a non-null Table payload
- `sub.compatible.studio-checks`: Studio checks the declaration, Table payloads and matching-field assignability

### Attribute declarations

Owner: [protocols.md#attribute-declarations](../reference/protocols.md#attribute-declarations)

- `sub.append.declaration`: AppendTo is declared on an incoming SubscriptionData root array field
- `sub.append.index`: AppendIndex is declared on the AppendTo array field and requires AppendTo
- `sub.append.non-nullable-arrays`: Append source and destination arrays are non-nullable, with assignable element types

### Studio validation and App responsibilities

Owner: [protocols.md#studio-validation-and-app-responsibilities](../reference/protocols.md#studio-validation-and-app-responsibilities)

- `sub.append.conflicting-writers`: Studio rejects two appends to one root array, or an append and a replacement or CopyTo write to it, in one report

### Reduced SubscriptionData design pattern

Owner: [protocols.md#reduced-subscriptiondata-design-pattern](../reference/protocols.md#reduced-subscriptiondata-design-pattern)

- `sub.reduced.stable-next-response`: Each stable variant NextResponse names the reduced variants it accepts

### Acceptance and destination are separate

Owner: [protocols.md#acceptance-and-destination-are-separate](../reference/protocols.md#acceptance-and-destination-are-separate)

- `sub.reduced.same-protocol`: Reduced and stable variants belong to the same protocol

<!-- /generated -->
