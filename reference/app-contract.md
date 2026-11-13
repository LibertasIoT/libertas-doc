# Libertas App contract

This is the common authoring contract for Rust and TypeScript Apps.

## Required design principle

<!-- rule: app.design.smallest-complete class=judgment enforced=no tasks=app-design -->
**The goal is the smallest design that remains logically complete.** All App
authors, including CodeBots, must follow this principle. It is a requirement,
not an optional style preference.

<!-- rule: app.design.complete-with-owners class=judgment enforced=no tasks=app-design -->
Implement every required behavior, condition, and outcome, with a clear owner
for each responsibility. Use the platform's defined behavior and normal App
lifecycle instead of duplicating them with additional mechanisms. Add App logic
when the application's requirements need it; simplicity must not omit necessary
validation, permissions, consistency, or user control.

<!-- rule: app.design.natural-data-model class=judgment enforced=no tasks=app-design,data-model -->
**Model data in its natural form.** Declare each stored type as what the data
is: its real fields, one owner per fact, and a Union where a value is one of
several shapes. Never reshape stored data for a screen, a workflow step, or a
question order. Human-facing types, the Request and Response payloads a person
edits and views, are views of that data: they may wrap it as the person's path
needs, for example as a wrapper member within a Union, while the stored model
stays as it is.

<!-- rule: app.design.human-shortest-path class=judgment enforced=no tasks=app-design -->
**The shortest path is the person's.** Human usability is part of what the
smallest design optimizes: the fewest steps, questions, and screens a person
needs to reach a complete, correct answer. A design that is shorter for its
author or a CodeBot never justifies a longer path for the person.

<!-- rule: app.design.no-contradiction class=judgment enforced=no tasks=app-design,data-model -->
**The two never conflict.** The goal is human first, and the natural data model
serves it: the platform presents the model in its natural form along the
person's shortest path, so neither is traded for the other. Where they seem to
conflict, the platform is missing a presentation mechanism; raise that gap
instead of bending the data or lengthening the person's path. Presentation and interaction adapt to the model through the
platform: schema attributes such as `AskOnAdd`, `ReadOnly`, and `Hidden`, and
Display Template rules such as `lock`, `flatten`, `promote`, and layouts. When
the platform has no way to present a natural model, that is a platform gap to
raise, not a reason to bend the data.

For ordinary Apps, persistence failure belongs to the Hub runtime. After storage repair
and a third-party `Enable` action, the App follows ordinary startup logic; it must not introduce a separate
storage-failure recovery system. Follow the
[Data API responsibility boundary](protocols.md#persistence-failure-and-the-no-status-api).

## Ask once, directly

<!-- rule: app.ask.no-gate-question class=judgment enforced=no tasks=app-design -->
A question exists only to collect a fact. Do not add a Boolean, Yes/No or
"something else applies" switch whose only effect is to reveal a follow-up
asking the same thing in detail: the person answers twice, the App needs extra
navigation and state, and the stored switch can disagree with the real answer. <!-- rule: app.ask.detailed-question class=judgment enforced=no tasks=app-design -->
Ask the detailed question itself: a Bitflags or Enumeration list with a "None
of these" member, a List that may be empty, or a required Union whose variants
carry their own data. Keep a separate question only when its answer decides
something the detailed answer cannot.

## AI access in every design step

An App's data and interactions serve people, other software and AI from one
description, so AI access is part of the design rather than an addition to it.
At every design stage (types, pages, rules, messages and tests):

<!-- rule: app.ai.document-pages-import class=judgment enforced=no tasks=app-design -->
- A page whose values come from a document declares
  [document import](llm-document-transformation.md). <!-- rule: app.import.printed-fields class=judgment enforced=no tasks=ai-import --> Every field whose value is
  printed on that document or its attached statements is an import target,
  including fields in sections that appear only conditionally. <!-- rule: app.import.records-not-targets class=judgment enforced=no tasks=ai-import --> A value that
  only the person's own records hold is not an import target; its description
  says where to find it. <!-- rule: app.import.classify-new-fields class=judgment enforced=no tasks=ai-import --> A new field on an importing page is classified when
  it is added. <!-- rule: app.import.preserve-person-data class=hard enforced=no tasks=ai-import --> Nothing an import can set may hold data only the
  person supplies: a composite on an import path is not Nullable, because a
  null result replaces it, and a Union whose variant the import chooses
  carries only document values, because a variant change discards the old
  payload; keep person-supplied values in a record the import does not select.
- [AI Search](protocols.md#ai-navigation-and-ai-search) selects among a page's
  offered links and eligible action choices. Links usually differ by the
  destination their Request carries: an Enumeration (or Union) field that `CopyFrom` fills from the
  tapped row or the response. The
  [authoring obligations](protocols.md#app-and-codebot-authoring-obligations)
  make each destination a bound candidate documented with the identifiers and
  common names a person would type or say, and own `AINavigationPrompt`
  guidance.

## Application model

The [Hub architecture overview](../concepts/hub.md) explains the platform's
resource management and the roles of human users, AI and applications.

A Libertas App is self-describing source code. Public functions define task
entry points. Function arguments define task configuration. Reachable public
types define configuration, persistent data, Endpoint protocols, and runtime
presentation. Studio extracts and validates those declarations and generates
the language-neutral schema and localized documents.

**App is data interaction.** Design the domain data, its meaning, the operations
participants can request, and the behavior that handles those operations.
Configuration, Requests, Responses and SubscriptionData describe that interaction
as typed values. Source code implements the behavior; declaring a protocol does
not execute it. <!-- rule: app.model.no-invented-interaction class=judgment enforced=no tasks=app-design --> Background tasks and TypesOnly packages retain their ordinary
roles and need no invented user interaction.

**Data is the GUI.** Clients present the declared values and operations as
editors, results, choices and actions. The schema defines their structure and
constraints, and source documentation supplies their localized names and
meaning. The [Display Template](presentation.md#display-template) arranges and
styles those values without changing their meaning. <!-- rule: app.model.data-first class=judgment enforced=no tasks=app-design --> Design the data interaction
first, then choose its presentation.

**One interaction serves humans, AI and other Apps.** A person uses generated
controls, AI uses schema-described tools, and another App exchanges typed
Endpoint messages. They use the same declared protocol and App behavior. <!-- rule: app.model.implement-once class=judgment enforced=no tasks=app-design -->
Implement the operation once in the task; the GUI, AI and software interfaces
derive from that common description. A person can use the GUI directly without
AI involvement.

For example, declare a `SetLimit` Request with a typed `limit` field and
[`Write` access](protocols.md#request-access), and a Response reporting the saved
limit. A person edits the generated field, AI supplies the tool argument, and
another App constructs the typed Request. The same task handler checks the
App's conditions, applies the limit and returns the result.

<!-- rule: app.model.access-checks class=hard enforced=no tasks=app-design -->
Each participant remains subject to its applicable access checks. Generated
clients follow the [human interaction lifecycle](protocols.md#human-task-interaction);
software may agree with the service on its own ordering within the declared
protocol. [`ExcludeUI`](protocols.md#declaring-an-endpoint) can limit the GUI and
AI projection while retaining the underlying Endpoint contract.

A package therefore has these parts: source code; the generated schema; the
localized documents; the optional `display.json` at the project root, the
version-controlled source of its presentation rules; and the compiled
[Display Template](presentation.md#display-template), which Studio builds from
the source presentation attributes and `display.json`. <!-- rule: app.template.not-schema class=hard enforced=no tasks=app-design --> The template is
mandatory, but an App with neither compiles to the valid empty template. It is
not part of the schema, and changing only the template never changes the App
version. <!-- rule: app.template.edit-source class=hard enforced=no tasks=app-design --> Edit `display.json` or source attributes, never the compiled template.

<!-- rule: app.model.no-second-schema class=hard enforced=no tasks=app-design -->
Do not create a second JSON schema, manually assign schema IDs, or treat
generated schema JSON as source.

The schema FQN for an owned public type is:

```text
developer-package::TypeName
```

<!-- rule: app.identity.package-name class=hard enforced=yes tasks=app-design title="An App identity has exactly one hyphen joining two lowercase ASCII snake_case names without edge or repeated underscores" -->
The App identity contains exactly one hyphen. Both sides are snake_case made of
lowercase ASCII letters, digits, and single underscores; neither side may have
leading, trailing, or consecutive underscores. For example,
`acme-temperature_monitor` is valid and `acme-temperature-monitor` is not. The
developer and package identify ownership. <!-- /rule --> [Importing or re-exporting a type from
another App does not transfer ownership](schema.md#32-foreign-wire-types).

## Project and discovery rules

<!-- rule: app.discovery.entry-functions class=hard enforced=no tasks=app-design -->
Declare task entry functions in `src/lib.rs` for Rust or Studio's selected
entry module for TypeScript. Other implementation can live in ordinary modules.
Function re-exports do not replace these declarations. The
[code organization practice](code-organization.md) explains module boundaries
and exact-name type exports.

<!-- rule: app.discovery.ordinary-roots class=hard enforced=no tasks=app-design -->
An ordinary App uses every non-ignored public/exported function as a root. Its
published type set is the minimum transitive closure needed by:

- function arguments;
- function persistent-data schemas;
- Endpoint protocol schemas;
- inherited and nested type references; and
- schema attributes that reference types, including Bitflags and compatible
  foreign types.

<!-- rule: app.discovery.unreachable-unpublished class=hard enforced=no tasks=app-design -->
An unreachable public type is not published by an ordinary App.

A [`TypesOnly` package](schema.md#8-types-only-and-interoperable-packages) publishes no executable functions. It automatically roots
every public package-owned named type exported from the language entry module,
unless the declaration is explicitly ignored. <!-- rule: app.discovery.types-only-helpers class=hard enforced=yes tasks=app-design title="A public helper function in a TypesOnly package must be ignored" --> A public helper function in a
`TypesOnly` package must also be ignored. <!-- /rule --> If an included type reaches an [ignored
type](schema.md#8-types-only-and-interoperable-packages), generation fails rather than emitting a dangling reference.

Every published named type must be [publicly exported from the entry module
under its exact declaration name](schema.md#31-local-named-types). Renamed root exports and duplicate flattened
names are invalid.

## Schema compilation with Codex

Studio provides the `compile_schema` tool for saved Rust and TypeScript App
source. <!-- rule: app.compile.edit-validate-loop class=judgment enforced=no tasks=app-design --> After changing public functions, types, attributes, or dependencies,
save the files, compile the affected project, fix the reported source errors,
and repeat until validation succeeds. <!-- rule: app.compile.edit-source class=hard enforced=no tasks=app-design --> Edit source declarations rather than
generated schemas. Continue to run language builds and tests separately.

<!-- rule: app.compile.project-directory class=hard enforced=yes tasks=app-design title="Supply the absolute project directory to compile_schema; an optional entry selects the entry file" -->
Supply the absolute project directory; an optional `entry` selects the entry
file. The default result contains diagnostics, source locations when known,
the schema revision, and a dependency summary. Set `includeArtifacts: true`
to receive the root schema, default document, and compiled Display Template
plus every transitive App dependency's schema, default document, Display
Template, revision, and dependency edges. Compilation validates `display.json`
with the schema; its errors are ordinary diagnostics naming `display.json` and
the one-based line when known. Each
dependency appears once. <!-- rule: app.compile.dependencies-complete class=hard enforced=yes tasks=app-design title="Missing, ambiguous or cyclic App dependencies fail compilation; a partial bundle is not a successful result" --> Missing, ambiguous, or cyclic App dependencies fail
compilation; a partial bundle is not a successful result.

<!-- rule: app.compile.saved-files-only class=hard enforced=no tasks=app-design -->
Compilation uses saved files and installed or locally resolvable dependencies.
It does not read unsaved editor buffers, install missing dependencies, overwrite
source or generated files, or deploy. No Hub connection is required. <!-- rule: app.compile.success-scope class=hard enforced=no tasks=app-design --> Success
means source schema validation passed; it does not establish exact deployed
dependency-version bindings or replace Studio's deployment checks.

The tool's instructions tell Codex and Claude Code to follow this
edit-and-validate workflow. See [Studio setup](../guides/studio.md#using-studio-mcp-with-codex-or-claude-code)
for registration and an example prompt. Deployment uses the separate
[MCP deployment contract](deployment.md#mcp-deployment).

## Automated QA through Studio MCP

Studio's QA tools use the registered Hub's saved login and CA pin. Keep VS Code
and Studio running. The login needs its ordinary task administration permission
for the App/task inventory and lifecycle; endpoint access keeps its ordinary
permissions. No credentials or internal Hub messages are returned or accepted.

1. Call `qa_session_open(hubId)` using an exact ID from `list_deployment_hubs`.
   The result contains `session`, login facts, installed Apps and their exact
   versions, task functions and their parameter schema identities, and visible
   tasks. `qa_session_list(session)` refreshes the inventory; add
   `includeArtifacts: true` when you need parameter definitions and types.
2. Call `qa_task_endpoints(session, task)` for an exact listed task ID. It returns
   send targets, protocol types, schema identities and Display Template digests;
   add `includeArtifacts: true` only when you need each schema definition and
   default document, which can be megabytes.
3. Call `qa_send` with `session`, a returned `endpoint`, and the role fields:
   `Request` and `SubscriptionRequest` take `variant` and the App schema JSON
   payload in `value`. `SubscriptionRelease` takes neither; it releases the
   endpoint's subscriptions. `CalculationRequest` takes the calculation
   `variant`, the bound current `request: {variant, value}`, and its bound
   `root` occurrence as a Libertas Schema Path. Studio constructs that invocation
   through its ordinary [calculation owner](protocols.md#calculation-root).
   Invalid roles, values and unqueried targets
   are refused. Supplied Request values receive no defaults or replacement
   answers. A send returns Studio's assigned `requestId` when the operation
   has one.
4. Consume `qa_receive(session, timeoutMs, max?)` events, oldest first. Each event
   contains `endpoint`, `role`, `value`, `sequence`, `arrivedAt` and the ordinary
   `requestId` when correlated to a send. Roles include Response, EditRequest,
   SubscriptionData, CalculationResult and Error. An empty queue waits up to
   `timeoutMs`; `max` defaults to 100.
5. Call `qa_session_close(session)` when finished. Studio exit also closes it.

<!-- rule: app.qa.import class=hard enforced=no tasks=client-runtime -->
`qa_import(session, endpoint, field, files)` runs the current Request's ordinary
[AI Import](llm-document-transformation.md) action at its `field` path. Plain
hosts use their document plan and typed result application, including normal
List-item array import and duplicate handling. Classifier hosts retain the
categorized review's default selection. The tool uses the Hub's configured
provider, returns imported values and the complete updated Request, and leaves
that Request open without submitting. Each file supplies `sourceMimeType`,
normalized `mimeType`, `contentBase64` and optional `name`: unchanged PDF bytes
or prepared JPEG images. The host's normal file-count, MIME and size limits
apply. Replaced editors, changed facts and revoked AI availability discard a
pending result under the ordinary import lifetime. Failed document rows retain their extraction error in the reply and import summary; unclassified pages remain identified as not classified.

Open begins ordinary endpoint discovery and tracking admission for listed tasks;
it returns the session before snapshots can fill the queue. Incoming events are
queued even before the first receive, including default subscription snapshots.
A per-task discovery refusal is reported by an explicit endpoint query and in
`endpointErrors` on the refreshed session listing; it does not grant access. <!-- rule: app.qa.lossless-events class=hard enforced=yes tasks=app-design title="Consume every queued QA event; capacity applies back-pressure rather than dropping events" --> Events are never
filtered, merged or dropped. The bounded queue applies transport back-pressure
at capacity; the next receive reports `overflow: {code: "queue_overflow"}` and
consumption permits reading to resume. Sequence numbers are contiguous per
session. Subscription requests and releases follow the ordinary
[subscription lifecycle](protocols.md#protocol-roles); reconnect means closing
and opening a session. Subscription demand is runtime-only.

`qa_task_create(session, app, function, name, parameters, enable)` creates a task
using an exact listed App ID and task function. <!-- rule: app.qa.task-create class=hard enforced=yes tasks=app-design title="Choose a listed task function and validate its supplied configuration with ordinary editor defaults" --> Studio applies its ordinary
editor defaults and single-candidate fill, then refuses a missing or invalid
value with its field path. It never chooses a function or invents an answer.
`enable` selects whether the task is enabled after creation.
`parameters` is an object keyed by the function parameter's native name. Values
use the App schema's JSON; at a new VirtualDevice or Endpoint field,
supply `{id, name, icon}` with its negative signed 64-bit working ID. Studio
preserves that ID and its name/icon association. Existing identities remain
scalar IDs.
`qa_task_upgrade(session, task, version)` selects an exact installed version ID
of that task's App and follows its normal upgrade lifecycle. Refresh and query
its endpoints afterward. No task-removal tool exists.

An optional absolute `journalPath` on open creates a new private file containing
only decoded endpoint events. <!-- rule: app.qa.journal-exclusive-create class=hard enforced=yes tasks=app-design title="A QA journal is a new private file of decoded endpoint events, with no saved credentials" --> Existing files are never overwritten. JSON uses
Booleans and Numbers, decimal strings for integer-typed values beyond JavaScript's
exact numeric range and for identity values, and base64 strings for Bytes
(byte arrays are also accepted on input). Union values have one declared
variant key; a payload-less variant has a null payload.

### Editor tools

The editor tools run Studio's normal request construction, reference context,
Display Template, calculations, validation and draft lifecycle. They take the
open QA `session`. Open returns an `editor`; later calls use that ID and return
its current `view`. The editor follows replies that replace its page.

| Tool | Inputs after `session` | Behavior |
| --- | --- | --- |
| `qa_editor_open` | `endpoint`, `from` | Construct a first Request from its variant name; use `{event: sequence}` for the currently owning EditRequest or Response carrying WorkflowRequest; or choose an offered action with `{choice, path?}`. |
| `qa_editor_view` | `editor`, optional `image: {form, part?}` | Read current rows, groups, choices, validation, actions and pending state. `form` is `phone`, `tablet` or `desktop`; PNG rendering is optional and shows the whole page, in numbered parts when it is long. |
| `qa_editor_set` | `editor`, `path`, `value` | Edit a visible, editable field with schema JSON. Union selection also accepts `{variant: name}`; null answers an outer Nullable choice, while `{Branch: null}` selects a nullable payload; `{set: true}` opens an absent Nullable composite. |
| `qa_editor_add`, `qa_editor_remove` | `editor`, `path`; Add also takes optional `kind` | Use ordinary List membership controls and the provisional Add questions. `kind` answers an offered Add choice. |
| `qa_editor_commit`, `qa_editor_cancel` | `editor`, open item `path` | Finish or leave that item under the normal item-editor rules. Cancel discards a provisional addition; already committed edits to an existing item remain. <!-- rule: app.qa.editor-local-items class=hard enforced=no tasks=client-runtime --> These calls do not submit the parent Request. |
| `qa_editor_import` | `editor`, host `path`, `files` | Run the offered [categorized import](llm-document-transformation.md#categorized-import) with the Hub's provider and ordinary default review selection. Returns the import result and updated view. |
| `qa_editor_action` | `editor`, `action` | Perform an offered action with its exact `name`, `path` and `choice`. Includes Submit, PrevRequest, GoTo, hierarchy `SelectChoice`, folds, tabs, summary navigation, draft answers and import Cancel. `{search: text}` runs the offered AI Search. |
| `qa_editor_close` | `editor` | Close the editor and release its presentation demand. Returns `closed: true`; session close closes every editor. |

Paths are [Libertas Schema Paths](schema.md#libertas-schema-path) from the returned
view. Rows retain their schema identity through promotion, flattening and table
projection. They include schema JSON values and available chooser models;
`displayValue`, when present, carries resolved display text. Hidden subtrees are
withheld. Read-only fields, disabled actions and stale editors are refused.
Mutating calls also accept the returned `revision` to refuse a stale view.
Pass an offered `choice` unchanged, including hierarchical and group choices.
The logical root row retains the screen's schema identity; `presentation`
identifies its full Request, compact single-field, hierarchical chooser, item
or Response body. Compact and hierarchical bodies follow the ordinary controls.
Selecting an image `form` also selects that form for the returned view and later
operations; a form change advances the revision without changing values.

<!-- rule: app.qa.editor-confirmation class=hard enforced=no tasks=client-runtime -->
`qa_editor_action` with Submit or PrevRequest returns `confirmation` when the
Request requires it. Answer the same action with `confirm: true` to send, or
`confirm: false` to cancel. Cancellation preserves the current surface under
the [ordinary confirmation rules](protocols.md#request-confirmation). <!-- rule: app.qa.editor-immediate-links class=hard enforced=no tasks=client-runtime --> Choosing
an input-free GoTo or Link follows ordinary immediate sending, with confirmation
when required. Stacked inspection pages, local Close and transient composers
follow the [Link lifetime](protocols.md#links).

An import file supplies `sourceMimeType`, normalized `mimeType`, `contentBase64`
and optional `name`; normalized input is PDF or JPEG. A pending import reports
`documentImport` and offers `CancelImport`. Saved-draft questions offer Restore,
Discard and Dismiss through the returned actions. Both states block ordinary
editing until answered. The raw `qa_import` tool remains available with
`session`, `endpoint`, host `field` and the same files.

Calculations are reported, rather than silently waited for. Consume their events
through `qa_receive`, then read the editor again. Editor sends carry
`direction: "sent"` in the same event stream; every incoming event remains
queued even when it no longer owns a displayed page. Raw `qa_send` values keep
their existing exact-value behavior.

A requested PNG uses the real Control renderer and retained presentation state;
it does not send a Request. <!-- rule: app.qa.editor-png-browser class=hard enforced=no tasks=client-runtime --> It requires an available Chromium installation.
The image covers the whole page at the form's width, including the part a
scrolling editor would hide, not only the first screen. A page taller than
2400 px is split into numbered parts of at most that height. The MCP result
contains JSON metadata and the view plus one image block per returned part, in
order: up to four parts and about 3 MB per call. `image` in the metadata reports
the whole `width` and `height`, the number of `parts`, the first returned `part`
and how many were `returned`. Pass `image.part` to fetch one numbered part, for
example the next after those returned. A page taller than 16000 px is cut there.
If the page changes while rendering, read it again rather than using a stale
image. Capture failures are reported without changing the editor's values.

## Type model

<!-- rule: app.types.closest-native class=judgment enforced=no tasks=app-design -->
Use the closest native type. Do not encode typed values as ad-hoc strings.

| Libertas meaning | TypeScript | Rust |
| --- | --- | --- |
| Boolean | `boolean` | `bool` |
| <!-- rule: app.types.integer-width class=hard enforced=no tasks=app-design --> Signed/unsigned integer | `number` plus `@LibertasNumber(type=...)` | concrete `i*`/`u*` |
| Floating point | `number`, optionally `@LibertasNumber(type=...)` | `f32`/`f64` |
| String | `string` | `String` |
| Bytes | `Uint8Array` | `Vec<u8>` |
| List | `T[]` | `Vec<T>` |
| Nullable | optional/nullable supported type | `Option<T>` |
| Table | interface/class/object type | named-field struct |
| Enumeration | TypeScript enum | payload-free enum or enum attribute |
| Union | discriminated union | enum with variants |
| Date/time | `LibertasDateTime`, `LibertasTimeOnly` | SDK equivalents |
| Platform object | Libertas global object type | SDK object type |
| Endpoint | `LibertasEndpoint` | `LibertasEndpoint` |

<!-- rule: app.types.date-time-encoding class=hard enforced=no tasks=app-design -->
`DateOnly` is an integer `yyyyMMdd` calendar tuple, not an epoch timestamp.
`TimeOnly` is seconds from midnight. `DateTime` is epoch seconds presented in
the viewer's local timezone.

### User-facing and non-user-facing types

Choose a representation for the value's meaning and its consumers. A public,
schema-reachable declaration is not necessarily user-facing: it may describe
persistent records or internal calculations. <!-- rule: app.types.review-occurrences class=judgment enforced=no tasks=app-design --> Review each occurrence of a type,
including its editability and visibility, rather than classifying it by `pub`
or `export` alone.

| Use | Design question |
| --- | --- |
| User input, including task configuration | What must the person enter or decide? Which answers may start blank, absent, or unselected? |
| User-visible output | What does the App report? How does an unavailable result differ from an empty result or zero? |
| Non-user-facing storage, calculation, or transport | Which states must the App preserve, and which distinctions affect its logic or compatibility? |

<!-- rule: app.types.separate-input-internal class=judgment enforced=no tasks=app-design -->
Reuse a type when these meanings and constraints agree. Use separate input and
internal types when they differ, and map between them explicitly. For example,
an editor can use blank text while a stored record uses absence. This does not
justify copying every type into a second hierarchy.

For user input, choose among these representations:

| Meaning | Recommended representation |
| --- | --- |
| <!-- rule: app.types.input-optional-text class=hard enforced=no tasks=app-design --> Ordinary optional text, where blank means not supplied | String with `Default=""`, `SizeMin=0`, and a suitable maximum. |
| <!-- rule: app.types.input-required-text class=hard enforced=no tasks=app-design --> Required text | String with `SizeMin` of at least 1; omit a Default when the person must supply the answer. |
| <!-- rule: app.types.input-optional-override class=judgment enforced=no tasks=app-design --> An optional override of a value the App computes or receives, or harmless supplementary detail | Nullable (`Option<T>` in Rust), with null meaning “use the computed or received value” or “not given”, stated in its `[NullText]`. For output and internal data, Nullable also marks an unavailable value as distinct from zero. |
| <!-- rule: design.nullable.explicit-domain-choice class=hard enforced=no tasks=app-design --> A yes-or-no decision whose “yes” carries details, or an optional override | Nullable labelled with `[NullText]` and `[SetText]`; no Default when the person must answer, `$null` when skipping means null. See [Nullable choices](#nullable-choices). |
| <!-- rule: app.types.input-optional-min-length class=hard enforced=no tasks=app-design --> Optional text that must meet a minimum length when supplied | Nullable String with the appropriate size bounds; null remains valid. |
| <!-- rule: app.types.input-named-alternatives class=judgment enforced=no tasks=app-design --> A decision between named alternatives, including “none” and “provide details” | A labeled Enumeration for choices without payloads, or a labeled Union when a choice carries additional input. |
| <!-- rule: app.types.input-folding class=judgment enforced=no tasks=app-design --> Details that only need to expand or collapse | A Table with [folding](presentation.md#folding-structs-and-unions). Folding changes presentation without changing the answer. |

#### Rows a person adds

A member of an editable List is a row a person can add, starting from type
initialization with no service values. <!-- rule: app.rows.hidden-needs-value class=hard enforced=yes tasks=app-design title="Every Hidden field in a row a person can add needs a Default, Fixed or CopyFrom" --> Give every Hidden field in such a row a
`Default`, `Fixed` or `CopyFrom`, since nobody can answer it; an undefined one
blocks submission of the new row. <!-- rule: app.rows.service-rows-readonly class=judgment enforced=no tasks=app-design --> When the service creates the rows and the
person only edits them, mark the List `ReadOnly`, which fixes its membership
while its members stay editable ([ReadOnly Lists](presentation.md#common-field-metadata)). <!-- /rule -->
Studio warns about a Hidden required field in a row a person can add.

<a id="default-or-no-default"></a>

### Default or no Default

<!-- rule: app.default.decided class=hard enforced=no tasks=app-design,data-model,source-documentation -->
Every field a person answers gets an explicit, reasoned decision, never an
accident of its source type or a copied pattern. **No `Default`** when the
person must answer: the field stays
[undefined](schema.md#undefined-and-its-sources) and blocks Submit until
answered, on a client-built request
([EditRequest or WorkflowRequest](protocols.md#strict-questions-and-data-pages)).
**A `Default`** only when that value is the answer the person would give
without reading, such as a usual value or `$null` where skipping lawfully means
null. Never default an answer that has consequences, such as eligibility, an
election, consent or a fact the App's rules test: a Default is never the
person's confirmation. Decide again whenever the field's question, type or
page changes. The rows below apply this decision to each kind of field.

### Nullable choices

**A Nullable (optional) value is a two-state Union:** one null state and one
state holding a value of its type, each with default label text unless the
field document names it (`[NullText]`, default “None”; `[SetText]`, default
“Specify”).

An editor presents an editable Nullable as one question with two named
answers ([Nullable choices](presentation.md#nullable-choices)), each client
using its best-practice control for the value type: a null answer labelled by
the field's `[NullText]` and a value answer labelled by its `[SetText]`, which
opens the field's ordinary editor. An editable Nullable must declare
`[NullText]`, and `[SetText]` too wherever the control draws a value answer
([standard properties](documentation.md#standard-properties))
(a Number, String, Bytes, date or time, Table or List; a Boolean's and a picker's value answers are their
own members), written as the two answers the person chooses between. <!-- rule: app.nullable.label-wording class=judgment enforced=no tasks=app-design -->
`NullText` is a complete answer; `SetText` introduces the value entered right
after it, so it reads as a lead-in to that value. For example: “Alive at the
end of 2026” and “Died in 2026 on” followed by the date; “Use the payer's
basis” and “My own basis is” followed by the amount; “Not a real estate
professional” and “Real estate professional” followed by the qualifying
details. <!-- rule: app.nullable.fallback-labels class=judgment enforced=no tasks=app-design --> The defaults “None” and “Specify” are only a client fallback; they
never explain the decision.

Decide whether skipping the question is a lawful answer:

<!-- rule: app.nullable.must-answer class=hard enforced=no tasks=app-design -->
- If the person must answer, give no Default. The choice then starts with no
  branch selected and blocks Submit until the person chooses, exactly like a
  Union without a Default. <!-- rule: app.nullable.skip-means-null class=hard enforced=no tasks=app-design -->
- If skipping means null, such as an optional override of a value the App
  computes or receives, or optional detail whose omission decides nothing,
  give the reserved Default [`$null`](schema.md#reserved-default-values)
  (Rust `#[libertas_default($null)]`). The choice starts on the null branch,
  and its `[NullText]` says what null means. <!-- rule: app.nullable.usual-value-default class=judgment enforced=no tasks=app-design -->
- If one value is the usual answer, give that value as the Default; the choice
  starts on the value branch with it.

<!-- rule: app.nullable.three-answers class=judgment enforced=no tasks=app-design -->
Use a labelled Enumeration or Union instead when there are three or more
answers, or when the negative answer carries data of its own. For output and
internal data, which no person answers, Nullable remains the normal way to
state that a value is absent, with `[NullText]` naming any meaning other than None. <!-- rule: app.nullable.native-names class=judgment enforced=no tasks=app-design --> Native `None`/`Some` or `Nothing`/`Something`
names never explain a decision; this is not the gate
[Ask once, directly](#ask-once-directly) forbids, because the value branch
carries the details itself and the person answers once.

<!-- rule: app.nullable.option-string-min class=hard enforced=no tasks=app-design -->
An `Option<String>` with `SizeMin=1` is valid: null is allowed, but a present
empty String is rejected. It therefore differs from a non-Nullable String with
`SizeMin=0`. Use the latter for ordinary optional text when that distinction
serves no purpose. Size bounds do not supply initial values. <!-- rule: app.nullable.unanswered-start class=hard enforced=no tasks=app-design --> Under the
[initialization contract](schema.md#shared-copyfrom-and-copyto-rules), when no
Fixed, Schema Default, or localized default supplies a value, a String starts
unanswered even if it permits empty text, an editable Nullable starts with no
branch selected, and a Union starts unselected. <!-- rule: app.nullable.default-not-confirmed class=judgment enforced=no tasks=app-design --> A Default selecting “none” does not establish
that the person confirmed that choice.

### Reviewing a representation change

<!-- rule: app.representation.review-uses class=judgment enforced=no tasks=app-design -->
When changing Nullable text to blank text, review presence checks, validation
rules, label fallbacks, calculations, and grouping keys. Translate `""` to
absence only where the destination uses that meaning, and translate absence
back to `""` for that editor. Do not accidentally group unrelated unnamed
records together. Keep unknown amounts distinct from zero.

<!-- rule: app.representation.update-docs-import class=judgment enforced=no tasks=app-design -->
Update [source comments](documentation.md#parameter-and-field) and
[AI Import declarations and instructions](llm-document-transformation.md#blank-values-and-output-types)
to match the new blank semantics: making manual entry optional does not enable
blank import results, which follow the
[optional-text import rule](llm-document-transformation.md#declared-blank-value).
Inspect the generated schema and document, and validate the affected entry and
import behavior.

<!-- rule: app.representation.compatibility class=judgment enforced=no tasks=app-design -->
Review published protocol and persistent-data compatibility before changing
any representation. Replacing Nullable with a String or custom Union is not
automatically compatible. Follow the
[data-type evolution contract](data-type-evolution.md); changing an editor does
not require changing an existing stored representation.

## External types and App dependencies

Resolve an external type through the [native language import/module system](schema.md#32-foreign-wire-types). The
parser converts the resolved declaration to the owning App FQN. It never guesses
an unbound plain name by scanning dependencies.

The external package must itself be a Libertas App and must [export the exact
type from its entry module](schema.md#32-foreign-wire-types). Ordinary npm/Cargo library types cannot become
Libertas schema references.

Use [`ForeignType`](schema.md#32-foreign-wire-types) only when this App publishes its own wire-compatible named
type as a direct representation of one official external type. The compatible
type retains its own FQN; it does not become the official type. <!-- rule: app.external.foreign-type-metadata class=hard enforced=no tasks=app-design --> The official
type supplies inherited schema/document metadata when consumers materialize
the graph, while explicit compatible attributes override same-name official
attributes.

<!-- rule: app.external.no-authored-dependencies class=hard enforced=no tasks=app-design -->
Do not write versioned `AppDependencies` in source or edit generated locks.
Studio discovers and verifies dependencies, deploys them first, and records the
actual exact versions. The [deployment dependency contract](deployment.md)
defines discovery, including which Endpoint constraints create a dependency,
Plaza preflight, installation, failure handling, and the exact
[`Version` pin](deployment.md#schema-availability-and-task-requirements).

## Functions, permissions, and singleton tasks

Public functions are task templates. Their arguments are configuration values.
A function may declare:

- a persistent-data Union;
- required process permissions and [same-key default-locale reasons](protocols.md#function-and-endpoint-permissions);
- required concrete network hostnames or reserved local/external grants and
  same-key default-locale access reasons;
- semantic validation expressions, each paired with a quoted package
  [`StringResources` key](validation-rules.md#3-rule-set) whose localized value is the user-facing error;
- package-owned string-resource keys used by the function; and <!-- rule: app.function.singleton class=hard enforced=no tasks=app-design -->
- `Singleton`, limiting the Hub to at most one task for the App and native
  function name across versions.

<!-- rule: tasks.creation.dependency-admission class=hard enforced=no tasks=client-runtime -->
Before creating a task, Android and iOS first fail fast on a `Singleton`
conflict, then queue the requested creation while statically checking required
task-process dependencies. They traverse the complete effective input schema.
A client Endpoint carrying `Default` or `Fixed` is absolutely required when
neither it nor an ancestor is `Nullable` and no ancestor List permits zero
items. Every such Endpoint needs a currently compatible server; `Fixed` also
requires the exact named task-process identity. <!-- rule: app.task-creation.install-required-tasks class=hard enforced=no tasks=client-runtime --> Missing servers open the
required-task installation UI, and the queued creation proceeds only after a
fresh complete check finds every dependency satisfied. Cancelling any required
dependency installation cancels the originating task creation. See the
[task pre-creation dependency check](schema.md#62-task-pre-creation-static-dependency-check).

Permissions grant the task process access to platform data/resources. <!-- rule: app.permissions.approval-view class=hard enforced=no tasks=client-runtime -->
The client approval view always shows the raw permission key as its current
summary and the requesting App's same-key localized `StringResources` value as
a separate explanation. <!-- rule: app.permissions.package-owned-reasons class=judgment enforced=no tasks=app-design --> Different Apps can request the same permission for
different reasons, so explanations are package-owned rather than global. <!-- /rule -->
`AccessHost` declares the task process's network access: concrete DNS
hostnames or the reserved `{network_access_local}` /
`{network_access_external}` broad grants. The public
[App network access contract](network-access.md) owns its default denial,
syntax, reasons, approval view and runtime enforcement.
[`AccessPrivilege`](protocols.md#request-access) is different: it limits which endpoint users may invoke one
runtime Request or SubscriptionRequest.

## Persistence responsibility

By default, Data API storage failures require the Hub runtime to stop the task rather than
returning a recoverable App status. See the
[persistence responsibility and restart rationale](protocols.md#persistence-failure-and-the-no-status-api).

A function may opt into checked Data API results using `DataErrorStatus`; see the
[Data API owner](protocols.md#explicit-error-status-opt-in). This exception transfers
consistency handling to that App without adding Hub-automatic restart.

## Task version selection

<!-- rule: app.version.selection-unchanged class=hard enforced=no tasks=client-runtime -->
A task runs the App version selected for it. Installing or upgrading an App
never changes any task's selection. <!-- rule: app.version.change-permission class=hard enforced=no tasks=client-runtime --> A person with configuration permission on
the task changes it through the task's settings; selecting a version of
another App retargets the task and also requires configuration permission on
that App, as task creation does. <!-- rule: app.version.change-keeps-state class=hard enforced=no tasks=client-runtime --> The Hub then replaces the task's runtime with
the selected version, keeping its arguments and enabled state. When the
arguments cannot be initialized under the selected version, the change fails
and the task keeps its version. <!-- rule: app.version.directional-evolution class=hard enforced=no tasks=app-design --> Which version reads the data another wrote
follows the [directional evolution rules](data-type-evolution.md).

## Task removal

On Hubs supporting removal preflight, Android and iOS ask the Hub to identify
tasks affected by removing a task's virtual devices, Endpoint servers, or
on/off device. This includes indirect effects when configuration cleanup
removes another task's outputs. Disabled tasks are included.

<!-- rule: app.removal.permissions class=hard enforced=no tasks=client-runtime -->
The user needs removal permission on the selected task and effective
configuration permission on every affected dependent task. Dependencies alone
produce a localized warning listing the affected tasks, with a choice to
cancel or continue. Missing configuration permission produces a localized
failure identifying the tasks that need permission; removal cannot continue.
The Hub's object-removal handling performs the actual configuration cleanup.

<!-- rule: app.removal.dependents class=hard enforced=no tasks=client-runtime -->
Removing a task removes its persistent data. The confirmation lists every
affected task together with the effect on it. An argument of another task that
bound an Endpoint of the removed task is set to null. The Hub then validates
that task's arguments: when they are still valid, the task is restarted
automatically with the new arguments; otherwise it is disabled with the new
arguments.

<!-- rule: tasks.removal.repeat-authorization class=hard enforced=no tasks=client-runtime -->
The Hub repeats the same analysis immediately before executing removal. An
earlier successful probe cannot bypass a later permission change. When the
repeated check fails, including when the Hub cannot inspect the complete task
configuration, the Hub reports the failure and does not begin removal.

<!-- rule: app.removal.atomic class=hard enforced=no tasks=client-runtime -->
Removal is atomic. The Hub applies it in the order of the dependency map: the
removed task and its data, then each dependent task's nulled arguments and its
restart or disabling. Either the whole removal takes effect or none of it does;
no person or task observes a partial state, and a removal interrupted by a
power loss is completed before the Hub resumes ordinary operation. Older Hubs retain their supported removal operation;
the clients warn that dependency and permission preflight requires a Hub
update.

## Documentation and localization

Source documentation creates the default-locale package document:

<!-- rule: app.docs.first-line-label class=hard enforced=no tasks=app-design -->
- the first documentation line is the display name; <!-- rule: app.docs.description-lines class=hard enforced=no tasks=app-design -->
- remaining ordinary lines are the description;
- custom properties use `[PropertyName]` followed by their value;
- [`[DefaultText]`](documentation.md#standard-properties) supplies a non-empty, one-line localized initial String value,
  Endpoint server name, or function task name; unlike schema `Default`, it may
  differ by locale; omit it when it exactly matches the ordinary fallback;
- [`[NullText]` and `[SetText]`](documentation.md#standard-properties) supply non-empty, one-line localized labels for
  the null and value branches of a nullable field's
  [Nullable choice](presentation.md#nullable-choices); an editable Nullable
  declares them as [Nullable choices](#nullable-choices) requires, even when
  “None” is the intended text;
  a client falls back to localized None and Specify only when a label is missing; <!-- rule: app.docs.persisted-text-wins class=hard enforced=no tasks=app-design -->
- persisted or user-assigned text, including an empty string, is immutable to
  programmatic defaulting; these defaults apply only while creating a value
  for which no persisted value exists; <!-- rule: app.docs.list-divider class=hard enforced=no tasks=app-design -->
- list documentation may contain a line of at least four hyphens; content above
  the divider describes the list and content below describes each item; and <!-- rule: app.docs.divider-before-properties class=hard enforced=no tasks=app-design -->
- the array divider must appear before custom document properties belonging to
  the list item.

<!-- rule: app.docs.plain-comments class=hard enforced=no tasks=app-design -->
Plain code comments are not localized schema documentation.

<!-- rule: app.docs.every-node-labelled class=hard enforced=no tasks=app-design -->
Every authored package, function, named type, parameter, field, List container
and item, Enumeration member, Union variant, and inline value must have a
non-empty natural label and description that make sense without source names or
implementation vocabulary. Custom document values and package string resources
follow the same user-language and accessibility rules. The complete normative
contract is the
[schema documentation invariants](documentation.md).

<!-- rule: app.docs.function-comments class=judgment enforced=no tasks=app-design -->
Every public function comment must additionally satisfy the
[developer function-comment invariants](function-comments.md). Its
description is a complete, actor-explicit account of the function's purpose,
user choices and actions, automatic behavior, outputs, and material limits; it
must not rely on generated type text to fill semantic gaps.

<!-- rule: lmf.resources.stable-keys class=hard enforced=no tasks=messages-lmf1 -->
String-resource keys are stable identifiers and are never translated. Values
are translated as LMF1, preserving the structure the
[message-format contract](message-format.md#lmf1-argument-syntax) defines.

Persistent records use the function's declared binary key union. Human-readable
names may use localized formatted values without changing record identity.
Follow the [transactional Data API](protocols.md#transactional-key-value-api) and
its append-only key and value schema rules.

## Wire and compatibility discipline

- Protocol and persistent-data schema versions must remain backward compatible.
  Keep each such Union's native name versionless and invariant for the App's
  lifetime; a new App version appends new payload variants to that same Union
  instead of declaring `ProtocolV2` or `StoredDataV2`. See the
  [App data-type design guidelines](data-type-evolution.md). <!-- rule: app.wire.attributes-keep-wire-type class=hard enforced=no tasks=app-design -->
- Attributes such as `Format`, `PhysicalUnit`, `DateOnly`, `Bitflags`,
  `ValidationRules`, chart metadata, `ReadOnly`, `Hidden`, and `ExcludeUI` affect
  validation/presentation without changing the underlying wire type.
  `ExcludeUI` is specifically a Control-plane projection: the Hub omits a
  marked server Endpoint from its Control UI-device list and generated clients
  omit marked request operations. It never removes nodes from admin creation,
  editing, approval, persistence, routing, or authorization. See the
  [Endpoint declaration contract](protocols.md#declaring-an-endpoint). <!-- rule: app.wire.union-order class=hard enforced=no tasks=app-design -->
- Union declaration order is significant. `variant_index!` uses declaration
  order and append-only protocol/data Union evolution depends on it. <!-- rule: app.wire.append-only class=hard enforced=no tasks=app-design -->
- Append new variants for compatible Union evolution; do not insert, remove,
  reorder, or change existing payload shapes in a published append-only line.
- A protocol may declare conformance separately from its optional variants.
  Configuration checks mandatory client/server compatibility. A client uses
  runtime variant indices to decide whether the selected server supports an
  optional request before sending it
  ([capabilities](protocols.md#required-and-optional-capabilities)). <!-- rule: app.wire.plaza-release-immutable class=hard enforced=no tasks=deployment-publication -->
- The common schema/dependency lock and each existing target's exact App version,
  binary, language, TypesOnly state, and distribution dependencies are immutable
  within one Plaza release. Changing any of them
  creates a new content-derived release; adding or removing targets and changing
  documents or allowed manifest metadata synchronizes the current release. App
  source never chooses the release tag or public version. See the
  [Plaza publication contract](publication.md#app-publication-and-version-identity).

## Completion checklist

- The project is a valid language library/module and typechecks.
- All published types are exact-name root exports.
- Every external plain name resolves through native imports.
- Function roots and `TypesOnly` roots are intentional.
- Every `FormattedText` use satisfies the [static read-only placement and
  request-value contract](presentation.md#static-read-only-placement-and-request-values).
- Mutating endpoint operations explicitly require [`Write` access](protocols.md#request-access).
- Permission reasons and string resources are complete.
- [Defaults, fixed values, bounds](schema.md#9-static-and-runtime-validation), [bit masks](presentation.md#bitflags), and [dates](validation-rules.md#141-dateonly-literals) satisfy their host types.
- Schema-visible Union order and wire-shape changes are intentional.
- No generated schema/document JSON was edited manually.
