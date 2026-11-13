# Writing a workflow

Load this card when a task adds or changes a workflow: a request editor the
service opens with `EditRequest` or `WorkflowRequest`, its Next/Submit and
`PrevRequest` actions, its completion, or the service's handling of a changed
earlier page. Its rules are generated from the
[workflow contract](../reference/protocols.md#workflow-interactions), which
owns them. The protocol itself follows the
[Endpoint protocol card](endpoint_protocol.md); master-detail placement
follows the [pane contract](../reference/presentation.md#master-detail-panes),
and local draft recovery follows
[CacheIdentity](../reference/annotations.md#cacheidentity).

## Order of work

1. Choose how each editor is entered by what it needs
   (`workflow.edit-request.why`, `workflow.client-built.why`,
   `workflow.workflow-request.why`): `EditRequest` when nothing would start
   undefined (stored data, or an unsaved value equal to its initialization); an
   offered client-built request (`NextRequest` action, link or landing) when
   fields stay undefined and the person chooses the page; `WorkflowRequest`
   when fields stay undefined and the service opens the page at once. Both
   commonly copy known facts by `CopyFrom`; who opens the page decides. A must-answer field is
   required with no Default (`workflow.strict-question.required`). Declare the
   replies each request admits (`workflow.next-response.*`).
2. Declare the previous actions and their construction (`workflow.prev.*`,
   `workflow.cookie.*`, `workflow.back.*`).
3. Design completion and progress (`workflow.completion.*`,
   `workflow.progress.*`).
4. Make the service reconcile a changed earlier page before it continues
   (`workflow.saved-pages.*`), then compile with Studio and fix every
   reported rule.

<!-- generated:codebot-rules task=workflow -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Eligibility and field placement

Owner: [annotations.md#eligibility-and-field-placement](../reference/annotations.md#eligibility-and-field-placement)

- `draft.eligibility.small-requests`: `CacheIdentity` is a Boolean field attribute. A workflow page is cacheable only when at least one root-record field has effective `CacheIdentity=true`. Use it for large workflow requests where losing completed field edits would require substantial re-entry. Small requests must omit CacheIdentity; clients do not guess eligibility from payload size or field count.

### Identity, revisions, and application responsibility

Owner: [annotations.md#identity-revisions-and-application-responsibility](../reference/annotations.md#identity-revisions-and-application-responsibility)

- `draft.revisions.application-owned`: Protocol changes are allowed. Application data revisions are independent of protocol/schema versions; tasks need not detect protocol changes. Management tools guard schema compatibility, and clients reject incompatible cached data at restoration. Compatible protocol changes alone do not invalidate drafts.

### Choosing a role

Owner: [protocols.md#choosing-a-role](../reference/protocols.md#choosing-a-role)

- `protocol.choice.list` (judgment): **Choosing the role for a need.** Each need has one role; the linked section owns its rules. Never substitute one role for another.
- `protocol.choice.one-shot` (judgment): **Do or find something once:** a `Request` whose `NextResponse` names its `Response` replies.
- `protocol.choice.offered-actions` (judgment): **Further actions the person may choose:** the Response's `NextRequest`; offered, opened only by the person or the [automatic singleton selection](../reference/protocols.md#default-request-entry-and-dynamic-actions).
- `protocol.choice.strict-question` (judgment): **Ask the person something now:** a Response declaring `WorkflowRequest`; a page the person chooses is an offered client-built request ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)).
- `protocol.choice.edit-existing` (judgment): **Edit data the service has stored:** `EditRequest` ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)).
- `protocol.choice.previous-page` (judgment): **Return to an earlier page:** `PrevRequest` on the Request; the service decides what follows and the client keeps no history ([previous actions](../reference/protocols.md#submit-previous-actions-and-completion)).

### Protocol roles

Owner: [protocols.md#protocol-roles](../reference/protocols.md#protocol-roles)

- `protocol.edit-request.request-variant`: `EditRequest` is a correlated server reply carrying a complete `Request` value: its variant must carry `Request`, and a Request-only variant suffices. EditRequest is an operation, not an additional schema role, and receiving it does not invoke the request.

### Workflowrequest semi role

Owner: [protocols.md#workflowrequest-semi-role](../reference/protocols.md#workflowrequest-semi-role)

- `workflow.workflow-request.opens-editor` (judgment): **WorkflowRequest semi-role.** `WorkflowRequest` specializes `Response` through a schema attribute. `NextRequest` identifies available next request actions; `WorkflowRequest` requires the client to construct and open its named request immediately when the variant is received as a correlated Response.

### Protocol roles

Owner: [protocols.md#protocol-roles](../reference/protocols.md#protocol-roles)

- `workflow.workflow-request.awaits-submission` (judgment): The client uses the received response under the reference context and request construction rule above. The response may contain arbitrary valid cookie data or be empty; it is not shown as an intermediate screen. The resulting editor may be incomplete and awaits explicit user submission, even for Nil, already complete, or entirely Fixed/ReadOnly payloads. Normal request visibility and access rules apply; an unavailable target must not be silently replaced with another request.
- `workflow.workflow-request.dual-role` (judgment): For example, a variant carrying both `Response` and `SubscriptionData` can use `WorkflowRequest` to open its named editor on correlated Response delivery. Subscription delivery of that same variant updates subscribed data without opening the editor, under the shared role rule above.

### Workflow interactions

Owner: [protocols.md#workflow-interactions](../reference/protocols.md#workflow-interactions)

- `workflow.mix-state-machine` (judgment): This is the unreleased workflow interaction contract. A protocol Union may mix state-machine interactions, whose stable screen is a response or subscription report, and workflows, whose stable screen is a request editor. The service selects every resulting transition. There is no separate machine-type marker.

### Valid data and editor state

Owner: [protocols.md#valid-data-and-editor-state](../reference/protocols.md#valid-data-and-editor-state)

- `workflow.valid-data` (judgment): Every transmitted value must be structurally valid for its Avro schema. An editor can also contain an undefined field: it has neither a valid value nor an encoded null. A required undefined field prevents submission. A validation rule that reads an undefined value is aborted and reports a missing answer ([validation rules](../reference/validation-rules.md)); undefined never changes the wire schema or makes the field nullable.
- `workflow.no-partial-requests` (judgment): No partially filled request is transmitted for initialization or backend draft persistence. The [CacheIdentity contract](../reference/annotations.md#cacheidentity) defines opt-in local recovery of committed field values on large workflow pages, independently of backend persistence. Uncommitted control text is discarded on exit. Without CacheIdentity, all unfinished page edits are discarded and the application resumes from its own saved, valid data.

### Strict questions and data pages

Owner: [protocols.md#strict-questions-and-data-pages](../reference/protocols.md#strict-questions-and-data-pages)

- `workflow.entry.undefined-not-encoded` (judgment): **EditRequest or WorkflowRequest.** A node is [undefined until it is initialized](../reference/schema.md#undefined-and-its-sources), and undefined cannot be encoded, so no message carries an unanswered field. An editor opened by either one carries the platform's two workflow buttons, **Go Back** (when the Request declares `PrevRequest`) and **Next/Submit**, whose captions a Display Template may customize through [`button_labels`](../reference/presentation.md#button-labels); the App adds no buttons of its own for them ([submit and previous actions](../reference/protocols.md#submit-previous-actions-and-completion)). What the page needs decides how it opens:
- `workflow.edit-request.why` (judgment): **Nothing would start undefined: EditRequest.** It carries a Request the service has stored, or an unsaved page's value equal to its ordinary initialization; the client adopts it as it is. Its Nullables are true optionals whose null is a real answer with the meaning its `NullText` names, such as "None" or "Does not apply".
- `workflow.client-built.why` (judgment): **Fields left undefined and the person chooses the page: a client-built request.** The client constructs it from an offered action: a `NextRequest` action, a link (available while its target is in that action set), or a landing request. Like a WorkflowRequest editor, it commonly copies known facts by `CopyFrom` from the displayed Response, and every field nothing initializes stays undefined. An offered request opens only when the person chooses it, or by the [automatic singleton selection](../reference/protocols.md#response-action-constraints). Who opens the page, not copying, separates it from WorkflowRequest.
- `workflow.workflow-request.why` (judgment): **Fields left undefined and the service opens the page at once: WorkflowRequest**, with or without copied facts, and only then. The service sends a Response; the client constructs the named Request, copies any known facts by `CopyFrom`, and leaves the rest undefined. An unanswered source path is absent, never null: CopyFrom stops at a valid null, which a non-Nullable field cannot take.
- `workflow.strict-question.required`: **A must-answer question** is a required field with no `Default` on a request the client constructs (an offered action, link or landing request, or a `WorkflowRequest` editor): it starts unanswered, shows no error, and blocks submission. Never model it as a `Nullable` whose `NullText` means "not answered yet", and never send it in an EditRequest.

### Entering and continuing a workflow

Owner: [protocols.md#entering-and-continuing-a-workflow](../reference/protocols.md#entering-and-continuing-a-workflow)

- `workflow.leading-page-row` (judgment): A resolved editing Request can identify its leading-page row through an [enum/protocol association](../reference/schema.md#enumeration-protocol-associations), including direct entry; see [master-detail panes](../reference/presentation.md#master-detail-panes).
- `workflow.edit-request.complete-value`: **EditRequest:** a server-to-client opcode carrying one complete, structurally valid Request variant from the Endpoint protocol Union. It is a correlated reply to the pending request that owns the next transition, for example Start, Submit, Prev, Resume, an action, or a link, and completes that operation under the [shared role rule](../reference/protocols.md#protocol-role-reuse). The client uses the supplied value directly, without running CopyFrom or default initialization over it.
- `workflow.edit-request.awaits-submission` (judgment): EditRequest also awaits explicit user submission, including Nil, already complete, or entirely Fixed/ReadOnly values; it must not automatically resubmit them. Its reference context follows the [current reference context rule](../reference/protocols.md#interaction-reference-context).
- `workflow.edit-request.replaces-editor` (judgment): EditRequest replaces the active editor even when the returned variant is the same variant as the previous step; variant identity is not step identity.
- `workflow.delivered-editor-access` (judgment): When a delivered editor, whether the target of a WorkflowRequest or the variant of an EditRequest, is beyond the user's [AccessPrivilege](../reference/protocols.md#request-access), the client refuses the reply by the error procedure and keeps the originating surface, its reference context, and its draft. `ExcludeUI` and [`LinkOnly`](../reference/protocols.md#link-only) govern what a client offers, not what the service delivers: a delivered editor that carries either is installed.
- `workflow.next-response.reply-membership` (judgment): When NextResponse is declared, a successful transition reply must be one of its named variants. Error responses and standardized failure statuses use the shared error/retry path and do not need to appear in this navigation list. These declarations constrain the possible replies; the client never selects the service's reply for it.

### Submit, previous actions, and completion

Owner: [protocols.md#submit-previous-actions-and-completion](../reference/protocols.md#submit-previous-actions-and-completion)

- `workflow.submit.captions` (judgment): These are control roles: a page's Display Template may supply localized [button captions](../reference/presentation.md#button-labels) without changing their behavior, eligibility, or request names inside a chooser.
- `workflow.prev.order` (judgment): Their declared order is presentation order. With one eligible target, Back selects it directly; with several, Back opens a chooser using their ordinary localized names. This is the workflow's in-app **Go Back** action, distinct from screen-navigation and system Back. With none, there is no workflow Back action.
- `workflow.prev.master-detail-suppressed` (judgment): While the editor is embedded in a master-detail trailing pane, clients suppress all workflow Back controls, including the chooser and shortcuts, under the [pane contract](../reference/presentation.md#master-detail-panes). The pane offers Close and Next; the leading pane owns navigation for the two-panel arrangement, and the App must make that navigation complete.
- `workflow.go-to-composer` (judgment): A request whose purpose is to choose where to go, such as a section chooser offered as a "Go to" action, belongs in a composer above the page it leaves. There is no reason to deliver it as a workflow editor through WorkflowRequest or EditRequest: such a page would exist only to be left, and the service already opens the chosen destination as the reply to the composed request.
- `workflow.prev.not-history` (judgment): `PrevRequest` names this action declaration, not a stored previous request or reference context. Its declared target list is not dynamically filtered by `ConstrainedBy`.
- `workflow.prev.construction` (judgment): Each selected Back action constructs an ordinary request using the [shared CopyFrom rules](../reference/schema.md#shared-copyfrom-and-copyto-rules). Its reference object is the current editor's selected payload, independently of the unfinished editor's validity.
- `workflow.cookie.app-owned` (judgment): A hidden application-defined cookie can identify any of several concurrent workflows. Cookie design and historical selection belong to the application. Accepting a historical edit must follow the [saved-page consistency rule](../reference/protocols.md#saved-page-consistency-after-historical-edits). Studio validates declarations and statically decidable mapping rules; it does not prove that cookie fields are initialized on every path.
- `workflow.back.abandonment` (judgment): Back expresses abandonment of the current edit. The backend decides what to return, including a complete historical request through EditRequest. The client does not navigate through cached historical requests. Construct and validate the action before sending it; a rejected selection or failed attempt follows the ordinary error/retry procedure. There is no server-side undo implied by discarding the current edit.
- `workflow.completion.terminal` (judgment): For a workflow Submit or Prev operation, a non-error response without WorkflowRequest ends the editor and is presented as the resulting stable response. NextRequest, if present, offers its declared state-machine actions. With no NextRequest, the response is a valid terminal screen, for example a FormattedText congratulations message. The state-machine acknowledgement rule for ordinary requests outside a workflow remains unchanged.
- `workflow.native-back.no-prev.authoring`: Design server entry and resumption for [local Back or Close without PrevRequest](../reference/protocols.md#submit-previous-actions-and-completion); leaving cannot undo an already received submission.

### Saved-page consistency after historical edits

Owner: [protocols.md#saved-page-consistency-after-historical-edits](../reference/protocols.md#saved-page-consistency-after-historical-edits)

- `workflow.saved-pages.compare` (judgment): Compare the submitted page answers with the saved answers before changing any stored page. Ignore transport cookies, navigation metadata, and presentation-only values in this comparison. An unchanged Next is review/navigation: preserve all saved pages and completion state. Opening, canceling, or rejecting an edit also preserves saved history.
- `workflow.saved-pages.reconcile` (judgment): When the backend accepts a changed page, it **must**, for each stored page that the changed answers affect, do one of the following: recompute and revalidate that page's data against the revised answers and save the consistent result; erase that page's data; or keep the page's answers and mark the page unanswered, so that it is asked again, with those answers, before the workflow can complete. This is a mandatory application rule, not an optional client navigation policy.
- `workflow.saved-pages.atomic` (judgment): Do not erase before validation or leave a partially updated state visible. No page remains answered for resume, calculation, or completion merely because it was valid before the change; derived results, cookies, allowed-choice sets, and saved progress must agree with the revised answers as well.
- `workflow.saved-pages.before-continuing` (judgment): Complete this reconciliation before continuing or resuming the workflow. Resume and EditRequest use only the consistent stored state; retries, concurrent editors, and late submissions must not restore data from before the change. The backend owns persistence and consistency; the client and Studio cannot infer application dependencies or repair saved pages.
- `workflow.saved-pages.opening-changes-nothing` (judgment): Opening a historical editor does not itself change anything. An abandoned or rejected draft changes no saved history; the rule applies when the backend accepts and saves the change. For example, changing a saved filing status from joint to single erases the spouse data that no page of a single filer owns, and asks again each affected page whose stored answers no longer validate, before the interview can complete.

### Validation context and progress

Owner: [protocols.md#validation-context-and-progress](../reference/protocols.md#validation-context-and-progress)

- `workflow.validation-context` (judgment): Editable requests use the [shared path contract](../reference/schema.md#context-and-syntax). Supported references use the [current reference context](../reference/protocols.md#interaction-reference-context), including on inherited types and hidden fields. CopyFrom remains separate initialization metadata with its defined invocation source. EditRequest context follows that same owner; it does not retain an older response in its place.
- `workflow.progress.formatted-text` (judgment): Progress is an ordinary FormattedText field following the [static read-only placement and request-value contract](../reference/presentation.md#static-read-only-placement-and-request-values). For example, the service can supply it in EditRequest or response data copied into a WorkflowRequest editor. There is no progress opcode, percentage schema, fixed step count, or client inference from previous-action lists or history.

### Undefined and its sources

Owner: [schema.md#undefined-and-its-sources](../reference/schema.md#undefined-and-its-sources)

- `init.undefined-relationships` (judgment): **A node is undefined until it is initialized** by `Fixed`, a `Default`, a copy, a supplied value, or [type initialization](../reference/schema.md#ordinary-initialization) (an empty List, a null non-editable Nullable); otherwise it stays undefined until answered, which is how a must-answer question is asked ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). Undefined cannot be encoded, so no protocol message carries it; only an editor holds it, showing its neutral empty state ([unanswered fields](../reference/presentation.md#unanswered-fields)) and blocking submission while required.
- `init.undefined.construction` (judgment): **Construction:** a `Default`, `$null` included, answers the node; `Fixed` answers and locks it; `CopyFrom` answers only the nodes it assigns.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Workflowrequest semi role

Owner: [protocols.md#workflowrequest-semi-role](../reference/protocols.md#workflowrequest-semi-role)

- `workflow.workflow-request.declaration`: WorkflowRequest is on a Response, names exactly one Request variant of the protocol, and excludes NextRequest and Error

### Entering and continuing a workflow

Owner: [protocols.md#entering-and-continuing-a-workflow](../reference/protocols.md#entering-and-continuing-a-workflow)

- `workflow.next-response.kinds`: NextResponse on a Request or SubscriptionRequest lists Response, SubscriptionData or Request variants

### Submit, previous actions, and completion

Owner: [protocols.md#submit-previous-actions-and-completion](../reference/protocols.md#submit-previous-actions-and-completion)

- `workflow.prev.declaration`: PrevRequest goes on a Request variant and names distinct Request variants of the same protocol

<!-- /generated -->
