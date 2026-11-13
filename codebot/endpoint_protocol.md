# Declaring an Endpoint protocol

Load this card when a task declares or changes an Endpoint, its protocol Union,
variant roles, replies, errors, access, `OutputOnly` fields, the hidden
context a request is built from, or protocol conformance. Its rules are
generated from the [Endpoint protocols and runtime contract](../reference/protocols.md),
which owns them; path syntax and the copy rules belong to the
[Libertas Schema Path specification](../reference/schema.md#libertas-schema-path).
The [annotation inventory](schema_annotations.md) gives each annotation's Rust
and TypeScript spelling. Load the card for each feature the protocol uses:
[workflows](workflows.md), [actions and entry](actions.md), [links](links.md),
[subscriptions](subscriptions.md), [calculations](calculations.md) and
[AI Search](ai_search.md), and the [Endpoint callback card](endpoint_runtime.md)
for the App code that serves or calls the Endpoint.

## Order of work

1. Declare the Endpoint field and its selectors (`endpoint.*`), and keep one
   versionless protocol Union (`conformance.versionless-union`).
2. Give every message-bearing variant its roles (`protocol.roles.*`), mark
   errors and their locator (`protocol.error*`), and mark server-only output
   (`protocol.output-only.*`).
3. Mark each mutating Request `Write` (`access.*`) and decide which Requests
   are withheld from AI tools and Control UI (`aitool.*`,
   `endpoint.exclude-ui.*`).
4. Design the data each request is built from: the reference context
   (`protocol.context.*`, `protocol.copyfrom.*`), its hidden continuation
   fields (`protocol.hidden.*`), and the acknowledgements that keep a screen
   (`protocol.ack.*`).
5. Declare mandatory and optional capabilities (`conformance.*`), then
   compile with Studio and fix every reported rule.

<!-- generated:codebot-rules task=endpoint-protocol -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Declaring an Endpoint

Owner: [protocols.md#declaring-an-endpoint](../reference/protocols.md#declaring-an-endpoint)

- `endpoint.client-selectors`: On a client Endpoint, `Default` and `Fixed` take a `developer-package::function` task-process identity.
- `endpoint.client-selectors.compatibility-first` (judgment): Every client first filters server Endpoints by mutual protocol-schema compatibility. `Default` leaves all compatible choices available. `Fixed` further restricts choices to servers owned by the named task process and makes its package a mandatory logical App dependency in finalized `AppDependencies`. Both Endpoint-ID selectors remain editable because one process can expose multiple eligible server instances. If the resulting set is empty, either value identifies the process to provision. Only `Fixed` forces the package into deployed `AppDependencies`; the client task pre-creation gate below may nevertheless require installation of the named task process for an absolutely required Endpoint carrying either attribute. These strings are not Endpoint IDs or schema-type references.

### Required client task-process dependencies

Owner: [protocols.md#required-client-task-process-dependencies](../reference/protocols.md#required-client-task-process-dependencies)

- `endpoint.client-prerequisite` (judgment): An App author models only the source contract: a client Endpoint carrying `Default` or `Fixed` can become a task-creation prerequisite when that occurrence and its complete path are non-nullable and every ancestor List requires at least one item.
- `endpoint.client-prerequisite.truthful-bounds` (judgment): Keep the real optionality and List bounds truthful; do not add App code or generated metadata to implement provisioning.
- `endpoint.exclude-ui.server-projection.authoring`: Use ExcludeUI to remove an Endpoint from Control and Hub AI projection; [it does not remove registration, routing or access checks](../reference/protocols.md#required-client-task-process-dependencies).
- `endpoint.exclude-ui.still-registered` (judgment): The Endpoint remains registered and available to routing, subscriptions, authorization, and administration.

### Choosing a role

Owner: [protocols.md#choosing-a-role](../reference/protocols.md#choosing-a-role)

- `protocol.choice.list` (judgment): **Choosing the role for a need.** Each need has one role; the linked section owns its rules. Never substitute one role for another.
- `protocol.choice.one-shot` (judgment): **Do or find something once:** a `Request` whose `NextResponse` names its `Response` replies.
- `protocol.choice.offered-actions` (judgment): **Further actions the person may choose:** the Response's `NextRequest`; offered, opened only by the person or the [automatic singleton selection](../reference/protocols.md#default-request-entry-and-dynamic-actions).
- `protocol.choice.strict-question` (judgment): **Ask the person something now:** a Response declaring `WorkflowRequest`; a page the person chooses is an offered client-built request ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)).
- `protocol.choice.edit-existing` (judgment): **Edit data the service has stored:** `EditRequest` ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)).
- `protocol.choice.entry` (judgment): **Start the endpoint with one input-free request:** `DefaultRequest`; otherwise the default subscription or landing owns entry ([entry](../reference/protocols.md#default-request-entry-and-dynamic-actions)).
- `protocol.choice.live-data` (judgment): **Data that changes while watched:** `SubscriptionRequest` with `SubscriptionData`; the first Nil SubscriptionRequest is the default ([subscriptions](../reference/protocols.md#default-and-explicit-subscriptions), [compatible data](../reference/protocols.md#compatible-subscriptiondata)).
- `protocol.choice.calculation` (judgment): **Field values computed while editing:** a calculation Request that each changeable input field names in `CalculationRequest`; an [invariant input](../reference/protocols.md#invariant-calculation-inputs) needs none ([calculations](../reference/protocols.md#field-calculation-requests)).
- `protocol.choice.link` (judgment): **Tapping a row or text invokes a request:** `Link`; a Request offered only through links is `LinkOnly` ([links](../reference/protocols.md#links), [link-only](../reference/protocols.md#link-only)).
- `protocol.choice.confirmation` (judgment): **A critical request needs consent:** `RequestConfirmation`; the client asks Send or Cancel before a person's choice, a landing or a submission sends it ([critical requests](../reference/protocols.md#request-confirmation)).
- `protocol.choice.error` (judgment): **Refusal or failure:** a Response marked `Error`, handled by each client's standard error procedure ([error results](../reference/protocols.md#protocol-roles)).

### Protocol role reuse

Owner: [protocols.md#protocol-role-reuse](../reference/protocols.md#protocol-role-reuse)

- `protocol.roles.reuse` (judgment): A message-bearing Protocol Variant may declare more than one role. Its declared roles identify its permitted uses; the actual operation and interaction context determine the applicable behavior. Additional roles do not themselves initiate operations or combine their effects. Each declaration must satisfy the constraints applicable to its roles. Operation-selection rules still apply when choosing which declared use to invoke.

### Protocol roles

Owner: [protocols.md#protocol-roles](../reference/protocols.md#protocol-roles)

- `protocol.roles.operation-variant`: A correlated Response operation requires a `Response` variant, and subscription delivery requires a `SubscriptionData` variant.

### Interaction reference context

Owner: [protocols.md#interaction-reference-context](../reference/protocols.md#interaction-reference-context)

- `protocol.context.definition` (judgment): **Reference context and request construction.** The **current request** is the value being composed or edited. Workflow and non-workflow requests may need related server data; a session ID is required only when the App needs one. A **reference context** is the lookup environment of a request under construction. It belongs to the received value that the request is built from: a displayed Response, a delivered EditRequest, or a subscription. Its contained schema-described values are **reference objects**.

### Protocol roles

Owner: [protocols.md#protocol-roles](../reference/protocols.md#protocol-roles)

- `protocol.context.ordinary-response` (judgment):

  | Context | Reference objects and request construction |
  | --- | --- |
  | Ordinary Response | CopyFrom probes the latest non-transition acknowledgement received since the displayed Response was installed, when one exists, then the displayed Response. A transitioning Response replaces both. |

- `protocol.context.edit-request` (judgment):

  | Context | Reference objects and request construction |
  | --- | --- |
  | EditRequest | The received complete Request is the reference object. Adopt its data directly without CopyFrom or default initialization. |

- `protocol.context.subscription` (judgment):

  | Context | Reference objects and request construction |
  | --- | --- |
  | Subscription | The reply that completed the SubscriptionRequest is the first state of the SubscriptionData snapshot and is not kept as a separate object. CopyFrom probes the most recent successful correlated Response received after it, when one exists, then the current snapshot. Either object may be absent. |

- `protocol.context.replacement` (judgment): Subscription reports, including reduced-type updates, update the SubscriptionData snapshot without replacing the retained Response. In a subscription, a successful correlated Response replaces the Response object even when an acknowledgement leaves the screen unchanged. Over a displayed Response, a non-transition acknowledgement replaces only the acknowledgement object, and the displayed Response remains the second object; a transitioning Response replaces both. In every context, a failure status or Error-marked Response leaves the reference context unchanged.
- `protocol.context.separate-objects` (judgment): The two objects of a context remain separate and are not merged into a synthetic value. The context of another displayed value, editor, or subscription, and historical responses, are never searched. With no eligible received data, the reference context has no objects.
- `protocol.copyfrom.reference-context`: `CopyFrom` names source paths on a receiving request node. It copies already-received data from the reference context without contacting the server.
- `protocol.copyfrom.probe-order` (judgment): For each mapping, probe objects in context order and paths within each object in declared order under the [shared copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules). Copied nodes override initialization; unaffected nodes retain ordinary initialization, including defaults or undefined. Direct EditRequest adoption preserves all received values and supplies a new context containing that Request.
- `protocol.context.carry-forward` (judgment): An App must carry forward required continuation data; missing values are not recovered from older responses.

### Hidden protocol context

Owner: [protocols.md#hidden-protocol-context](../reference/protocols.md#hidden-protocol-context)

- `protocol.hidden.context-fields` (judgment): **Hidden protocol context.** Session identifiers, continuation cookies, record identifiers/revisions used only for application coordination, and application data carried only to initialize a later request must be schema-defined `Hidden` fields in the protocol payload. Apply Hidden to each such field or to its containing cookie/context field.
- `protocol.hidden.visible-sources` (judgment): A visible user-facing value may also be a copy source; do not hide that value merely because another request copies it.
- `protocol.hidden.copy-into-requests` (judgment): The server supplies required context in the Response, complete EditRequest, or SubscriptionData snapshot from which the next action is constructed. Declare CopyFrom on receiving request nodes that need those values. Keep session/coordination fields Hidden in the request too; a field initialized for the user to review or edit remains visible. Hidden on the source is presentation metadata, not a command to hide the destination of a value copy.
- `protocol.hidden.typed-payload` (judgment): Hidden data remains typed payload data: it participates in reference reads, copying, validation, subscription updates, and encoding. Hiding a field does not initialize it or exempt it from completeness checks. Follow the [Hidden presentation rule](../reference/presentation.md#common-field-metadata).
- `protocol.hidden.every-construction-path` (judgment): Supply required data on every permitted construction path, including initial entry, subsequent actions, workflow navigation, and resubscription. A complete EditRequest already supplies its hidden values; clients must not initialize over them. For copied requests, copied nodes override initialization and unaffected nodes follow the shared initialization rules. Never rely on a user to fill a hidden missing value or invent a placeholder session ID.
- `protocol.hidden.continuation-replacement` (judgment): Design continuation data against the reference-context replacement rules above. Carry it forward when its supplying object is replaced; an ordinary success acknowledgement can replace the retained Response of a subscription, or the previous acknowledgement over a displayed Response, while the screen stays unchanged. SubscriptionData or the displayed Response remains the second reference object, but an older Response or acknowledgement is not a fallback. Error does not replace either object.
- `protocol.hidden.reduced-reports` (judgment): Reduced reports preserve or update hidden fields under the same projection rules as visible fields.
- `protocol.hidden.first-time-entry` (judgment): First-time entry has no prior reference objects. Entry requests must be constructible from their own valid inputs and normal initialization; the server then supplies any continuation context needed for later actions. Handle absent external validation reads under the [validation contract](../reference/validation-rules.md).
- `protocol.hidden.no-copy-for-validation` (judgment): Do not copy a value into the request solely to validate against it when a reference-context read suffices.
- `protocol.hidden.not-authorization` (judgment): Hidden controls presentation, not confidentiality or authorization. These bytes reach the client. The server must validate submitted identifiers, revisions, session ownership, and allowed operations; client-side invisibility is not evidence of trust.
- `protocol.hidden.session-id-optional` (judgment): A session identifier is needed only when the App's interaction requires one.

### Protocol roles

Owner: [protocols.md#protocol-roles](../reference/protocols.md#protocol-roles)

- `protocol.context.validation-reads` (judgment): `ValidationRules`, `RangeMin`, and `RangeMax` use the same reference context for external reads, retaining their own path syntax and first-non-null rules. Current-data reads inspect the request being edited. No matching value or an empty context yields null, which first-time composition must handle. Reading neither copies values nor fetches server data.
- `protocol.error.no-payload-shape` (judgment): It does not define an error payload shape or change serialization, routing, transitions, or compatibility.
- `protocol.error.first-formatted-text` (judgment): Therefore put the most useful user-facing explanation in the first `FormattedText` field; the payload may still contain any typed diagnostic or retry context required by App code.
- `protocol.error-field-path.value` (judgment): Null means the refusal names no field. A non-null value is a square-form [Libertas Schema Path](../reference/schema.md#libertas-schema-path) rooted at the rejected Request value, the payload of the variant the client sent. For example, `[incomes/2/box12/0/amount]` selects one amount in nested Lists. It is a [locator path](../reference/schema.md#locator-paths): List members are named by position in that sent request or, at a keyed List, by a key step such as `[incomes/="W2-58"/box12/0/amount]`.
- `protocol.error-field-path.empty-is-null` (judgment): An empty string is treated as null; Apps should send null instead.
- `protocol.error-field-path.first-failing` (judgment): A refusal names one field, the first failing one.
- `protocol.error-field-path.not-output-only` (judgment): An App must not return a locator that resolves into an [OutputOnly](../reference/protocols.md#output-only-request-fields) node, a value it never receives; doing so is an App defect. Clients treat it as no locator and show only the dialog, and Studio also reports it in its developer diagnostics.

### In-page reveal paths

Owner: [protocols.md#in-page-reveal-paths](../reference/protocols.md#in-page-reveal-paths)

- `protocol.reveal-path.value` (judgment): Null or the empty string names no field. Any other value is a square-form [Libertas Schema Path](../reference/schema.md#libertas-schema-path): a [locator path](../reference/schema.md#locator-paths), formed exactly as an [error field path](../reference/protocols.md#error-field-path) is. A row's path is rooted at the Request value of the editor that shows the row; a Request's path is rooted at the value of the page its reply lands. Prefer key steps at keyed Lists, so the path still names the same member after the person edits the List.
- `protocol.reveal-path.copy`: A RevealPath field is an ordinary Nullable String for [CopyFrom](../reference/schema.md#shared-copyfrom-and-copyto-rules): it may receive its value by `CopyFrom`, and it may be the source of another field's `CopyFrom`. Its ReadOnly placement does not prevent either. The copied string is used unchanged; neither copy direction rewrites, re-roots or validates the path.
- `protocol.reveal-path.from-row` (judgment): A row whose reveal path names a field on another page usually gives its Link Request's reveal path an ordinary `CopyFrom` of the row's own: the row reveals in place when its path resolves in the open editor, and otherwise the Link sends it, so the page it opens lands on the field.

### Request reveal paths

Owner: [protocols.md#request-reveal-paths](../reference/protocols.md#request-reveal-paths)

- `protocol.reveal-path.request`: A Request's reveal path names where the page its reply lands opens, as a URL fragment (`#anchor`) names where a browser scrolls once a page loads: the client applies it to whatever page lands, without comparing pages. It is an ordinary field of the Request, so the Hub receives it and may read it. When the client sends the Request, by Submit, Prev or a [Link](../reference/protocols.md#links), the operation that owns the reply remembers the field's value; a Request without the field, or with null or an empty value, remembers none.

### Landing paths

Owner: [protocols.md#landing-paths](../reference/protocols.md#landing-paths)

- `protocol.landing-path.value`: A landing path is set only by the server, in the page value it returns; the client never composes, copies or edits it, so a page the client composes has none. The empty string names no node. Any other value is a [locator path](../reference/schema.md#locator-paths) rooted at that page's own value, preferring key steps at keyed Lists. It applies when its page [lands](../reference/protocols.md#page-landing), after a remembered reveal path that resolves.
- `protocol.output-only.no-client-validation` (judgment): Clients run no client validation on an OutputOnly node or anything inside it, incremental or final: neither its own constraints (size, Pattern, ranges, required or undefined) nor the `ValidationRules` hosted on it. Rules hosted elsewhere that read it evaluate normally, against its displayed value.
- `protocol.output-only.sent-empty` (judgment): Submitting the Request that carries the node sends its empty value, not its Default: null when it is Nullable, otherwise `""`, empty bytes or `[]`. That is the only change. Display, EditRequest, CopyTo and Default filling are unchanged.
- `protocol.output-only.copyfrom-displayed` (judgment): A Prev action, a link or a calculation request is a different request built by [CopyFrom](../reference/schema.md#shared-copyfrom-and-copyto-rules): a CopyFrom that reads an OutputOnly node copies its current displayed value under the ordinary copy rules, and only that request's own OutputOnly nodes, if any, are sent empty.
- `protocol.output-only.save-path` (judgment): The App's save path never reads an OutputOnly node, and it must not name one, or a node inside one, in an [ErrorFieldPath](../reference/protocols.md#error-field-path): that is an App defect, handled there.
- `protocol.ack.optional-transition` (judgment): Outside a workflow, a normal `Response` is not required to declare `NextRequest` or `WorkflowRequest`. Omitting both is an intentional non-transition and is common for shared acknowledgement variants such as `Success` or `Failure`. It does not mean that every Request in the protocol becomes available.
- `protocol.error.regardless-of-next-request` (judgment): An Error-marked correlated Response uses the separate standard error procedure above, regardless of `NextRequest`.

### Request access

Owner: [protocols.md#request-access](../reference/protocols.md#request-access)

- `access.privilege.default` (judgment): Omission also means `Read,Write`. A user with administrator privileges grants a user endpoint read or write access, and the Hub keeps that grant in its access control list. Libertas cannot infer semantic mutation.
- `access.privilege.mark-writes` (judgment): The developer must mark a mutating operation `Write`; otherwise a read-only user is allowed to issue it.
- `access.privilege.hub-authoritative` (judgment): Clients may hide a disallowed action, but the Hub remains authoritative.
- `access.privilege.every-invocation` (judgment): The Hub checks native client requests and AI tool invocations against the authenticated user's current endpoint read/write grants and the submitted Request's `AccessPrivilege`. Every applicable Request variant selected in a nested payload must independently accept at least one of those grants. Listing an action in an assistant's tool catalog confers no authority: each invocation is checked again, including after a permission change. A tool offered for a superseded task deployment cannot invoke its replacement deployment.
- `access.privilege.not-permissions` (judgment): This is unrelated to task-process `Permissions`, which grant access to platform resources during task configuration.

### AI tool calls

Owner: [protocols.md#ai-tool-calls](../reference/protocols.md#ai-tool-calls)

- `aitool.requests`: Whatever a person can do with an App through a client, an AI assistant can do the same way, when the App's developer and the person let it and it is capable of the task. The developer writes nothing to enable this, and can only withhold a request: `LLMToolCall` `false` or `ExcludeUI`, below. The person lets it through the access they grant and the confirmations they give. Every `Request` variant of an Endpoint protocol is automatically an AI tool: the Hub generates it for each user, from the same declaration and comments that clients use to draw the request editor, and only from the server Endpoints that user can reach.
- `aitool.same-request` (judgment): A tool call is one more way to send the request; the App receives the same Request value, in the same encoding, as from a client or another program, and cannot tell them apart.
- `aitool.subscription-only` (judgment): A `SubscriptionRequest`-only variant is not a tool.
- `aitool.authority` (judgment): **Authority and validation.** [Request access](../reference/protocols.md#request-access) is checked on every call, and the service validates the request as it validates any other. Results are shown as described in [AI tool-call output](../reference/presentation.md#ai-tool-call-output).

### Request and response ordering

Owner: [protocols.md#request-and-response-ordering](../reference/protocols.md#request-and-response-ordering)

- `ordering.human-task-scope.authoring`: Rely on platform-owned [human-task ordering](../reference/protocols.md#human-task-interaction); software peers may agree other ordering within the declared protocol roles and access checks.
- `ordering.subscription-updates-separate` (judgment): Unsolicited SubscriptionData/DataReport updates are not correlated reply slots; they retain their own subscription ownership and sequencing rules.

### Reusing a variant for reading and editing

Owner: [protocols.md#reusing-a-variant-for-reading-and-editing](../reference/protocols.md#reusing-a-variant-for-reading-and-editing)

- `protocol.reuse.read-and-edit` (judgment): A variant may carry both `Request` and `Response`. Add `NextRequest` naming that variant to offer editing after receiving it, and `CopyFrom("$")` on the request payload to initialize the complete editor from the [current reference context](../reference/protocols.md#interaction-reference-context). `NextResponse` may name the same variant for the server's updated result. Root copying copies nested values into an independent editing value; changing the editor does not mutate the received response.
- `protocol.reuse.cycle` (judgment): This supports a receive, edit, submit, and receive cycle using the same data shape. It does not automatically turn response display into an inline editor. The existing [protocol navigation rules](../reference/endpoint-presentation.md#protocol-transaction-stack) control presentation under the [shared role rule](../reference/protocols.md#protocol-role-reuse). The [shared copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules) determine initialization when no reference context is available.

### Strict questions and data pages

Owner: [protocols.md#strict-questions-and-data-pages](../reference/protocols.md#strict-questions-and-data-pages)

- `workflow.entry.undefined-not-encoded` (judgment): **EditRequest or WorkflowRequest.** A node is [undefined until it is initialized](../reference/schema.md#undefined-and-its-sources), and undefined cannot be encoded, so no message carries an unanswered field. An editor opened by either one carries the platform's two workflow buttons, **Go Back** (when the Request declares `PrevRequest`) and **Next/Submit**, whose captions a Display Template may customize through [`button_labels`](../reference/presentation.md#button-labels); the App adds no buttons of its own for them ([submit and previous actions](../reference/protocols.md#submit-previous-actions-and-completion)). What the page needs decides how it opens:
- `workflow.edit-request.why` (judgment): **Nothing would start undefined: EditRequest.** It carries a Request the service has stored, or an unsaved page's value equal to its ordinary initialization; the client adopts it as it is. Its Nullables are true optionals whose null is a real answer with the meaning its `NullText` names, such as "None" or "Does not apply".
- `workflow.client-built.why` (judgment): **Fields left undefined and the person chooses the page: a client-built request.** The client constructs it from an offered action: a `NextRequest` action, a link (available while its target is in that action set), or a landing request. Like a WorkflowRequest editor, it commonly copies known facts by `CopyFrom` from the displayed Response, and every field nothing initializes stays undefined. An offered request opens only when the person chooses it, or by the [automatic singleton selection](../reference/protocols.md#response-action-constraints). Who opens the page, not copying, separates it from WorkflowRequest.
- `workflow.workflow-request.why` (judgment): **Fields left undefined and the service opens the page at once: WorkflowRequest**, with or without copied facts, and only then. The service sends a Response; the client constructs the named Request, copies any known facts by `CopyFrom`, and leaves the rest undefined. An unanswered source path is absent, never null: CopyFrom stops at a valid null, which a non-Nullable field cannot take.

### Operation ownership

Owner: [protocols.md#operation-ownership](../reference/protocols.md#operation-ownership)

- `ops.no-retraction` (judgment): Leaving the UI cannot retract a submission already received by the backend. Persistence, retry idempotency, and resume selection belong to the application; the client never serializes an incomplete draft to supply them.
- `exit.app-admits-entry` (judgment): Keeping the exchange consistent for a person across leaving and entering again is the App developer's responsibility: the service admits the entry request in every state in which the endpoint can be left.
- `exit.waiting-initiator` (judgment): **Waiting is the initiator's decision.** The protocol defines no timeout. The initiator of a Request decides how long to wait for its reply: a program or an AI participant applies its own limit, and a person dismisses when they choose to. Giving up is leaving the endpoint as above; it never produces a reply of its own and never moves the exchange to another state. A client that initiates auxiliary requests itself, such as field calculations or a document transformation, is their initiator and may bound them; an expired auxiliary request is a failed one and changes no data.

### Chaining, caching, and copied values

Owner: [protocols.md#chaining-caching-and-copied-values](../reference/protocols.md#chaining-caching-and-copied-values)

- `protocol.next-request.absence-not-all` (judgment): With neither NextRequest nor WorkflowRequest outside a workflow, the response is the non-transition acknowledgement defined under [Protocol roles](../reference/protocols.md#protocol-roles); absence is never expanded to every Request in the protocol Union.
- `protocol.copyfrom.editor-keeps-context` (judgment): Copying from the current editor does not replace the interaction's [reference context](../reference/protocols.md#interaction-reference-context).

### Protocols describe capabilities

Owner: [protocols.md#protocols-describe-capabilities](../reference/protocols.md#protocols-describe-capabilities)

- `conformance.prefix-capability` (judgment): A protocol path prefix identifies a payload capability through its enclosing protocol Union variants. It is not a path to every field inside that payload. Once the capability is selected, ordinary serialization/deserialization handles the payload under its compatible schema. Capability correspondence and encoding are separate concerns; finding a matching prefix does not make incompatible payload definitions compatible.

### Capability prefixes across nesting depths

Owner: [protocols.md#capability-prefixes-across-nesting-depths](../reference/protocols.md#capability-prefixes-across-nesting-depths)

- `conformance.server-discriminants` (judgment): Integers in a payload that name protocol choices, a response's [allowed-action list](../reference/protocols.md#response-action-constraints) and [named action-control](../reference/protocols.md#named-action-controls) lists, always name discriminants of the server schema, the wire representation. The translator copies them unchanged with the payload, and a client App reads them against the server's schema, never its own arrangement.

### Required and optional capabilities

Owner: [protocols.md#required-and-optional-capabilities](../reference/protocols.md#required-and-optional-capabilities)

- `conformance.versionless-union` (judgment): Protocol declaration and optional runtime support solve different problems. The protocol Union itself has one versionless, lifetime-invariant name. New App versions append new protocol payload variants to that same Union; they do not replace it with a `V2` Union. The complete evolution contract is in the [App data-type design guidelines](../reference/data-type-evolution.md).
- `conformance.protocol-conformance`: `ProtocolConformance` may be declared on a protocol Union on either client or server. During task configuration, the client expression is evaluated against the server declaration and the server expression is evaluated against the client declaration; both must pass. Operators are:
- `conformance.default-all-mandatory` (judgment): Without an expression, every direct variant in that Union context is mandatory.
- `conformance.pass-through-payloads` (judgment): Matching variants must also have pass-through-compatible payloads. Libertas translates only the [capability prefix](../reference/protocols.md#capability-prefixes); inside a payload, positional fields and overlapping nested Union slots must remain aligned, and nested optional variants may only extend the tail of a smaller Union. Payload bytes are not decoded by the translator; the receiving server or client App decoder validates them against its own schema.
- `protocol.examples.complete-authored-declarations`: The following focused snippets are non-copyable schematics: the referenced variants and their required natural-language documentation are omitted so the placement of `ProtocolConformance` remains visible. A generated App must use the complete declaration style shown in the preceding role example.
- `protocol.no-caching-attribute` (judgment): No attribute enables protocol Request/Response or subscription-payload caching. `CacheIdentity` concerns local editor drafts only.

### Function and Endpoint permissions

Owner: [protocols.md#function-and-endpoint-permissions](../reference/protocols.md#function-and-endpoint-permissions)

- `perm.endpoint-downstream`: A server Endpoint may declare downstream permissions without same-key reason resources; the selected client task must have those permissions when the relationship is configured.
- `perm.not-access-privilege` (judgment): Never substitute `AccessPrivilege` for a process permission or vice versa.

### 4.5 Union, endpoint, and data attributes

Owner: [schema.md#45-union-endpoint-and-data-attributes](../reference/schema.md#45-union-endpoint-and-data-attributes)

- `protocol.llm-tool-call.runtime`:

  | Attribute | Meaning |
  | --- | --- |
  | <!-- rule: protocol.llm-tool-call class=hard enforced=yes tasks=endpoint-protocol title="`LLMToolCall` must have a Boolean value" --> `LLMToolCall` | Boolean switch. For the automatic [AI tool](../reference/protocols.md#ai-tool-calls), it defaults to `true`. On a protocol Union, `false` excludes descendant Request variants from Hub LLM tool generation; a variant's explicit `true` overrides the Union. The nearest declaration or occurrence on the path wins. `false` on one Request variant excludes only that variant. It does not remove an Endpoint or Request from Control, alter wire data, or bypass `ExcludeUI` or access checks. |

### Dynamic schema properties

Owner: [schema.md#dynamic-schema-properties](../reference/schema.md#dynamic-schema-properties)

- `dynamic.schema.permission`: Omission adds no restriction. Only Boolean `true` permits an operation, subject to all existing static constraints and enclosing restrictions. False, null, unavailable data, and evaluation failure grant nothing. Wrong types are errors, never truthy values. Static ReadOnly, Fixed, Hidden, collection limits, and ordinary validation remain authoritative. An App's save path still checks the submitted operation and current business state; editor permissions do not prove the history of a submitted value.
- `dynamic.schema.scope`: `Editable=false` on a record prohibits user mutations throughout its subtree, including membership changes to contained Lists. It does not prevent removing that record from its parent List. On a List, `Editable` protects existing member values; addition and removal use `Addable` and member `Removable` independently. An enclosing record restriction can prohibit all three. Thus a List can permit additions while protecting existing values, or permit dismissal of server-supplied members while prohibiting additions.
- `dynamic.schema.add`: New-member composition uses normal initialization and the normal insertion lifecycle. Recheck `Addable` when insertion commits. A new member obeys its own restrictions; a List's protection of existing members does not prohibit normal construction of a new one. Apps supply a constructible member through ordinary initialization, composition, or server-supplied values. Addition cannot bypass restrictions on candidate fields.
- `dynamic.schema.preserve`: Permission changes do not clear values, remove members, suppress validation, or change encoding. Unchanged values remain submittable under normal validation. Platform initialization, CopyFrom, calculation CopyTo, and server replies keep their existing authority. Dynamic non-editability is not static ReadOnly and never makes a calculation input invariant. Calculation availability follows [dynamic permissions and calculations](../reference/protocols.md#dynamic-permissions-and-calculations).

### 6.1 Endpoint protocol validation

Owner: [schema.md#61-endpoint-protocol-validation](../reference/schema.md#61-endpoint-protocol-validation)

- `protocol.conformance-expression.runtime`: On a Union-typed variant, the variant's attribute becomes the conformance expression for that nested Union in that declaration context and takes precedence over an attribute on the referenced Union declaration. The expression uses the direct variant `NativeName` operands of the Union being evaluated with `!` (NOT), `&` (AND), `^` (XOR), `|` (OR), and parentheses. Operator precedence, from highest to lowest, is `!`, `&`, `^`, then `|`.

### 7. Path-based object and choice binding

Owner: [schema.md#7-path-based-object-and-choice-binding](../reference/schema.md#7-path-based-object-and-choice-binding)

- `baseobjects.derived-name-icon`: For an endpoint carrying `BaseObjects`, the Hub derives and caches its displayed name and icon from the resolved base objects, in their declared order:

### Dynamic property expressions

Owner: [validation-rules.md#dynamic-property-expressions](../reference/validation-rules.md#dynamic-property-expressions)

- `dynamic.expression.failure.authoring`: Account for [unsettled calculation inputs](../reference/validation-rules.md#dynamic-property-expressions) withholding Dynamic permission until the expression resolves.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Declaring an Endpoint

Owner: [protocols.md#declaring-an-endpoint](../reference/protocols.md#declaring-an-endpoint)

- `endpoint.protocol-union`: An Endpoint field references a protocol Union; EndpointServer marks the serving side

### Required client task-process dependencies

Owner: [protocols.md#required-client-task-process-dependencies](../reference/protocols.md#required-client-task-process-dependencies)

- `endpoint.exclude-ui.server`: ExcludeUI on a server Endpoint requires a resolved ProtocolSchema
- `endpoint.exclude-ui.operation`: ExcludeUI on an operation requires a Request or SubscriptionRequest reachable from an Endpoint protocol Union
- `endpoint.exclude-ui.same-protocol-root`: Server Endpoints that resolve to the same protocol root use the same ExcludeUI value

### Protocol roles

Owner: [protocols.md#protocol-roles](../reference/protocols.md#protocol-roles)

- `protocol.error.marker`: An error result is a Response marked with the no-argument Error marker

### Error field path

Owner: [protocols.md#error-field-path](../reference/protocols.md#error-field-path)

- `protocol.error-field-path.declaration`: At most one Nullable String ErrorFieldPath field, and only in an Error-marked Response

### In-page reveal paths

Owner: [protocols.md#in-page-reveal-paths](../reference/protocols.md#in-page-reveal-paths)

- `protocol.reveal-path.declaration`: At most one Nullable String RevealPath field per type, on a row shown read-only or directly on a Request payload

### Landing paths

Owner: [protocols.md#landing-paths](../reference/protocols.md#landing-paths)

- `protocol.landing-path.declaration`: At most one Hidden String LandingPath field directly on a page's value, with no CopyFrom

### Output only request fields

Owner: [protocols.md#output-only-request-fields](../reference/protocols.md#output-only-request-fields)

- `protocol.output-only.declaration`: OutputOnly goes only inside a Request, on a statically ReadOnly node with an empty value
- `protocol.output-only.no-size-min`: An OutputOnly node cannot have SizeMin greater than 0

### Request access

Owner: [protocols.md#request-access](../reference/protocols.md#request-access)

- `access.privilege.declaration`: AccessPrivilege on Request or SubscriptionRequest is an alternative-access bitmap: Read=1, Write=2, either=3

### Chaining, caching, and copied values

Owner: [protocols.md#chaining-caching-and-copied-values](../reference/protocols.md#chaining-caching-and-copied-values)

- `protocol.names.native`: NextRequest, NextResponse and CopyFrom variant references use native variant names
- `protocol.next-request.non-empty`: NextRequest is optional on a Response; when present it is non-empty

### Capability prefixes across nesting depths

Owner: [protocols.md#capability-prefixes-across-nesting-depths](../reference/protocols.md#capability-prefixes-across-nesting-depths)

- `conformance.prefix.role-less-variants`: A role-less prefix variant is a Union or holds exactly one Union field; a role-less cycle is invalid
- `conformance.unique-correspondence`: Each endpoint protocol exposes a capability identity at most once; duplicates reject the match

### Function and Endpoint permissions

Owner: [protocols.md#function-and-endpoint-permissions](../reference/protocols.md#function-and-endpoint-permissions)

- `perm.function-reason`: A function permission constant needs a same-key nonempty reason in the package StringResources

### 4.5 Union, endpoint, and data attributes

Owner: [schema.md#45-union-endpoint-and-data-attributes](../reference/schema.md#45-union-endpoint-and-data-attributes)

- `protocol.llm-tool-call`: `LLMToolCall` must have a Boolean value

### 6.1 Endpoint protocol validation

Owner: [schema.md#61-endpoint-protocol-validation](../reference/schema.md#61-endpoint-protocol-validation)

- `protocol.conformance-expression`: `ProtocolConformance` is nonempty and goes on a Union or Union-typed variant

### 7. Path-based object and choice binding

Owner: [schema.md#7-path-based-object-and-choice-binding](../reference/schema.md#7-path-based-object-and-choice-binding)

- `baseobjects.placement`: `BaseObjects` goes only on a built-in Endpoint; its paths resolve to set values unique per endpoint instance

<!-- /generated -->
