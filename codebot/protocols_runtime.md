# Endpoint protocols and runtime

This page routes Endpoint and persistence work to its task cards. The
[Endpoint protocols and runtime contract](../reference/protocols.md) owns every
rule; each card quotes the rules of one task, generated from that contract.
Path syntax and context belong to the
[Libertas Schema Path specification](../reference/schema.md#libertas-schema-path);
exact callback and function signatures belong to the installed public SDK.

## Choose the task cards

Load every card the task touches:

| The task changes | Card |
| --- | --- |
| An Endpoint, its protocol Union, roles, errors, access, `OutputOnly` fields, the hidden context requests are built from, AI tool exposure or protocol conformance | [Endpoint protocol](endpoint_protocol.md) |
| Endpoint entry, the Actions surface, input presentation, `RequestConfirmation`, `ConstrainedBy`, named action controls or native Back | [Actions, entry and choices](actions.md) |
| `Link`, link returns, stacked links or `LinkOnly` | [Links](links.md) |
| `EditRequest`, `WorkflowRequest`, `PrevRequest`, workflow completion or a changed earlier page | [Workflows](workflows.md) |
| `SubscriptionRequest` or `SubscriptionData`, compatible or reduced reports, append or live charts | [Subscriptions](subscriptions.md) |
| Field calculations or a calculated allowed set | [Calculations](calculations.md) |
| AI Search: `GoTo`, `AINavigationPrompt` and searchable destinations | [AI Search](ai_search.md) |
| The App code that serves or calls an Endpoint | [Endpoint callback code](endpoint_runtime.md) |
| A storage schema, keys, the key-value API or data-failure handling | [Persistent data](data_api.md) |
| Payload types, attributes, defaults, Schema Paths and ordinary `CopyFrom`/`CopyTo` | [Data model](data_model.md) |

## The four relations

Every runtime rule follows from four relations. Use them to find the card
that owns a behavior.

### 1. Reachability: a Request is reached only through a declaration that names it

| Declaration | Reached by | Card |
| --- | --- | --- |
| `NextRequest` on the displayed Response or SubscriptionData | the **Actions** surface | [Actions](actions.md) |
| [`ActionControl`](../reference/protocols.md#named-action-controls) array on that page | a named control | [Actions](actions.md) |
| `Link` on a displayed value's field or type | a tap on that value's row | [Links](links.md) |
| `Link` on a request-editor field | the row's link button | [Links](links.md) |
| `CalculationRequest` on an editor field | a change of that input | [Calculations](calculations.md) |
| `DefaultRequest` | endpoint entry | [Actions](actions.md) |
| `EditRequest` / `WorkflowRequest` reply | the server opening that editor | [Workflows](workflows.md) |
| `PrevRequest` on the edited Request | the workflow Back button | [Workflows](workflows.md) |
| none of the above | the request-list landing, only without a default entry | [Actions](actions.md) |

Nothing else reaches a Request: the [actions](actions.md) and
[links](links.md) cards hold the rules that infer nothing from `CopyFrom`, an
input-free payload or a missing `NextRequest`.

### 2. Replies: replace the working screen or are consumed in place

| Reply | Card |
| --- | --- |
| Response with `NextRequest`, plain acknowledgement, Error and its field locator | [Endpoint protocol](endpoint_protocol.md) |
| `EditRequest`, `WorkflowRequest`, and the end of a workflow | [Workflows](workflows.md) |
| plain Response to a link from an editor (a link return) or a stacked page | [Links](links.md) |
| calculation reply | [Calculations](calculations.md) |
| SubscriptionData report | [Subscriptions](subscriptions.md) |

### 3. Copies: a target is built from the reference context

`CopyFrom` follows the
[shared copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)
and the [context table](../reference/schema.md#context-and-syntax), quoted on
the [data model card](data_model.md); the
[reference context](../reference/protocols.md#interaction-reference-context)
and its hidden continuation data are on the
[Endpoint protocol card](endpoint_protocol.md), a link's construction on the
[links card](links.md), and a previous action's on the
[workflow card](workflows.md).

### 4. Screens: landing entry, one working page, one composer

Landing kinds, endpoint entry and native Back are on the
[actions card](actions.md); leaving an endpoint and waiting for a reply are on
the [Endpoint protocol card](endpoint_protocol.md); the screen stack is owned by
[endpoint presentation](../reference/endpoint-presentation.md#protocol-transaction-stack),
and draft recovery by [CacheIdentity](../reference/annotations.md#cacheidentity).

## Persistent data schemas

Storage schemas, keys, the key-value API and data-failure handling are on the
[persistent data card](data_api.md); record names follow
[data and database names](../reference/protocols.md#data-and-database-names).
