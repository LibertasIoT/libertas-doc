# Offering actions, entry and choices

Load this card when a task decides what a person can do on a page: endpoint
entry (`DefaultRequest`, landing forwarding), the Actions surface and its
input presentation, `RequestConfirmation`, response-level and field-level
`ConstrainedBy`, named action controls, or what native Back does. Its rules
are generated from the
[Endpoint protocols and runtime contract](../reference/protocols.md#default-request-entry-and-dynamic-actions),
which owns them. Rows that open a request are [links](links.md); a calculated
allowed set is on the [calculation card](calculations.md).

## Order of work

1. Decide endpoint entry and its landing (`entry.*`, `landing.*`).
2. Declare each page's `NextRequest` set and, when the service decides its
   membership, the response `ConstrainedBy` (`actions.response-constraint.*`,
   `actions.auto-selection.authoring`) and named controls (`actions.control.*`).
3. Check how each offered Request is presented by its input
   (`actions.tap.*`, `actions.input.*`, `actions.hierarchical.*`,
   `actions.editor.*`) and mark the critical ones
   (`actions.confirmation.*`).
4. Constrain enumerated and Union request fields from service data
   (`choices.*`), then compile with Studio and fix every reported rule.

<!-- generated:codebot-rules task=actions -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Choosing a role

Owner: [protocols.md#choosing-a-role](../reference/protocols.md#choosing-a-role)

- `protocol.choice.list` (judgment): **Choosing the role for a need.** Each need has one role; the linked section owns its rules. Never substitute one role for another.
- `protocol.choice.offered-actions` (judgment): **Further actions the person may choose:** the Response's `NextRequest`; offered, opened only by the person or the [automatic singleton selection](../reference/protocols.md#default-request-entry-and-dynamic-actions).
- `protocol.choice.entry` (judgment): **Start the endpoint with one input-free request:** `DefaultRequest`; otherwise the default subscription or landing owns entry ([entry](../reference/protocols.md#default-request-entry-and-dynamic-actions)).
- `protocol.choice.confirmation` (judgment): **A critical request needs consent:** `RequestConfirmation`; the client asks Send or Cancel before a person's choice, a landing or a submission sends it ([critical requests](../reference/protocols.md#request-confirmation)).

### Backend-constrained choices

Owner: [protocols.md#backend-constrained-choices](../reference/protocols.md#backend-constrained-choices)

- `choices.constrained-by.own-source`: Each occurrence of the constrained field is filtered only by the source its own path resolves to.
- `choices.source.service-written`: The service writes it and the user never does: populate it from backend response data through CopyFrom, including copying its containing cookie, supply it directly in EditRequest, or assign it from a field calculation reply or a [link return](../reference/protocols.md#link-return) through CopyTo under the [calculated allowed sets](../reference/protocols.md#calculated-allowed-sets) rules. Between those writes the set is stable; ordinary editing never replaces the source or its containing data.
- `choices.entries.values`: For an Enumeration, entries are its declared numeric values. For a Union, entries are its original schema discriminants, never positions in the filtered chooser. `[1]` permits only choice 1; `[0, 2]` permits choices 0 and 2.
- `choices.entries.keyed-enumsource`: For a keyed EnumSource host, entries are key values: the chooser offers only the source List's members whose key is in the set, together with the existing `Unique` and empty-key exclusions. Eligibility never removes a member from the source: a stored key keeps displaying its member's summary from the whole List, in choosers, summaries and read-only rows. A stored key outside the set is the same field error as an Enumeration value outside it. The singleton rule and `when_single_choice` treat one eligible member as one answer, exactly as for a declared choice.
- `choices.empty-and-null`: An absent or null source offers no choice and blocks submission with the field error, also for a Nullable field. An empty source offers no declared choice: it blocks submission for a non-nullable field, while for a [Nullable](../reference/presentation.md#nullable-choices) field it leaves exactly one answer, null, which is how a service says the question does not apply. Duplicate entries do not create extra choices. Null is not a numeric choice; ordinary Nullable semantics apply when the allowed set is nonempty. Undefined remains unanswered and cannot be submitted for a non-nullable field.
- `choices.consistent-prefill` (judgment): Clients show only permitted choices and validate the selection before sending. A selected Union branch retains its ordinary editable payload and nested-field validation. A singleton allowed set does not construct an unfinished payload on the server or automatically submit the editor. Missing or malformed cookie data, an empty allowed set, or a prefilled choice outside the set blocks submission with a field error. Clients do not silently replace a conflicting prefill; the application must supply consistent values and cookies.
- `choices.singleton`: **Singleton allowed set.** When a non-nullable field is undefined and its current allowed set has exactly one entry, the client selects that choice through ordinary initialization, exactly as a Union [`Default`](../reference/schema.md#43-value-constraint-and-presentation-attributes) selects its variant. The field is not locked: a Union branch's payload stays editable, submission still validates membership, and nothing is submitted automatically. A nullable field with a nonempty allowed set is not selected, because null is a second answer; a nullable field whose allowed set is empty, and which is unanswered, is answered with null the same way, since null is then its only answer. A value it holds outside an empty set stays a field error that the person resolves by choosing the null answer. When a [calculated allowed set](../reference/protocols.md#calculated-allowed-sets) shrinks to one entry while the field is undefined, the same selection is part of that atomic editor change; if the set later widens, the selection stays as an ordinary one. This is a data rule, owned here and not by presentation: it writes the value whether or not the field's row is drawn, and a Display Template [`when_single_choice`](../reference/presentation.md#single-choice-rows) rule only decides whether to draw the row of a field this rule has answered without a payload. A scalar [EnumSource](../reference/presentation.md#common-field-metadata) host follows the same rule when its source offers exactly one selectable member; a [List host](../reference/schema.md#enumsource-list-hosts) never takes it.

### Default request entry and dynamic actions

Owner: [protocols.md#default-request-entry-and-dynamic-actions](../reference/protocols.md#default-request-entry-and-dynamic-actions)

- `entry.default-request.optional`: DefaultRequest is optional; an input-free payload alone does not make a Request the default.
- `entry.default-request.owns-entry.authoring`: Declare DefaultRequest for Request-led entry; otherwise the [default subscription or available-request landing](../reference/protocols.md#default-request-entry-and-dynamic-actions) owns entry.

### Single-action confirmation and input presentation

Owner: [protocols.md#single-action-confirmation-and-input-presentation](../reference/protocols.md#single-action-confirmation-and-input-presentation)

- `actions.input.defaults-keep-input.authoring`: Use Default and CopyFrom to prepopulate values; [editable input still requires the person’s submission](../reference/protocols.md#single-action-confirmation-and-input-presentation).

### Hierarchical single field requests

Owner: [protocols.md#hierarchical-single-field-requests](../reference/protocols.md#hierarchical-single-field-requests)

- `actions.hierarchical.shape`: **Hierarchical single-field requests.** A single editable choice field may be a non-nullable Union whose branches carry Enumerations or further such Unions. Single-field record wrappers may preserve that shape; Hidden construction fields do not add editable input. Every branch must have this choice-only shape. Other editable payloads, nullable containers, Lists, or visible ReadOnly/Fixed companion content retain the ordinary full editor. This is general one-field request composition, applicable to category/item selection as well as navigation; it requires no new attribute, grouping catalog or template opt-in.

### Single-action confirmation and input presentation

Owner: [protocols.md#single-action-confirmation-and-input-presentation](../reference/protocols.md#single-action-confirmation-and-input-presentation)

- `actions.hierarchical.sources-available`: Sources needed while browsing inactive branches must be available through normal construction without changing the live selection, invoking calculations, or sending domain requests to discover choices.
- `actions.hierarchical.overview-leaf` (judgment): For hierarchical navigation, a group's overview must be an explicit destination leaf; its expand/collapse header is not an action. [AI Search](../reference/protocols.md#candidate-discovery-and-assembly-before-ai) reuses this builder to construct one complete invocation for each eligible leaf of an offered action. The navigation hierarchy can therefore serve both Go to and Search without a second catalog of bound links.
- `actions.confirmation.dialog.authoring`: Mark critical actions RequestConfirmation to require the [platform’s Send/Cancel confirmation](../reference/protocols.md#request-confirmation) at person-facing entry points.

### Request confirmation

Owner: [protocols.md#request-confirmation](../reference/protocols.md#request-confirmation)

- `actions.confirmation.declaration`: **Critical requests.** `@LibertasRequestConfirmation` and `#[libertas_request_confirmation]` take no argument and emit `{"Name":"RequestConfirmation","Value":"true"}` on a protocol Request variant, the marker form of DefaultRequest and LinkOnly.

### Single-action confirmation and input presentation

Owner: [protocols.md#single-action-confirmation-and-input-presentation](../reference/protocols.md#single-action-confirmation-and-input-presentation)

- `actions.confirmation.ai-assistant`: The marker is a rule for presentation to a person and a requirement on the AI assistant. For the AI assistant, the Hub starts the generated tool's description with "Requires user confirmation." and instructs the assistant, in every turn that offers such a tool, to state the action and its values and to call the tool only after the user confirms in a later message; the Hub does not verify the confirmation. The marker does not change the wire form of the request or its validation. A program that calls the endpoint directly is not affected, and the service still validates every request.
- `actions.auto-requests-only`: Apart from the [landing forwarding rule](../reference/protocols.md#default-request-entry-and-dynamic-actions), the only requests that follow a reply without the person's choice are auxiliary [field calculations](../reference/protocols.md#field-calculation-requests) and the [refresh](../reference/presentation.md#master-detail-panes) of a master-detail leading page.

### Request editor variant identity

Owner: [protocols.md#request-editor-variant-identity](../reference/protocols.md#request-editor-variant-identity)

- `actions.editor.one-variant.authoring`: Expose operation choices through NextRequest, Link and PrevRequest; [each editor owns one protocol variant](../reference/protocols.md#request-editor-variant-identity), while payload Unions remain data choices.

### Response action constraints

Owner: [protocols.md#response-action-constraints](../reference/protocols.md#response-action-constraints)

- `actions.auto-selection.authoring`: Supply visible response content when it must remain displayed; [automatic action selection never submits the selected request](../reference/protocols.md#response-action-constraints).
- `actions.response-constraint.membership-only`: NextRequest declares the complete possible action set and its display order. The ConstrainedBy array supplies membership only: its order never reorders actions. For example, NextRequest `A,B,C` with allowed discriminants for `C,A` displays `A,C`. No separate order attribute is needed.
- `actions.response-constraint.server-discriminants`: Each integer in the response's allowed array identifies an original protocol Union discriminant of the server schema, never a position in NextRequest or a filtered action list; a client App whose protocol arranges the Union differently reads it under the [capability prefix rule](../reference/protocols.md#capability-prefixes).
- `actions.response-constraint.empty-and-duplicates`: Only declared actions in the allowed set are displayed, after the ordinary access and ExcludeUI restrictions. An empty array is valid and means no actions; duplicate entries do not duplicate actions. Landing uses its separate [forwarding rule](../reference/protocols.md#default-request-entry-and-dynamic-actions).
- `actions.response-constraint.removal-keeps-surface`: The set belongs to the installed response and is replaced only with a newly accepted response or subscription update. It is not evaluated against an editor or another interaction. An update that removes a request from the set neither leaves nor replaces the surface: a popup, editor, or confirmation dialog already opened for that request may still send it, and the service checks it as it checks every request. Every action entry point, including declared [links](../reference/protocols.md#links) on its rows and menus, uses the same filtered set. Backend authorization and state checks still apply when a selected request arrives.

### Named action controls

Owner: [protocols.md#named-action-controls](../reference/protocols.md#named-action-controls)

- `actions.control.dual-role`: On a dual-role variant it applies only while the value is displayed, never in its Request editor. Hidden keeps the source list out of content and editors; the marker creates a separate control for its choices.
- `actions.control.membership`: The App supplies each control's current membership as original protocol Union Request discriminants of the server schema in the marked field, read under the [capability prefix rule](../reference/protocols.md#capability-prefixes). The field array never names a NextRequest position or changes order. Start with `NextRequest`, apply the response's existing ConstrainedBy ceiling, access and ExcludeUI restrictions, then intersect each control with its own array. LinkOnly requests are absent from every control. Duplicates collapse; an empty array hides only that control. Without an `Actions` field, Actions offers the shared eligible set. A named control does not remove its requests from Actions. To separate the controls, the App supplies an explicit Actions array.
- `actions.control.links-shared-set`: Links continue to use the shared response set independently of control-specific arrays.

### Undefined and its sources

Owner: [schema.md#undefined-and-its-sources](../reference/schema.md#undefined-and-its-sources)

- `init.undefined-relationships` (judgment): **A node is undefined until it is initialized** by `Fixed`, a `Default`, a copy, a supplied value, or [type initialization](../reference/schema.md#ordinary-initialization) (an empty List, a null non-editable Nullable); otherwise it stays undefined until answered, which is how a must-answer question is asked ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). Undefined cannot be encoded, so no protocol message carries it; only an editor holds it, showing its neutral empty state ([unanswered fields](../reference/presentation.md#unanswered-fields)) and blocking submission while required.
- `init.undefined.constrained-by` (judgment): **[`ConstrainedBy`](../reference/protocols.md#backend-constrained-choices):** an undefined non-nullable field with a one-entry allowed set takes that entry, as a Union `Default` selects its variant; an empty set leaves a Nullable field only null; a constructed value outside the set stays a field error. When a result changes a [calculated set](../reference/protocols.md#calculated-allowed-sets), a selection outside it resets to undefined, never null; its `Default` reapplies only if the new set contains it, and the singleton rule follows.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Backend-constrained choices

Owner: [protocols.md#backend-constrained-choices](../reference/protocols.md#backend-constrained-choices)

- `choices.constrained-by.placement`: Field ConstrainedBy constrains an Enumeration, a Union or a keyed EnumSource host in request data; on a Response it constrains NextRequest
- `choices.source.hidden-int-list`: The ConstrainedBy source is a Hidden non-nullable list of non-nullable integers
- `choices.studio-checks`: A WorkflowRequest initializes the ConstrainedBy source or its containing cookie through CopyFrom

### Default request entry and dynamic actions

Owner: [protocols.md#default-request-entry-and-dynamic-actions](../reference/protocols.md#default-request-entry-and-dynamic-actions)

- `entry.default-request.declaration`: DefaultRequest marks at most one input-free Request variant, never ExcludeUI or LinkOnly
- `entry.default-request.no-chart`: A chart-producing request cannot be DefaultRequest

### Single-action confirmation and input presentation

Owner: [protocols.md#single-action-confirmation-and-input-presentation](../reference/protocols.md#single-action-confirmation-and-input-presentation)

- `actions.confirmation.placement`: RequestConfirmation goes only on a variant carrying Request, never on a DefaultRequest

### Response action constraints

Owner: [protocols.md#response-action-constraints](../reference/protocols.md#response-action-constraints)

- `actions.response-constraint.declaration`: Response ConstrainedBy needs NextRequest, excludes WorkflowRequest and Error, and reads a hidden integer list

### Named action controls

Owner: [protocols.md#named-action-controls](../reference/protocols.md#named-action-controls)

- `actions.control.declaration`: Named action controls sit on direct Hidden non-nullable integer-List fields of a displayed NextRequest page
- `actions.control.name`: A control name is a case-sensitive identifier unique on its page; Actions names the built-in control

<!-- /generated -->
