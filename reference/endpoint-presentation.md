# Runtime endpoint presentation

This document defines the application-shell presentation shared by the Android,
iOS and Studio Libertas clients. It is not part of the chart schema grammar.

## Presentation decision

First apply [default request entry](protocols.md#default-request-entry-and-dynamic-actions).
The following chooser/workspace rules apply when explicit request selection is
needed; they do not bypass an explicit default request, default subscription, or an installed response's legal action set. Without a default entry, the request list is the landing; executable chart requests appear as previews instead of rows. The client resolves that presentation from the endpoint schema:

- <!-- rule: endpoint.presentation.quick-actions class=judgment enforced=no tasks=client-runtime --> an endpoint whose requests are all non-chart opens a protocol-only **Quick
  Actions** drawer;
- <!-- rule: endpoint.presentation.workspace class=judgment enforced=no tasks=client-runtime --> an endpoint with one or more chart-producing requests opens a full-height
  **Endpoint Workspace**. On a phone this is a navigation destination. A wide
  client may use a full-height pane, but content gestures never dismiss it.

<!-- rule: endpoint.presentation.chart-boundary class=judgment enforced=no tasks=client-runtime -->
Chart capability in the complete endpoint schema is the deterministic promotion
boundary. A chart-producing request promotes the endpoint even when its request
fields lack complete defaults and therefore cannot produce an automatic preview.
For this contract, chart-producing includes a response or subscription-data
path that resolves to either the `Chart` or the strictly parallel `Chart3D`
presentation namespace.
Loading state, default completeness, response success, chart row count,
orientation, and prior visits must not change the selected presentation after it
opens.

## Endpoint list with displayed subscription data

For a task with multiple endpoints, the endpoint list may display default
subscription data. That inline data also serves as its endpoint's landing.
Other landings, such as chart and workflow landings, stay unopened until the
endpoint is tapped and then stack above the list. A task with exactly one
endpoint enters its landing directly, with no endpoint-list screen in the stack. <!-- rule: endpoint.list.header-drawer class=hard enforced=no tasks=client-runtime --> Tapping its header opens a request drawer. Use the displayed
value's legal NextRequest choices in their declared order, with existing access,
ExcludeUI, applicable ConstrainedBy, and [LinkOnly](protocols.md#link-only)
filtering. Construct the selected request
using the originating interaction's [reference context](protocols.md#interaction-reference-context).
Show the drawer even for one choice; with
no legal choices the header has no request action.

<!-- rule: endpoint.header.no-duplicate-subscription class=hard enforced=no tasks=client-runtime -->
Do not reissue the default subscription or stack a duplicate data screen merely
because the endpoint header was tapped. This is action selection on an existing
destination, not initial endpoint entry. An endpoint with no displayed data and
a sole endpoint opened directly still follow the default-entry rules. Selecting
a drawer action retains the normal composer, reply, chart, and Back behavior.

## Endpoint Workspace

<!-- rule: endpoint.navigation.native-back class=hard enforced=no tasks=client-runtime -->
The workspace owns one vertical content scroll and a persistent title/back bar.
Because workspaces, previews, Chart Explorer, protocol data, and request
composers are navigation destinations, every full-height destination uses the
normal back affordance; [stacked-link pages](protocols.md#links) additionally
require their system Close. Its section order is
invariant:

1. <!-- rule: endpoint.workspace.actions-section class=judgment enforced=no tasks=client-runtime --> the top **Actions** section contains every non-chart request and every
   chart-producing request that lacks complete executable defaults;
2. <!-- rule: endpoint.workspace.charts-section class=judgment enforced=no tasks=client-runtime --> the following **Charts** section contains the automatic previews whose
   chart-producing requests have complete executable defaults.

A request appears exactly once. A ready preview owns exactly one header rail.
That rail contains the chart contract title, an icon-only Configure action when
the request has editable variables, and the icon-only action that opens
Chart Explorer. There is no separate protocol-title/configure row above a ready
preview. <!-- rule: endpoint.workspace.configure-variables class=hard enforced=no tasks=client-runtime --> `Nil` is never a variable. `Fixed` and `ReadOnly` values are not
variables, while `Default` only supplies an initial value and remains editable.
A request whose effective tree contains no editable variable is not configurable
and must not show Configure. Loading and unavailable previews may use the
localized request label, but their Configure action follows the same rule. <!-- rule: endpoint.workspace.fixed-in-task-configuration class=hard enforced=no tasks=client-runtime -->
This request-payload rule does not reinterpret `Fixed` on a client Endpoint in
task configuration: there it constrains the peer task process while the
Endpoint-ID selector remains editable across all schema-compatible instances.

A chart preview is a live but gesture-neutral summary. It retains the chart
contract's deterministic initial viewport, clipping, category-density rules,
resampling disclosure, and overflow indicators. It accepts no direct pan,
zoom, selection scrub, or internal vertical/horizontal scrolling. A tap or the
localized open-chart control opens Chart Explorer. Only Chart Explorer owns
chart viewport manipulation, tooltips, guide overlays, and the complete
scrollable chart content.

For a 3D preview, the localized open-chart action is **Explore 3D chart**. It
opens a new full-screen destination without an inline camera mode. All direct 3D inspection and camera gestures
are installed only in the pushed Chart Explorer destination.

<!-- rule: endpoint.preview.failure-retains-workspace class=hard enforced=no tasks=client-runtime -->
If a default chart cannot be rendered, its preview follows the shared
[chart failure policy](charts/2d.md#10-validation-and-fallback). The implementation
may provide an ordinary-data fallback or display an error message. Preserve the
endpoint workspace and its response ownership. The same choice applies to Chart
Explorer; an implementation may return a failed Explorer to its retained preview
and present the fallback or error there. Presentation failure never closes the
workspace or reissues the request.

Opening Chart Explorer or a chart preview pushes a new chart destination.
Leaving a full-screen chart pops exactly that destination and reveals the
previous chart or workspace instead of closing the complete interaction stack.
The universal Chart3D vocabulary calls these navigation operations
`fullscreen.open` and `fullscreen.close`; for a 3D chart the latter is performed
through the application shell's normal Back affordance rather than separate
chart Close chrome.

<!-- rule: endpoint.preview.direct-explorer class=hard enforced=no tasks=client-runtime -->
Configure pushes a separate request-composer screen above the retained chart
preview workspace. On a successful chart response, remove only the composer and
push Chart Explorer above the retained workspace. Do not replace the preview
with ProtocolData or insert a ProtocolData destination between the preview and
Explorer. Back returns to the same preview workspace. This applies to both 2D
and 3D charts on Android and iOS. Cancelling the composer leaves the workspace
intact; an error preserves the composer for retry without changing the preview.
The protocol transaction stack below governs protocol-data interactions, not
this preview-originated chart customization path.

<!-- rule: endpoint.workspace.preview-context class=hard enforced=no tasks=client-runtime -->
Each automatic preview owns its [reference context](protocols.md#interaction-reference-context).
Configure uses that exact preview's context for `CopyFrom` and supported external
references under the [shared path contract](schema.md#context-and-syntax), including
its retained successful Response and current SubscriptionData when subscribed.
Do not substitute a sibling preview's data or the endpoint value that originally
opened the chooser after this preview receives its own data. Before it receives
any data, retain the chooser's originating reference context when available;
otherwise ordinary initialization with an empty context applies.

Rotation preserves navigation depth and the current interaction. Chart sizing,
clipping, overflow, and rejection follow the [2D](charts/2d.md) or
[3D](charts/3d.md) contract for the displayed data.

<!-- rule: endpoint.workspace.explicit-close class=hard enforced=no tasks=client-runtime -->
The workspace's vertical scroll therefore has one owner on every pointer path,
including paths that begin over a chart preview. Closing the workspace is an
explicit navigation action; downward content overscroll never closes it.

## Protocol transaction stack

<!-- rule: endpoint.composer.pending-reply-owner class=hard enforced=no tasks=client-runtime -->
A request opened from an existing screen is a **transient request composer**
stacked above that screen, including a request selected from an endpoint action
list. Cancelling before submission removes only the composer and preserves the
originating screen, its data, and its reference context. It does not run
landing navigation. While submission is pending, the composer remains the reply
owner and cannot be dismissed through native Back;
[exit while an operation is pending](protocols.md#exit-while-pending) states
the one exit. An Error keeps it available
for correction and retry; a non-transition acknowledgement removes only the
composer; a transitioning reply installs or updates the response destination
as described below. Subscription updates may continue to refresh their owning
screen underneath the composer. An ordinary [link](protocols.md#links) opens
this composer when its target has editable input; above a stable editor its
plain reply is applied to that editor as a
[return](protocols.md#link-return) rather than an acknowledgement. <!-- rule: endpoint.composer.no-workflow-back class=hard enforced=no tasks=client-runtime --> A composer
never offers a workflow Back control, even when its Request declares
`PrevRequest`; its Cancel closes it while idle
([previous actions](protocols.md#submit-previous-actions-and-completion)).

A **stable endpoint destination** is an endpoint action/data screen, or the
request editor entered through EditRequest or WorkflowRequest. Landing entry
may deliver such a stable editor; composing an ordinary landing request still
uses the transient composer and its cancellation rules. Receiving a workflow
entry or installing a response ends the transient composer's role; subsequent
Back follows the stable destination's landing rules. Workflow steps replace
one active editor rather than building a history of editors.

A subscription landing retains its live page, layout, and scroll position
underneath at most one working page. Request-list and DefaultRequest entry use
one working page, with later results replacing or updating it.

Native Back from the bottom working page exits the endpoint, including a
workflow editor or a dynamically selected resume page. It does not compare
message types or reveal or restart landing. A stacked idle composer, picker,
modal, chart, or [stacked-link page](protocols.md#links) closes first and
reveals the underlying live page; sent requests
follow the pending-operation exit rule. See
[landing and Back](protocols.md#control-endpoint-landing-and-native-back).

Neither model keeps a history of intermediate working pages. A transient
composer may sit above the current page. [Stacked links](protocols.md#links)
add live read-only Response pages above that same surface, with per-page
reply/subscription ownership and local pop-only Close; their owner defines
unlimited nesting and release on non-stacked navigation. A multi-endpoint task retains its
endpoint list outside that interaction; a single-endpoint task has no
endpoint-list page. See [landing and Back](protocols.md#control-endpoint-landing-and-native-back).

On Android and iOS, ProtocolData displays the response or subscription data
owned by that screen. Its page title represents the current Union
discriminator using its localized name, or the text of a Display Template
[`label` or `title`](presentation.md#labels-and-titles) rule on the variant. A
[promoted](presentation.md#promotion-flattening-and-union-selectors)
child appears on a separate row at the same indentation, preserving the
Union's identity.
The heading and native top bar use the title of the active displayed Request
or Response under the [protocol page-title rule](presentation.md#protocol-page-heading-and-top-bar),
including selected-variant fallback when the root Union suppresses its label.
An editor uses its own variant, not the originating Response.
The **Actions** button exposes the requests below with its complete localized
label and native-sized tap target. The
[page-button rule](presentation.md#protocol-page-heading-and-top-bar)
places it, with any named controls, in the top bar whenever the complete group
fits there, ahead of the top-bar title; otherwise the group uses the row below
the content heading, as it does when the page's Display Template asks for
the [named-control region](presentation.md#named-action-controls). The same
rule owns pinning while the content scrolls and the content heading. A
trailing master-detail page retains its own content heading, with its
buttons on the row below it; the leading page owns the shared top bar.

<!-- rule: endpoint.actions.validate-before-send class=hard enforced=no tasks=client-runtime -->
Requests come from the displayed variant's effective `NextRequest`, preserving
declared order and existing visibility/access, ConstrainedBy, and LinkOnly filtering,
narrowed to an explicit `Actions` array when the page supplies one; a page may
also declare further [named action controls](protocols.md#named-action-controls). Promoting a
child changes presentation only; request resolution still belongs
to the enclosing protocol variant. Follow the
[Actions selection and input rules](protocols.md#single-action-confirmation-and-input-presentation):
hide the button with no choice; tapping it selects a sole choice directly or
opens the chooser for several. Selection sends an input-free request after
validation and any required confirmation, or opens editable input and waits. <!-- rule: endpoint.actions.missing-next-request class=hard enforced=no tasks=client-runtime -->
Missing `NextRequest` never implies every request in the protocol.

<!-- rule: endpoint.actions.showing-never-sends class=hard enforced=no tasks=client-runtime -->
The underlying ProtocolData screen remains intact while choosing or composing.
Cancelling returns to that same screen. Selection and composition preserve its
displayed data and subscription ownership until the ordinary reply handling
applies. Showing or relocating Actions never sends a request.
A transitioning action reply belongs to its working interaction, even when it
uses the default subscription's variant. Never use variant equality to replace
the retained subscription landing or pop back to it; actual subscription
deliveries continue to update their owning snapshot.

An ordinary transitioning transaction follows this sequence:
default protocol data -> Actions -> optional next-request chooser -> composer ->
send and wait -> response carrying a declared `NextRequest` -> one interactive
protocol-data destination. A sent composer remains the correlated response
owner while waiting. Receipt installs or updates the single interactive
destination with that response and exactly its declared next-request choices,
preserving the protocol interaction chain.

<!-- rule: endpoint.acknowledgement class=hard enforced=no tasks=client-runtime -->
For a regular action above a displayed value, outside the reply to the landing
entry request itself, SubscriptionRequest completion, a stacked-link completion
under [Links](protocols.md#links), and a master-detail
[refresh](presentation.md#master-detail-panes), a normal correlated Response with neither
`WorkflowRequest` nor a declared `NextRequest` is instead a
non-transition acknowledgement. It completes the in-flight attempt but does
not install a response destination, replace the displayed protocol value,
change the current choices, or interpret every
protocol Request as a choice. This is the ordinary shape for reusable
`Success`/`Failure` results. Transient request/wait presentation may finish,
but the protocol destination below it remains unchanged. Reference context
follows the [reference context rule](protocols.md#interaction-reference-context).

Apply these presentation rules under the
[shared protocol role rule](protocols.md#protocol-role-reuse). An Error-marked
correlated Response uses the separate out-of-band error procedure, preserving
retry and taking precedence over normal-response presentation.

<!-- rule: endpoint.subscription-start class=hard enforced=no tasks=client-runtime -->
A request invoked as SubscriptionRequest starts a subscription. The operation
selected under the shared role rule, not the response variant, determines this.
Later live data updates each eligible interaction associated with
that endpoint and compatible response type; it does not become an unrelated
interaction's reference context. `NextResponse`, when present, restricts the
eligible response variants. An acknowledgement leaves the displayed state and
subscription active; later matching live data can install or update that
interaction's SubscriptionData snapshot under the shared reference context rule.

<!-- rule: endpoint.subscription-release class=hard enforced=no tasks=client-runtime -->
On Hubs with explicit subscription release support (image version 20260901 or
later), leaving the last interactive screen for an explicit endpoint
subscription stops that client's explicit subscriptions for the endpoint.
Older Hubs release them when the Control connection closes. The endpoint-wide
scope can affect another explicit subscription for the same client and endpoint;
it does not stop the automatic default subscription. Reopen the interaction
when you need to subscribe again.

### Workflow editor destinations

The [workflow contract](protocols.md#workflow-interactions) defines correlated
EditRequest, automatic WorkflowRequest entry, PrevRequest construction, and
terminal response presentation. <!-- rule: endpoint.workflow.one-editor class=hard enforced=no tasks=client-runtime --> Keep one active workflow editor destination;
each successful step replaces it instead of accumulating historical editors.
Retain an actual subscription landing beneath the flow; a non-subscription
flow replaces its current stable working page.
No cookie response is displayed between steps. A completed flow replaces its
editor with the final response, including responses with no NextRequest.

<!-- rule: workflow.navigation.no-history class=hard enforced=no tasks=client-runtime -->
Display explicit Next/Submit in every workflow editor. Outside an embedded
master-detail trailing pane, also offer eligible workflow Back actions. Back
opens the ordered previous-action chooser only when several eligible actions exist; Next
submits the current editor. Entry through either workflow path always waits
for user submission. Native screen Back/dismissal exits the flow; nested-editor
and picker dismissal return to the same parent editor. Those local movements
must not invoke PrevRequest or restore historical workflow drafts.

<!-- rule: workflow.navigation.row-reserves-space class=hard enforced=no tasks=client-runtime -->
The workflow controls share one compact, persistent action row outside the
scrolling form: outside an embedded trailing pane, **Go back** is a secondary
action at the leading edge when an eligible previous action exists, and
**Next** is the primary action at the trailing edge. Multiple previous actions
retain their ordered chooser. The
row reserves its own space; it never covers fields or validation feedback.
The form uses the remaining viewport and can scroll its last field and error
fully into view. On Android and iOS, hide the whole action row while the
software keyboard is visible, including in embedded panes; restore it when
the keyboard is dismissed. Showing or dismissing the keyboard never submits
or sends a previous action. The working surface accounts for the keyboard
and system safe areas once, including when the editor is an embedded pane. Text remains
readable and controls remain usable with enlarged text and localized labels.
In an embedded master-detail pane, the row offers **Close + Next** and suppresses
all workflow previous-action controls, including their chooser and shortcuts,
under the [pane rules](presentation.md#master-detail-panes). The leading pane
owns navigation for the whole arrangement, and the App must provide all required
routes there. The pane rules also own restoration of ordinary previous
actions when the editor becomes a full page, including after narrowing the
window. Local picker and nested-field-editor dismissal retain their normal
behavior.

<!-- rule: endpoint.workflow.next-uniform class=hard enforced=no tasks=client-runtime -->
Next retains the same submission behavior on every page. Clients do not infer
a final step or change it to a special Submit action from the content,
variant names or possible replies; the service selects the result under the
[workflow contract](protocols.md#submit-previous-actions-and-completion).

## Quick Actions

The Quick Actions drawer contains only protocol requests. Native swipe or
backdrop dismissal remains valid because the drawer contains no chart preview
or chart viewport. Selection dismisses the drawer before the selected request is
presented under the
[single-action presentation rule](protocols.md#single-action-confirmation-and-input-presentation),
and cancellation/selection completion occurs exactly once.

## Cross-platform requirement

Android and iOS use the same chart-capability promotion boundary, action-first
section order, single preview header, Configure visibility, deduplication,
localized actions, preview immutability, chart/protocol stack ownership,
Chart3D full-screen-only manipulation with no inline mode,
single-interaction-data rule, adaptive title/Actions placement and single-choice
builder routing, non-transition acknowledgement behavior, and the
[single-action presentation rule](protocols.md#single-action-confirmation-and-input-presentation). Platform
navigation components and animations may remain native; they may not change
these interaction semantics.
