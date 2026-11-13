# Designing an App

Load this card for every App-authoring task. It holds the rules of the App
contract that apply to any App: the smallest complete design, asking once, the
data-first model, types and Nullable choices, discovery and dependencies,
function permissions, the task lifecycle, documentation, wire compatibility
and Studio's compile loop. Its rules are generated from the
[App contract](../reference/app-contract.md), the
[MCP tools](../reference/deployment.md#mcp-deployment) and the
[shared reference](../reference/index.md), which own them.

Load the card for each feature the App uses as well:
[AI Import](ai_import.md) for a page whose values come from a document,
[messages](messages_lmf1.md) for `StringResources` templates and formatted
values, [network access](network_access.md) for `AccessHost`, the
[Endpoint protocol](endpoint_protocol.md), [Endpoint callback](endpoint_runtime.md),
[AI Search](ai_search.md) and [persistent data](data_api.md) cards for those
features, and [deploying and publishing](deployment_publication.md) when the
user asks for it.

## Order of work

1. Design the data interaction first, with every behavior owned once and each
   fact asked once (`app.design.*`, `app.ask.*`, `app.model.*`, `app.ai.*`);
   presentation follows (`app.template.*`).
2. Choose a type for each occurrence (`app.types.*`, `app.rows.*`,
   `app.nullable.*`, `app.representation.*`) and keep published shapes
   compatible (`app.wire.*`).
3. Declare the package: identity, entry functions, exports and dependencies
   (`app.identity.*`, `app.discovery.*`, `app.external.*`), then functions,
   permissions and the task lifecycle (`app.function.*`, `app.task-creation.*`,
   `app.permissions.*`, `app.persistence.*`, `app.version.*`,
   `app.removal.*`), using the installed SDK's runtime APIs (`sdk.*`).
4. Document every schema-visible node (`app.docs.*`) under the
   [source documentation card](source_documentation.md).
5. Compile with Studio until validation succeeds (`app.compile.*`,
   `deploy.mcp.*`), check `app.checklist.*`, then
   [validate and deliver](delivery.md).

<!-- generated:codebot-rules task=app-design -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Required design principle

Owner: [app-contract.md#required-design-principle](../reference/app-contract.md#required-design-principle)

- `app.design.smallest-complete` (judgment): **The goal is the smallest design that remains logically complete.** All App authors, including CodeBots, must follow this principle. It is a requirement, not an optional style preference.
- `app.design.complete-with-owners` (judgment): Implement every required behavior, condition, and outcome, with a clear owner for each responsibility. Use the platform's defined behavior and normal App lifecycle instead of duplicating them with additional mechanisms. Add App logic when the application's requirements need it; simplicity must not omit necessary validation, permissions, consistency, or user control.
- `app.design.natural-data-model` (judgment): **Model data in its natural form.** Declare each stored type as what the data is: its real fields, one owner per fact, and a Union where a value is one of several shapes. Never reshape stored data for a screen, a workflow step, or a question order. Human-facing types, the Request and Response payloads a person edits and views, are views of that data: they may wrap it as the person's path needs, for example as a wrapper member within a Union, while the stored model stays as it is.
- `app.design.human-shortest-path` (judgment): **The shortest path is the person's.** Human usability is part of what the smallest design optimizes: the fewest steps, questions, and screens a person needs to reach a complete, correct answer. A design that is shorter for its author or a CodeBot never justifies a longer path for the person.
- `app.design.no-contradiction` (judgment): **The two never conflict.** The goal is human first, and the natural data model serves it: the platform presents the model in its natural form along the person's shortest path, so neither is traded for the other. Where they seem to conflict, the platform is missing a presentation mechanism; raise that gap instead of bending the data or lengthening the person's path. Presentation and interaction adapt to the model through the platform: schema attributes such as `AskOnAdd`, `ReadOnly`, and `Hidden`, and Display Template rules such as `lock`, `flatten`, `promote`, and layouts. When the platform has no way to present a natural model, that is a platform gap to raise, not a reason to bend the data.

### Ask once, directly

Owner: [app-contract.md#ask-once-directly](../reference/app-contract.md#ask-once-directly)

- `app.ask.no-gate-question` (judgment): A question exists only to collect a fact. Do not add a Boolean, Yes/No or "something else applies" switch whose only effect is to reveal a follow-up asking the same thing in detail: the person answers twice, the App needs extra navigation and state, and the stored switch can disagree with the real answer.
- `app.ask.detailed-question` (judgment): Ask the detailed question itself: a Bitflags or Enumeration list with a "None of these" member, a List that may be empty, or a required Union whose variants carry their own data. Keep a separate question only when its answer decides something the detailed answer cannot.

### AI access in every design step

Owner: [app-contract.md#ai-access-in-every-design-step](../reference/app-contract.md#ai-access-in-every-design-step)

- `app.ai.document-pages-import` (judgment): A page whose values come from a document declares [document import](../reference/llm-document-transformation.md).

### Application model

Owner: [app-contract.md#application-model](../reference/app-contract.md#application-model)

- `app.model.no-invented-interaction` (judgment): Background tasks and TypesOnly packages retain their ordinary roles and need no invented user interaction.
- `app.model.data-first` (judgment): Design the data interaction first, then choose its presentation.
- `app.model.implement-once` (judgment): Implement the operation once in the task; the GUI, AI and software interfaces derive from that common description. A person can use the GUI directly without AI involvement.
- `app.model.access-checks`: Each participant remains subject to its applicable access checks. Generated clients follow the [human interaction lifecycle](../reference/protocols.md#human-task-interaction); software may agree with the service on its own ordering within the declared protocol. [`ExcludeUI`](../reference/protocols.md#declaring-an-endpoint) can limit the GUI and AI projection while retaining the underlying Endpoint contract.
- `app.template.not-schema`: The template is mandatory, but an App with neither compiles to the valid empty template. It is not part of the schema, and changing only the template never changes the App version.
- `app.template.edit-source`: Edit `display.json` or source attributes, never the compiled template.
- `app.model.no-second-schema`: Do not create a second JSON schema, manually assign schema IDs, or treat generated schema JSON as source.

### Project and discovery rules

Owner: [app-contract.md#project-and-discovery-rules](../reference/app-contract.md#project-and-discovery-rules)

- `app.discovery.entry-functions`: Declare task entry functions in `src/lib.rs` for Rust or Studio's selected entry module for TypeScript. Other implementation can live in ordinary modules. Function re-exports do not replace these declarations. The [code organization practice](../reference/code-organization.md) explains module boundaries and exact-name type exports.
- `app.discovery.ordinary-roots`: An ordinary App uses every non-ignored public/exported function as a root. Its published type set is the minimum transitive closure needed by:
- `app.discovery.unreachable-unpublished`: An unreachable public type is not published by an ordinary App.

### Schema compilation with Codex

Owner: [app-contract.md#schema-compilation-with-codex](../reference/app-contract.md#schema-compilation-with-codex)

- `app.compile.edit-validate-loop` (judgment): After changing public functions, types, attributes, or dependencies, save the files, compile the affected project, fix the reported source errors, and repeat until validation succeeds.
- `app.compile.edit-source`: Edit source declarations rather than generated schemas. Continue to run language builds and tests separately.
- `app.compile.saved-files-only`: Compilation uses saved files and installed or locally resolvable dependencies. It does not read unsaved editor buffers, install missing dependencies, overwrite source or generated files, or deploy. No Hub connection is required.
- `app.compile.success-scope`: Success means source schema validation passed; it does not establish exact deployed dependency-version bindings or replace Studio's deployment checks.

### Type model

Owner: [app-contract.md#type-model](../reference/app-contract.md#type-model)

- `app.types.closest-native` (judgment): Use the closest native type. Do not encode typed values as ad-hoc strings.
- `app.types.integer-width`:

  | Libertas meaning | TypeScript | Rust |
  | --- | --- | --- |
  | Signed/unsigned integer | `number` plus `@LibertasNumber(type=...)` | concrete `i*`/`u*` |

- `app.types.date-time-encoding`: `DateOnly` is an integer `yyyyMMdd` calendar tuple, not an epoch timestamp. `TimeOnly` is seconds from midnight. `DateTime` is epoch seconds presented in the viewer's local timezone.

### User-facing and non-user-facing types

Owner: [app-contract.md#user-facing-and-non-user-facing-types](../reference/app-contract.md#user-facing-and-non-user-facing-types)

- `app.types.review-occurrences` (judgment): Review each occurrence of a type, including its editability and visibility, rather than classifying it by `pub` or `export` alone.
- `app.types.separate-input-internal` (judgment): Reuse a type when these meanings and constraints agree. Use separate input and internal types when they differ, and map between them explicitly. For example, an editor can use blank text while a stored record uses absence. This does not justify copying every type into a second hierarchy.
- `app.types.input-optional-text`:

  | Meaning | Recommended representation |
  | --- | --- |
  | Ordinary optional text, where blank means not supplied | String with `Default=""`, `SizeMin=0`, and a suitable maximum. |

- `app.types.input-required-text`:

  | Meaning | Recommended representation |
  | --- | --- |
  | Required text | String with `SizeMin` of at least 1; omit a Default when the person must supply the answer. |

- `app.types.input-optional-override` (judgment):

  | Meaning | Recommended representation |
  | --- | --- |
  | An optional override of a value the App computes or receives, or harmless supplementary detail | Nullable (`Option<T>` in Rust), with null meaning “use the computed or received value” or “not given”, stated in its `[NullText]`. For output and internal data, Nullable also marks an unavailable value as distinct from zero. |

- `design.nullable.explicit-domain-choice`:

  | Meaning | Recommended representation |
  | --- | --- |
  | A yes-or-no decision whose “yes” carries details, or an optional override | Nullable labelled with `[NullText]` and `[SetText]`; no Default when the person must answer, `$null` when skipping means null. See [Nullable choices](../reference/app-contract.md#nullable-choices). |

- `app.types.input-optional-min-length`:

  | Meaning | Recommended representation |
  | --- | --- |
  | Optional text that must meet a minimum length when supplied | Nullable String with the appropriate size bounds; null remains valid. |

- `app.types.input-named-alternatives` (judgment):

  | Meaning | Recommended representation |
  | --- | --- |
  | A decision between named alternatives, including “none” and “provide details” | A labeled Enumeration for choices without payloads, or a labeled Union when a choice carries additional input. |

- `app.types.input-folding` (judgment):

  | Meaning | Recommended representation |
  | --- | --- |
  | Details that only need to expand or collapse | A Table with [folding](../reference/presentation.md#folding-structs-and-unions). Folding changes presentation without changing the answer. |

### Rows a person adds

Owner: [app-contract.md#rows-a-person-adds](../reference/app-contract.md#rows-a-person-adds)

- `app.rows.service-rows-readonly` (judgment): When the service creates the rows and the person only edits them, mark the List `ReadOnly`, which fixes its membership while its members stay editable ([ReadOnly Lists](../reference/presentation.md#common-field-metadata)).

### Default or no Default

Owner: [app-contract.md#default-or-no-default](../reference/app-contract.md#default-or-no-default)

- `app.default.decided`: Every field a person answers gets an explicit, reasoned decision, never an accident of its source type or a copied pattern. **No `Default`** when the person must answer: the field stays [undefined](../reference/schema.md#undefined-and-its-sources) and blocks Submit until answered, on a client-built request ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). **A `Default`** only when that value is the answer the person would give without reading, such as a usual value or `$null` where skipping lawfully means null. Never default an answer that has consequences, such as eligibility, an election, consent or a fact the App's rules test: a Default is never the person's confirmation. Decide again whenever the field's question, type or page changes. The rows below apply this decision to each kind of field.

### Nullable choices

Owner: [app-contract.md#nullable-choices](../reference/app-contract.md#nullable-choices)

- `app.nullable.label-wording` (judgment): `NullText` is a complete answer; `SetText` introduces the value entered right after it, so it reads as a lead-in to that value. For example: “Alive at the end of 2026” and “Died in 2026 on” followed by the date; “Use the payer's basis” and “My own basis is” followed by the amount; “Not a real estate professional” and “Real estate professional” followed by the qualifying details.
- `app.nullable.fallback-labels` (judgment): The defaults “None” and “Specify” are only a client fallback; they never explain the decision.
- `app.nullable.must-answer`: If the person must answer, give no Default. The choice then starts with no branch selected and blocks Submit until the person chooses, exactly like a Union without a Default.
- `app.nullable.skip-means-null`: If skipping means null, such as an optional override of a value the App computes or receives, or optional detail whose omission decides nothing, give the reserved Default [`$null`](../reference/schema.md#reserved-default-values) (Rust `#[libertas_default($null)]`). The choice starts on the null branch, and its `[NullText]` says what null means.
- `app.nullable.usual-value-default` (judgment): If one value is the usual answer, give that value as the Default; the choice starts on the value branch with it.
- `app.nullable.three-answers` (judgment): Use a labelled Enumeration or Union instead when there are three or more answers, or when the negative answer carries data of its own. For output and internal data, which no person answers, Nullable remains the normal way to state that a value is absent, with `[NullText]` naming any meaning other than None.
- `app.nullable.native-names` (judgment): Native `None`/`Some` or `Nothing`/`Something` names never explain a decision; this is not the gate [Ask once, directly](../reference/app-contract.md#ask-once-directly) forbids, because the value branch carries the details itself and the person answers once.
- `app.nullable.option-string-min`: An `Option<String>` with `SizeMin=1` is valid: null is allowed, but a present empty String is rejected. It therefore differs from a non-Nullable String with `SizeMin=0`. Use the latter for ordinary optional text when that distinction serves no purpose. Size bounds do not supply initial values.
- `app.nullable.unanswered-start`: Under the [initialization contract](../reference/schema.md#shared-copyfrom-and-copyto-rules), when no Fixed, Schema Default, or localized default supplies a value, a String starts unanswered even if it permits empty text, an editable Nullable starts with no branch selected, and a Union starts unselected.
- `app.nullable.default-not-confirmed` (judgment): A Default selecting “none” does not establish that the person confirmed that choice.

### Reviewing a representation change

Owner: [app-contract.md#reviewing-a-representation-change](../reference/app-contract.md#reviewing-a-representation-change)

- `app.representation.review-uses` (judgment): When changing Nullable text to blank text, review presence checks, validation rules, label fallbacks, calculations, and grouping keys. Translate `""` to absence only where the destination uses that meaning, and translate absence back to `""` for that editor. Do not accidentally group unrelated unnamed records together. Keep unknown amounts distinct from zero.
- `app.representation.update-docs-import` (judgment): Update [source comments](../reference/documentation.md#parameter-and-field) and [AI Import declarations and instructions](../reference/llm-document-transformation.md#blank-values-and-output-types) to match the new blank semantics: making manual entry optional does not enable blank import results, which follow the [optional-text import rule](../reference/llm-document-transformation.md#declared-blank-value). Inspect the generated schema and document, and validate the affected entry and import behavior.
- `app.representation.compatibility` (judgment): Review published protocol and persistent-data compatibility before changing any representation. Replacing Nullable with a String or custom Union is not automatically compatible. Follow the [data-type evolution contract](../reference/data-type-evolution.md); changing an editor does not require changing an existing stored representation.

### External types and App dependencies

Owner: [app-contract.md#external-types-and-app-dependencies](../reference/app-contract.md#external-types-and-app-dependencies)

- `app.external.foreign-type-metadata`: The official type supplies inherited schema/document metadata when consumers materialize the graph, while explicit compatible attributes override same-name official attributes.
- `app.external.no-authored-dependencies`: Do not write versioned `AppDependencies` in source or edit generated locks. Studio discovers and verifies dependencies, deploys them first, and records the actual exact versions. The [deployment dependency contract](../reference/deployment.md) defines discovery, including which Endpoint constraints create a dependency, Plaza preflight, installation, failure handling, and the exact [`Version` pin](../reference/deployment.md#schema-availability-and-task-requirements).

### Functions, permissions, and singleton tasks

Owner: [app-contract.md#functions-permissions-and-singleton-tasks](../reference/app-contract.md#functions-permissions-and-singleton-tasks)

- `app.function.singleton`: `Singleton`, limiting the Hub to at most one task for the App and native function name across versions.
- `app.permissions.package-owned-reasons` (judgment): Different Apps can request the same permission for different reasons, so explanations are package-owned rather than global.

### Task version selection

Owner: [app-contract.md#task-version-selection](../reference/app-contract.md#task-version-selection)

- `app.version.directional-evolution`: Which version reads the data another wrote follows the [directional evolution rules](../reference/data-type-evolution.md).

### Documentation and localization

Owner: [app-contract.md#documentation-and-localization](../reference/app-contract.md#documentation-and-localization)

- `app.docs.first-line-label`: the first documentation line is the display name;
- `app.docs.description-lines`: remaining ordinary lines are the description;
- `app.docs.persisted-text-wins`: persisted or user-assigned text, including an empty string, is immutable to programmatic defaulting; these defaults apply only while creating a value for which no persisted value exists;
- `app.docs.list-divider`: list documentation may contain a line of at least four hyphens; content above the divider describes the list and content below describes each item; and
- `app.docs.divider-before-properties`: the array divider must appear before custom document properties belonging to the list item.
- `app.docs.plain-comments`: Plain code comments are not localized schema documentation.
- `app.docs.every-node-labelled`: Every authored package, function, named type, parameter, field, List container and item, Enumeration member, Union variant, and inline value must have a non-empty natural label and description that make sense without source names or implementation vocabulary. Custom document values and package string resources follow the same user-language and accessibility rules. The complete normative contract is the [schema documentation invariants](../reference/documentation.md).
- `app.docs.function-comments` (judgment): Every public function comment must additionally satisfy the [developer function-comment invariants](../reference/function-comments.md). Its description is a complete, actor-explicit account of the function's purpose, user choices and actions, automatic behavior, outputs, and material limits; it must not rely on generated type text to fill semantic gaps.

### Wire and compatibility discipline

Owner: [app-contract.md#wire-and-compatibility-discipline](../reference/app-contract.md#wire-and-compatibility-discipline)

- `app.wire.attributes-keep-wire-type`: Attributes such as `Format`, `PhysicalUnit`, `DateOnly`, `Bitflags`, `ValidationRules`, chart metadata, `ReadOnly`, `Hidden`, and `ExcludeUI` affect validation/presentation without changing the underlying wire type. `ExcludeUI` is specifically a Control-plane projection: the Hub omits a marked server Endpoint from its Control UI-device list and generated clients omit marked request operations. It never removes nodes from admin creation, editing, approval, persistence, routing, or authorization. See the [Endpoint declaration contract](../reference/protocols.md#declaring-an-endpoint).
- `app.wire.union-order`: Union declaration order is significant. `variant_index!` uses declaration order and append-only protocol/data Union evolution depends on it.
- `app.wire.append-only`: Append new variants for compatible Union evolution; do not insert, remove, reorder, or change existing payload shapes in a published append-only line.

### MCP deployment

Owner: [deployment.md#mcp-deployment](../reference/deployment.md#mcp-deployment)

- `deploy.mcp.rule-debugging-input`:

  | Rule debugging tool | Input and result |
  | --- | --- |
  | `evaluate_validation_rules` | `project` and optional `entry` identify saved App source as for `compile_schema`; `type` names a Request variant or data type. Supply either `value` in schema JSON value form or `avroPath`, a saved Avro request file. Optional `phase` is `final` (default) or `incremental`. Returns structured rule traces and a short text summary, with compile diagnostics. |

- `deploy.rule-debugger.no-hub-or-submit`: Rule debugging compiles the same saved source and materialized dependencies as `compile_schema`, then uses Studio's ordinary rule evaluator. It needs no Hub connection and never submits the supplied request.
- `deploy.mcp.rule-debugging-values`: JSON supplies the named Request's payload; omitted members remain unanswered and explicit null remains absence. A saved Avro Request includes its protocol Union discriminant. `avroPath` is absolute or relative to `project`. See [Debugging rules](../reference/validation-rules.md#debugging-rules) for outcomes, lazy reads, developer presentation, and CI fixtures.
- `deploy.mcp.preview-display`: `preview_display` shows one screen of the saved workspace App in Studio's Display Template preview, Hub-independent like `compile_schema`. It takes the absolute project directory, the protocol variant (a Union member such as a Response; `Union.Member` names it exactly when two Unions have a member of that name, and the result's `ambiguousVariant` lists the candidates when an unqualified name matched several) or function name to show, a device `form` (`phone`, `tablet`, `desktop`, or `tv`), optional `width`, `height`, `orientation`, `pointer`, and `theme` size-class/condition overrides, and an optional Libertas Schema Path `select` naming a node to inspect. It compiles the project, then opens a preview if none is open for it or refocuses the open one and switches it to this screen and device; later saves re-render the same preview live. Its JSON result reports `preview` (`opened`, `refocused`, or `headless` with a `reason`), the resolved device conditions and window, the root node's resolved layout, style, and hints, and, when `select` is given, the matched rules in cascade order with their winning members and each rule's source (a `display.json` line or a source attribute's file and line). Compile diagnostics are included, and `templateCurrent` is `false` when a failed compilation leaves the last good template on screen. It requires VS Code and Libertas Studio to be running and fails explicitly otherwise. For a local Request sample, optionally supply `value` (the variant's JSON payload, without its outer variant key) or `avroPath` (a saved raw protocol Avro Request including its Union discriminant, absolute or relative to the project), never both: the sample overrides open-task values and defaults, preserves unanswered JSON fields without initialization, and stays selected on later source saves; malformed input or an Avro variant mismatch fails explicitly. The result identifies `displayed` as `sample-value` or `sample-file` and `sample` names the input source, qualified variant and file path when applicable. Samples use local codecs without contacting a Hub or submitting a request; function and non-Request screens accept no sample.

### Standard properties

Owner: [documentation.md#standard-properties](../reference/documentation.md#standard-properties)

- `doc.boolean.semantics`: **A Boolean's `true` and `false` can carry any two meanings the field needs**, such as Accept and Decline: Yes and No are only the fallback captions, `[TrueText]` and `[FalseText]` state any other meaning, and the App's rules read the value under that meaning.
- `doc.nullable.null-text-rationale` (judgment): This is deliberate: AI CodeBots especially assume null means None, which has caused subtle design flaws; writing `[NullText]` on every question a person answers forces the author to decide what null means there.
- `doc.nullable.null-text-meaning`: **Null can carry any meaning the field needs**: it means None only until `[NullText]` states another meaning, which every Nullable, editable or not, must then declare; a non-editable Nullable shows it wherever its null is displayed.

### Shared detailed reference

Owner: [index.md#shared-detailed-reference](../reference/index.md#shared-detailed-reference)

- `sdk.owns-runtime-apis`: The installed public SDK owns exact runtime APIs: [`@libertas-ai/types`](https://www.npmjs.com/package/@libertas-ai/types) for TypeScript and [`libertas-rs`](https://github.com/LibertasIoT/libertas-rs) for Rust. Use the version selected by the project. Chart references define their versioned contracts and fallback behavior; they do not certify every older client build.

### Uses and storage

Owner: [message-format.md#uses-and-storage](../reference/message-format.md#uses-and-storage)

- `lmf.resources.extraction-only`: **`StringResources` is for schema and document extraction only.** The parser reads the catalog constants from source whatever the build configuration, and clients and the Hub render every message from the package document. For a message, App runtime code therefore emits its resource key and typed arguments only: it never reads, compares, renders or sends the text, and the built App contains no message text, neither a catalog nor a copy of one of its sentences. Keep every catalog out of normal builds: in Rust a module gated out, such as `#[cfg(test)] mod strings;`, and in TypeScript a module no runtime module imports. Tests may read the catalog. App code names a resource by its key, a literal or one built at run time; a key built at run time is checked by the App's tests, since neither the compiler nor Studio can see it.

### Choosing a role

Owner: [protocols.md#choosing-a-role](../reference/protocols.md#choosing-a-role)

- `protocol.choice.list` (judgment): **Choosing the role for a need.** Each need has one role; the linked section owns its rules. Never substitute one role for another.

### Strict questions and data pages

Owner: [protocols.md#strict-questions-and-data-pages](../reference/protocols.md#strict-questions-and-data-pages)

- `workflow.entry.undefined-not-encoded` (judgment): **EditRequest or WorkflowRequest.** A node is [undefined until it is initialized](../reference/schema.md#undefined-and-its-sources), and undefined cannot be encoded, so no message carries an unanswered field. An editor opened by either one carries the platform's two workflow buttons, **Go Back** (when the Request declares `PrevRequest`) and **Next/Submit**, whose captions a Display Template may customize through [`button_labels`](../reference/presentation.md#button-labels); the App adds no buttons of its own for them ([submit and previous actions](../reference/protocols.md#submit-previous-actions-and-completion)). What the page needs decides how it opens:
- `workflow.edit-request.why` (judgment): **Nothing would start undefined: EditRequest.** It carries a Request the service has stored, or an unsaved page's value equal to its ordinary initialization; the client adopts it as it is. Its Nullables are true optionals whose null is a real answer with the meaning its `NullText` names, such as "None" or "Does not apply".
- `workflow.client-built.why` (judgment): **Fields left undefined and the person chooses the page: a client-built request.** The client constructs it from an offered action: a `NextRequest` action, a link (available while its target is in that action set), or a landing request. Like a WorkflowRequest editor, it commonly copies known facts by `CopyFrom` from the displayed Response, and every field nothing initializes stays undefined. An offered request opens only when the person chooses it, or by the [automatic singleton selection](../reference/protocols.md#response-action-constraints). Who opens the page, not copying, separates it from WorkflowRequest.
- `workflow.workflow-request.why` (judgment): **Fields left undefined and the service opens the page at once: WorkflowRequest**, with or without copied facts, and only then. The service sends a Response; the client constructs the named Request, copies any known facts by `CopyFrom`, and leaves the rest undefined. An unanswered source path is absent, never null: CopyFrom stops at a valid null, which a non-Nullable field cannot take.
- `workflow.strict-question.required`: **A must-answer question** is a required field with no `Default` on a request the client constructs (an offered action, link or landing request, or a `WorkflowRequest` editor): it starts unanswered, shows no error, and blocks submission. Never model it as a `Nullable` whose `NullText` means "not answered yet", and never send it in an EditRequest.

### Undefined and its sources

Owner: [schema.md#undefined-and-its-sources](../reference/schema.md#undefined-and-its-sources)

- `init.undefined-relationships` (judgment): **A node is undefined until it is initialized** by `Fixed`, a `Default`, a copy, a supplied value, or [type initialization](../reference/schema.md#ordinary-initialization) (an empty List, a null non-editable Nullable); otherwise it stays undefined until answered, which is how a must-answer question is asked ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). Undefined cannot be encoded, so no protocol message carries it; only an editor holds it, showing its neutral empty state ([unanswered fields](../reference/presentation.md#unanswered-fields)) and blocking submission while required.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Application model

Owner: [app-contract.md#application-model](../reference/app-contract.md#application-model)

- `app.identity.package-name`: An App identity has exactly one hyphen joining two lowercase ASCII snake_case names without edge or repeated underscores

### Project and discovery rules

Owner: [app-contract.md#project-and-discovery-rules](../reference/app-contract.md#project-and-discovery-rules)

- `app.discovery.types-only-helpers`: A public helper function in a TypesOnly package must be ignored

### Schema compilation with Codex

Owner: [app-contract.md#schema-compilation-with-codex](../reference/app-contract.md#schema-compilation-with-codex)

- `app.compile.project-directory`: Supply the absolute project directory to compile_schema; an optional entry selects the entry file
- `app.compile.dependencies-complete`: Missing, ambiguous or cyclic App dependencies fail compilation; a partial bundle is not a successful result

### Automated QA through Studio MCP

Owner: [app-contract.md#automated-qa-through-studio-mcp](../reference/app-contract.md#automated-qa-through-studio-mcp)

- `app.qa.lossless-events`: Consume every queued QA event; capacity applies back-pressure rather than dropping events
- `app.qa.task-create`: Choose a listed task function and validate its supplied configuration with ordinary editor defaults
- `app.qa.journal-exclusive-create`: A QA journal is a new private file of decoded endpoint events, with no saved credentials

### Rows a person adds

Owner: [app-contract.md#rows-a-person-adds](../reference/app-contract.md#rows-a-person-adds)

- `app.rows.hidden-needs-value`: Every Hidden field in a row a person can add needs a Default, Fixed or CopyFrom

### Bitflags

Owner: [presentation.md#bitflags](../reference/presentation.md#bitflags)

- `bitflags.zero-label.answer`: A Bitflags field is never Nullable; a required one answers none with its zero label

<!-- /generated -->
