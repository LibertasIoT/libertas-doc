# Declaring links

Load this card when a task makes a displayed value or an editor field open a
request: `Link` (ordinary or stacked), `LinkOnly`, a link target's
construction and its reply, including a link return into the editor. Its
rules are generated from the [link contract](../reference/protocols.md#links),
which owns them. The target's own presentation follows the
[actions card](actions.md); a link return that writes a calculation's
destination or a constrained-choice source also follows the
[calculation card](calculations.md).

## Order of work

1. Place each link and name its one target (`links.declaration`,
   `links.placement`, `links.direct-declaration`, `links.rows-only-declared`).
2. Map the target's construction from the invoking node
   (`links.direction.*`, `links.typed-selector-binding`, `links.editor.*`).
3. Design the target's reply: an ordinary reply, a link return
   (`links.return.*`) or a stacked page (`links.stacked.*`).
4. Keep a row-only request off the Actions surface with `LinkOnly`
   (`links.link-only.*`), then compile with Studio and fix every reported
   rule.

<!-- generated:codebot-rules task=links -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Choosing a role

Owner: [protocols.md#choosing-a-role](../reference/protocols.md#choosing-a-role)

- `protocol.choice.list` (judgment): **Choosing the role for a need.** Each need has one role; the linked section owns its rules. Never substitute one role for another.
- `protocol.choice.link` (judgment): **Tapping a row or text invokes a request:** `Link`; a Request offered only through links is `LinkOnly` ([links](../reference/protocols.md#links), [link-only](../reference/protocols.md#link-only)).

### Protocol roles

Owner: [protocols.md#protocol-roles](../reference/protocols.md#protocol-roles)

- `links.editor.source-order`: A [link](../reference/protocols.md#links) declared in a request editor probes the bound current editor first, then the reference objects in the interaction context's order. Within each source, it tries the declared paths in order, using the shared copy availability and valid-null rules. A valid editor match wins. Parent-relative paths retain the host field's editor ancestry, and a [return](../reference/protocols.md#link-return) applies to the bound current-request destination. These logical contexts need not use separate stored objects: a received EditRequest may be both the current request and its reference object. PrevRequest construction retains its current-editor source under the [path context rules](../reference/schema.md#context-and-syntax).

### Links

Owner: [protocols.md#links](../reference/protocols.md#links)

- `links.direct-declaration`: As for [inline choices](../reference/presentation.md#inline-choices), each declaration is read directly, not through effective-attribute resolution: a link on either the field or its type enables it, a field cannot switch off its type's link, and when both declare a link the field's complete declaration, including its stacking option, applies to that occurrence. Follow named references one declaration at a time, including intermediate aliases; the nearest declaration supplies the complete Link value before any farther named type is considered.
- `links.rows-only-declared`: Rows carry actions only through declared links. No row action is inferred from CopyFrom: a Request whose CopyFrom resolves to a displayed value is not thereby offered on that value's row. Each occurrence has one target, so a row never opens a request chooser. The **Actions** surface keeps offering the displayed value's filtered NextRequest set, link targets included, except [link-only requests](../reference/protocols.md#link-only) and, when the page supplies an explicit `Actions` array, requests outside it ([named action controls](../reference/protocols.md#named-action-controls)).
- `links.direction.displayed`: On a displayed Response or SubscriptionData occurrence, including inline endpoint-list data, the tapped node is the invoking node. The client constructs the target through ordinary CopyFrom with the displayed value's [interaction reference context](../reference/protocols.md#interaction-reference-context).
- `links.direction.editor`: In a request editor, the host field is the invoking node. The client constructs the target from the current editor first, then the editor's reference objects in [context order](../reference/protocols.md#interaction-reference-context). The draft is read, never transmitted; only the constructed target is sent, and it must be complete and valid even while the editor is not. A [transient request composer](../reference/endpoint-presentation.md#protocol-transaction-stack) is a request editor for this purpose: a link on one of its fields is offered; an ordinary link's plain reply is a [return](../reference/protocols.md#link-return) into the composer, and its transitioning reply replaces the composer. A stacked link follows the display-only rules below.
- `links.typed-selector-binding`: Typed selectors in the target's CopyFrom bind the nearest self-or-ancestor occurrence of the named type on the invoking node's path, at any depth and across nested Lists, including CopyFrom on the target's descendants, under the [bound-occurrence rule](../reference/schema.md#typed-ancestors-and-bound-occurrences). From Actions, a menu, automatic singleton selection, or any other entry with no tap, the selector is unbound and the next alternative applies. The binding belongs to that one construction and its reply; it is never stored or transmitted, and a later construction does not inherit it.
- `links.return.apply.authoring`: Use a plain link Response with CopyTo to update the bound host editor under the [link-return contract](../reference/protocols.md#link-return); transitioning replies navigate instead.
- `links.stacked.declaration`: **Stacked links.** Author `@LibertasLink("ViewRecord", stacked=true)` in TypeScript or `#[libertas_link("ViewRecord", stacked = true)]` in Rust to display the reply on a read-only page above the sending surface. Availability requires stacked-link support in Studio and the client. The optional `stacked` argument is Boolean and defaults to `false`. Omission or explicit `false` emits exactly `{"Name":"Link","Value":"ViewRecord"}`, preserving existing ordinary-link behavior and schema digests. `true` emits exactly `{"Name":"Link","Value":"ViewRecord,stacked=true"}`; this is one target with an option, not a target list. A client that cannot resolve this form must display the row without a link. The option is valid wherever Link is valid.
- `links.stacked.close`: Every stacked page has a platform-owned **Close** at the leading edge of its heading. It is not an App Request and sends nothing. Neither schema nor Display Template rules may suppress or remove it; the template's [`button_labels.close`](../reference/presentation.md#button-labels) may change only its caption, with the ordinary platform-caption fallback.
- `links.stacked.navigate-away.authoring`: Use stacked links for read-only inspection with platform Close; [navigation to a non-stacked destination](../reference/protocols.md#links) leaves drafts under the ordinary CacheIdentity lifetime.
- `links.offered-when-eligible`: On a Response or SubscriptionData, a link row is offered only while its target is in the displayed value's [filtered action set](../reference/protocols.md#response-action-constraints). In a request editor the link is offered only when the target passes the same access and ExcludeUI filtering; an editor has no ConstrainedBy set. Visible regular-response content, including link rows, remains displayed even when only one action is eligible; the person opens that action through Actions.
- `links.migrate-copyfrom-rows`: Existing Apps that relied on rows tappable through CopyFrom alone must declare links.
- `links.link-only.offered-only-by-links.authoring`: Use LinkOnly for row-only destinations to avoid duplicate Actions entries, while [NextRequest and ConstrainedBy still govern availability](../reference/protocols.md#link-only).

### Link only

Owner: [protocols.md#link-only](../reference/protocols.md#link-only)

- `links.link-only.declaration`: **Link-only requests.** `@LibertasLinkOnly` and `#[libertas_link_only]` take no argument and emit `{"Name":"LinkOnly","Value":"true"}` on a protocol Request variant, or a SubscriptionRequest variant targeted by a stacked link.
- `links.link-only.older-clients`: Clients that predate the marker ignore it and keep offering the request in Actions; the request remains valid there, so adding the marker changes presentation only.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Links

Owner: [protocols.md#links](../reference/protocols.md#links)

- `links.declaration`: A Link targets exactly one Request variant native name of the same protocol; a stacked link may target a SubscriptionRequest
- `links.placement`: A Link goes on any field or on a Table, Union or Enumeration type declaration

### Link return

Owner: [protocols.md#link-return](../reference/protocols.md#link-return)

- `links.return.no-transition-copyto`: A link target NextResponse names no Response declaring CopyTo with NextRequest or WorkflowRequest

### Links

Owner: [protocols.md#links](../reference/protocols.md#links)

- `links.stacked.target-constructible`: A stacked target is LinkOnly and fully constructible with no editable input
- `links.stacked.display-only-reply`: A stacked target declares NextResponse whose non-Error replies are plain display Responses
- `links.studio.next-request-lists-target`: On a displayed occurrence the containing variant NextRequest lists the link target
- `links.studio.return-copyto`: An editor link target plain-reply CopyTo is valid against the originating request; every direct field has CopyTo or is Hidden
- `links.studio.displayable-host`: A link host is visible and its variant is displayable as rows, never an Error or WorkflowRequest Response

### Link only

Owner: [protocols.md#link-only](../reference/protocols.md#link-only)

- `links.link-only.studio`: LinkOnly needs a link targeting it and excludes DefaultRequest and ExcludeUI

<!-- /generated -->
