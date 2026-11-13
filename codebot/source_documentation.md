# Writing source documentation

Load this card when a task adds or changes the documentation of a
schema-visible declaration: package text, a function comment, a named type,
parameter, field, List container or item, Enumeration member, Union variant or
inline value, a custom document property such as `[DefaultText]`,
`[TrueText]`, `[FalseText]`, `[NullText]` or `[SetText]`, or a package
`StringResources` value. Load it for every public function whose behavior
changes. Its rules are generated from the
[schema documentation contract](../reference/documentation.md) and the
[developer function-comment guide](../reference/function-comments.md), which
own them. The [documentation quick reference](documentation_invariants.md)
lists the standard properties and the
[function-comment template](function_comment_invariants.md) gives the comment
shape.

## Order of work

1. Inventory every schema-visible declaration and each document node it emits
   (`doc.workflow.*`, `doc.coverage.*`), including both List halves
   (`doc.list.*`).
2. Write a label and a description for each node in the exact source syntax
   (`doc.source.*`, `doc.syntax.*`), in user language (`doc.label.*`,
   `doc.description.*`, `doc.prose.*`, `doc.language.*`, `doc.discovery.*`),
   under its node's rules (`doc.package.*`, `doc.function.*`, `doc.type.*`,
   `doc.field.*`, `doc.enum.*`, `doc.union.*`, `doc.hidden.*`, `doc.chart.*`).
3. For each public function, write the complete discovery description
   (`fn.*`) and end it with the developer attribution, exactly
   `Developed by Display Name (account_name).`, or
   `Developed by account_name.` before a Plaza account exists
   (`fn.attribution.*`).
4. Add only documented custom properties on their owner (`doc.property.*`,
   `doc.boolean.*`, `doc.defaulttext.*`, `doc.nullable.*`,
   `doc.initial-names.*`, `doc.initial-text.*`, `doc.answer-labels.*`).
5. Write the package `StringResources` values (`doc.resources.*`).
6. Generate and inspect the default document, then run the language checker
   and Studio validation (`doc.review.*`, `fn.workflow.*`, `fn.review.*`).

<!-- generated:codebot-rules task=source-documentation -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Default or no Default

Owner: [app-contract.md#default-or-no-default](../reference/app-contract.md#default-or-no-default)

- `app.default.decided`: Every field a person answers gets an explicit, reasoned decision, never an accident of its source type or a copied pattern. **No `Default`** when the person must answer: the field stays [undefined](../reference/schema.md#undefined-and-its-sources) and blocks Submit until answered, on a client-built request ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). **A `Default`** only when that value is the answer the person would give without reading, such as a usual value or `$null` where skipping lawfully means null. Never default an answer that has consequences, such as eligibility, an election, consent or a fact the App's rules test: a Default is never the person's confirmation. Decide again whenever the field's question, type or page changes. The rows below apply this decision to each kind of field.

### Schema documentation invariants

Owner: [documentation.md#schema-documentation-invariants](../reference/documentation.md#schema-documentation-invariants)

- `doc.coverage.authored-public`: These are codebot authoring and review requirements. Generated document JSON permits empty strings so parsers can represent compiler-generated nodes and older or incomplete source. That structural allowance is not permission for a codebot to omit documentation from an authored public declaration.
- `doc.coverage.discriminator-exception`: The only ordinary exception is a compiler-generated TypeScript discriminator field whose document text the parser intentionally suppresses. Non-schema implementation details do not need schema documentation.
- `doc.coverage.no-hidden-exception`: `Hidden`, `ReadOnly`, protocol, persistent-data, and chart-related nodes are not exceptions: their meaning can still be used by validation, accessibility, localization, tools, management interfaces, or a future presentation.

### Source and output invariants

Owner: [documentation.md#source-and-output-invariants](../reference/documentation.md#source-and-output-invariants)

- `doc.source.authoritative` (judgment): Source comments are authoritative. Never patch generated schema or document JSON to improve wording.
- `doc.source.label-and-description`: Give every authored document node at least two ordinary lines: a concise natural label followed by a complete plain-language description. Do not rely on a parser fallback derived from a native identifier.
- `doc.source.rust-comments`: Rust package text uses inner rustdoc (`//!`) at the start of `src/lib.rs`. Rust declarations use outer rustdoc (`///`).
- `doc.source.typescript-comments`: TypeScript package text uses leading triple-slash comments (`///`) on the selected entry module. TypeScript declarations use JSDoc (`/** ... */`).
- `doc.source.engineering-comments` (judgment): Ordinary `//` comments are not localized. Use them, a README, or developer documentation for engineering details that must not appear in the user document.
- `doc.source.same-change` (judgment): Update source documentation in the same change as the behavior, meaning, choices, results, constraints, or presentation it describes.
- `doc.source.inspect-output` (judgment): Inspect the generated default-locale document as an output. Fix the owning source comment when a label, description, extension, or List half lands on the wrong node.
- `doc.output.generated-id`: it is not prose and is never authored or translated.
- `doc.output.use-site-and-type` (judgment): Documentation traversal stops at a named type reference. Consequently, the use-site field or parameter and the referenced named type must each make sense on their own. One never substitutes for the other.

### Universal user-language invariant

Owner: [documentation.md#universal-user-language-invariant](../reference/documentation.md#universal-user-language-invariant)

- `doc.language.audience` (judgment): Write for the people who use the capability, including nontechnical home, commercial, research, engineering, and healthcare users. Use specialist domain terms when that audience genuinely uses them, and explain unfamiliar terms when the surrounding context does not make them clear.
- `doc.label.short-natural` (judgment): be a short, natural name or action phrase;
- `doc.label.distinct` (judgment): distinguish the node from its siblings;
- `doc.label.standalone` (judgment): work when read aloud or shown without its parent; and
- `doc.label.not-identifier` (judgment): not be a source identifier, version token, full sentence, or keyword list.
- `doc.description.adds-meaning` (judgment): A description must add meaning beyond the label and declared type. It must say what the person chooses, enters, observes, receives, or can accomplish.
- `doc.description.behavior` (judgment): When it describes behavior, use active voice and identify the actor, action, object, scope, material condition, and modality. Preserve distinctions such as “can,” “must,” and “automatically.”
- `doc.prose.concrete-terms` (judgment): use concrete domain nouns and verbs rather than vague words such as “data,” “value,” “item,” “configuration,” or “information” by themselves;
- `doc.prose.special-values` (judgment): explain an important empty, unavailable, zero, false, `None`, or `Unknown` meaning when a person could otherwise misinterpret it;
- `doc.prose.no-visual-cues` (judgment): remain understandable without relying on order, screen position, color, icon, shape, animation, or a particular gesture;
- `doc.prose.client-neutral` (judgment): avoid instructions tied to one client, such as “click the button” or “tap the row,” unless the behavior itself exists only on that client;
- `doc.prose.explicit-nouns` (judgment): use explicit nouns instead of an ambiguous “it,” “this,” or “they”;
- `doc.prose.no-stuffing` (judgment): avoid unexplained acronyms, keyword stuffing, unsupported claims, and lists of capabilities the App does not provide; and
- `doc.prose.translatable` (judgment): be complete enough to translate without guessing an actor, scope, condition, or consequence.
- `doc.prose.no-internal-vocabulary` (judgment): Do not expose internal vocabulary in localized prose. In particular, do not describe something to a user as a function, field, type, struct, table, enum, Union, variant, schema, payload, protocol, endpoint, request, response, runtime, discriminator, index, FQN, Avro value, wire value, or implementation version.
- `doc.prose.no-mechanics` (judgment): Do not expose native casing, storage encoding, Libertas Schema Paths, integer codes, byte layouts, or transport mechanics. Preserve required engineering facts in an ordinary source comment or developer document instead.
- `doc.language.technology-neutral` (judgment): Apply this rule both when authoring source comments and when translating their localized text. Write natural, idiomatic language for the intended audience; keep it technology-neutral by describing the capability rather than its implementation. An App's “weather endpoint,” for example, is a “weather service” to its users. Prefer the specific purpose or the natural equivalent of “service” in the target language. This is a wording choice, not a mechanical word replacement: retain specialist terms when they describe the user's domain and their precise meaning matters.
- `doc.language.preserve-meaning` (judgment): Preserve every material meaning, choice, control, automatic behavior, result, warning, condition, prerequisite, and limit.
- `doc.language.identifiers-exact`: This guidance changes prose only; identifiers, structural keys, and message placeholders retain their exact spelling and syntax.

### Domain references and discoverability

Owner: [documentation.md#domain-references-and-discoverability](../reference/documentation.md#domain-references-and-discoverability)

- `doc.discovery.domain-references` (judgment): Document the names people use to identify a supported capability. Where relevant, connect formal domain references, document or form numbers, standard abbreviations, and common names in natural descriptive prose. For example, a tax topic can identify the forms and statutory provisions it covers, while an equipment topic can identify the industry standard it implements. These are user-domain references, distinct from internal identifiers and wire codes. Preserve their exact meaning during translation and explain unfamiliar abbreviations. Keep labels concise; put this context in descriptions.
- `doc.discovery.actual-scope` (judgment): State the actual purpose, supported scope, and material conditions. A reference to a subject does not establish that every case within it is supported.
- `doc.discovery.link-topics` (judgment): For links that share one request but bind different topics or records, document the individual topic or selected choice as well as the common operation, so the destination is understandable from its own documented meaning and data.
- `doc.discovery.enumeration-destination` (judgment): When an Enumeration member identifies a navigation destination, its extracted member description owns the destination’s domain references, abbreviations, and common names. For example, a qualified business income topic can associate Section 199A, Form 8995/8995-A, and QBI deduction with that supported topic. The client supplies the descriptions of the actual bound members; a generic request description does not replace them. Maintain this information in its owning source documentation when behavior changes; generated documents and private code comments are not substitutes.

### Package

Owner: [documentation.md#package](../reference/documentation.md#package)

- `doc.package.label-description` (judgment): The package label is the recognizable product, service, or capability name, not the package identifier. Its description explains the overall outcome, intended setting or audience, major capabilities, and material prerequisites or limits. Avoid internal architecture and unsupported marketing claims.

### Function

Owner: [documentation.md#function](../reference/documentation.md#function)

- `doc.function.discovery-description` (judgment): The function label names the activity a person can configure or use. Its description is the complete discovery description and must additionally follow the [developer function-comment guide](../reference/function-comments.md), including the seven completeness categories, explicit actors, and final developer attribution.
- `doc.function.attribution-only-functions`: Developer attribution belongs only in function descriptions; do not append it to package, type, field, member, or extension text.

### Named type

Owner: [documentation.md#named-type](../reference/documentation.md#named-type)

- `doc.type.concept` (judgment): Describe the real-world concept represented by one value and why a person sees or supplies it. Do not say that it is a struct, table, grouping, payload, or a container of fields. A reusable named type must make sense independently of every place that references it.

### Parameter and field

Owner: [documentation.md#parameter-and-field](../reference/documentation.md#parameter-and-field)

- `doc.field.input-output` (judgment): Give every parameter and field a contextual natural label. For an input, explain what the user chooses or enters and the effect or scope of that choice. For output or read-only information, explain what the App reports, records, or calculates and when it is meaningful.
- `doc.field.empty-string` (judgment): Match the wording to the [type's use](../reference/app-contract.md#user-facing-and-non-user-facing-types). For ordinary optional text stored as `""`, say “Leave it empty if absent,” and explain any consequence of leaving it empty, such as using another name or leaving an activity ungrouped. Do not say “Leave it undefined” when the answer is an empty String.
- `doc.field.nullable-absence` (judgment): For a genuinely nullable answer, explain what absence means and how it differs from a present empty answer or zero. AI-import instructions must use the same meaning under the [blank-value conversion contract](../reference/llm-document-transformation.md#blank-values-and-output-types).
- `doc.field.output-and-internal` (judgment): An output description explains the reported result and its availability rather than telling the person to enter it. A non-user-facing schema field still needs domain documentation: describe what the App records and the meaning of missing or empty content. Keep mapping code, storage encodings, and other engineering details in ordinary `//` comments.
- `doc.field.boolean` (judgment): A Boolean label states the positive choice or state. Its description makes the `true` and `false` consequences unambiguous when they matter; “Enabled” by itself is not enough.
- `doc.field.quantities` (judgment): For a number, date, time, duration, or physical quantity, describe the domain meaning and scope. Do not explain its integer width, epoch, raw date encoding, or storage unit as implementation mechanics. Declare machine-readable unit and format metadata with the appropriate schema annotation.
- `doc.field.no-ai-hints` (judgment): The platform supplies [AI value-representation hints](../reference/llm-document-transformation.md#value-representation-hints) from that metadata; do not duplicate those hints in source comments. Once a `PhysicalUnit` is declared, the comment never names the unit again (no "in gallons" or "(percent)"). Clients already display the declared unit with the value under the presentation contract ([physical units](../reference/charts/2d.md#551-physical-units): `50` with `percent` shows as `50%`), so a comment naming it would repeat it on screen.
- `doc.field.system-object` (judgment): For a Libertas system object, name its actual role, such as “Rain gauge,” “Patient,” “Delivery vehicle,” or “Sprinkler valve.” Never call it a device ID, user ID, endpoint ID, or plain integer. Keep the Libertas system-object type intact so clients can resolve its localized identity.
- `doc.field.free-text` (judgment): For free text, describe what content is appropriate and any material use or visibility. Never present a secret as ordinary documented input.
- `doc.field.use-site-retained` (judgment): A use-site field retains its own documentation even when its named type is thoroughly documented.

### Enumeration and bitflags

Owner: [documentation.md#enumeration-and-bitflags](../reference/documentation.md#enumeration-and-bitflags)

- `doc.enum.members` (judgment): The Enumeration type describes the decision, state, or classification as a whole. Every member has a natural label and a description of its observable meaning or consequence. This includes `None`, `Unknown`, and zero-valued members. Never expose the native constant name or numeric code.
- `doc.enum.bitflags` (judgment): A Bitflags catalog follows the same rules, but each nonzero member describes one independently selectable option. Do not phrase its type as one mutually exclusive choice.
- `doc.enum.no-positional`: The parsers' public positional `@LibertasEnum(A, B, C)` and `#[libertas_enum(A, B, C)]` compatibility forms fabricate member nodes without a source-document location for each member. A codebot therefore must not generate those forms: they cannot satisfy this all-member documentation invariant. Use a native numeric enum whose declaration and every member can be documented. Existing positional catalogs remain parseable, but parser support does not waive the codebot authoring rule.

### Union and variant

Owner: [documentation.md#union-and-variant](../reference/documentation.md#union-and-variant)

- `doc.union.variants` (judgment): The Union type describes the family of possible actions, events, states, or results. Every variant has a standalone natural label plus a description of the observable action or state, its trigger or scope, and its important consequence. Do not label variants “Request,” “Response,” “V1,” or by their discriminator tokens.
- `doc.union.inline-fields` (judgment): Document every inline field inside a variant by the ordinary field rules.
- `doc.union.typescript-source`: TypeScript may obtain final variant text from a discriminator member, the Union branch, or the variant's named payload type. Choose one authoritative source for the intended result. If more than one source contains prose, keep the text semantically identical; never depend on precedence to hide contradictory wording. Rust takes the variant's own rustdoc.

### List container and item

Owner: [documentation.md#list-container-and-item](../reference/documentation.md#list-container-and-item)

- `doc.list.two-sections`: Every List has two complete document sections separated by a line containing at least four hyphens:
- `doc.list.container-half`: Above `----`, use a plural label and explain the collection's purpose, meaningful ordering, limits, or selection behavior.
- `doc.list.item-half`: Below `----`, use a singular label and explain what one entry represents and how a person recognizes or uses it.
- `doc.list.both-halves` (judgment): Author both halves even when the item is a separately documented named type.
- `doc.list.property-ownership`: Custom properties above the divider belong to the List container; properties below it belong to the List item.

### Hidden, interaction-context, and chart data

Owner: [documentation.md#hidden-interaction-context-and-chart-data](../reference/documentation.md#hidden-interaction-context-and-chart-data)

- `doc.hidden.truthful` (judgment): Document hidden and read-only values truthfully without revealing a secret. Explain their domain purpose, such as associating a follow-up action with the same inspection, rather than describing them as a cookie, token, cursor, or transaction mechanism.
- `doc.chart.accessible` (judgment): Chart documentation describes the domain relationship and the meaning of each visible quantity or category. It must remain useful to a screen reader and must not depend on axis position, color, shape, rendering library, or client-computed geometry.
- `doc.chart.visible-text` (judgment): Visible human text uses a localized Enumeration, `FormattedText`, or the appropriate Libertas system-object type when applicable. A plain `String` is reserved for genuine literal domain or user-authored text, not a fixed English vocabulary.

### Exact source-document syntax

Owner: [documentation.md#exact-source-document-syntax](../reference/documentation.md#exact-source-document-syntax)

- `doc.syntax.label-line`: The first ordinary line becomes `DisplayName` for a package, function, or named type, and `Name` for a field, item, member, or variant. Remaining ordinary lines become `Description`. For reliable cross-language output, always provide both a label line and at least one description line.
- `doc.syntax.paragraph-breaks`: Codebots may wrap a long source sentence across adjacent rustdoc/JSDoc lines, but must leave a blank documentation line only when one paragraph break is intended. This normalization applies to `Description`, not custom-property or `StringResources` values.
- `doc.syntax.typescript-param`: TypeScript function parameters may use `@param name` documentation. The parser merges that text over documentation attached directly to the parameter, so use one authoritative location or keep both semantically identical.
- `doc.syntax.rust-param`: Rust parameter rustdoc and schema attributes are written on the typed parameter inside the exported function signature.

### Rust example

Owner: [documentation.md#rust-example](../reference/documentation.md#rust-example)

- `doc.syntax.example-profile` (judgment): Examples use a placeholder developer profile. A codebot must substitute the current Plaza account and display name under the function-comment contract.

### Custom document properties

Owner: [documentation.md#custom-document-properties](../reference/documentation.md#custom-document-properties)

- `doc.property.marker-line`: The case-sensitive `[PropertyName]` marker is the only content on its line.
- `doc.property.after-description`: Put extensions after the ordinary label and description.
- `doc.property.before-tags`: Put all ordinary text and extension blocks before the first `@Libertas...` schema tag in TypeScript JSDoc. The TypeScript document parser stops collecting document text at that first tag. This ordering is also safe in Rust rustdoc.
- `doc.property.no-repeat`: Multiple extensions are allowed. Never repeat a property name in one section.
- `doc.property.value-normalization`: Leading and trailing blank space is removed while internal newlines are preserved.
- `doc.property.localized-values`: Property names and structural keys are stable and are never translated. Property values are localized.
- `doc.property.unknown` (judgment): Unknown valid properties are preserved and translated, but Libertas gives them no generic display or behavior. Use one only when a documented consumer contract defines its semantics. Never invent an extension from its name.
- `doc.property.structural-names`: A name that collides with a structural key at its owning level, including `Locale`, `Types`, `Name`, or `Type`, is also reserved.

### Standard properties

Owner: [documentation.md#standard-properties](../reference/documentation.md#standard-properties)

- `doc.boolean.label-effects`: They label `true` and `false` respectively; omitted labels use localized Yes and No. Either property selects an [explicit choice control](../reference/presentation.md#boolean-answer-controls). Labels do not supply defaults, change Boolean values, or cause submission. Use ordinary document localization and fallback.
- `doc.boolean.semantics`: **A Boolean's `true` and `false` can carry any two meanings the field needs**, such as Accept and Decline: Yes and No are only the fallback captions, `[TrueText]` and `[FalseText]` state any other meaning, and the App's rules read the value under that meaning.
- `doc.boolean.switch`: A non-nullable Boolean uses an on/off switch only when it has an explicit Boolean Default and neither custom label is supplied. Otherwise it uses the two-choice control, including when a default preselects an answer. This decision follows metadata, not whether the user has already answered. Fixed and ReadOnly retain their existing restrictions on editing; no new schema attribute or opcode is added.
- `doc.defaulttext.real-content` (judgment): It is real, safe content suitable for the locale and context, never a placeholder instruction, sentinel, resource key, source identifier, or schema `Default`. It does not emit or replace the `Default` schema attribute.
- `doc.defaulttext.omit` (judgment): Omit it when the ordinary fallback already gives exactly the intended value, or no truthful general value exists. Never fabricate user-specific identity, clinical, financial, or security content.
- `doc.nullable.null-text-rationale` (judgment): This is deliberate: AI CodeBots especially assume null means None, which has caused subtle design flaws; writing `[NullText]` on every question a person answers forces the author to decide what null means there.
- `doc.nullable.null-text-meaning`: **Null can carry any meaning the field needs**: it means None only until `[NullText]` states another meaning, which every Nullable, editable or not, must then declare; a non-editable Nullable shows it wherever its null is displayed.
- `doc.nullable.no-settext-boolean-picker`: A nullable Boolean (TrueText, FalseText, NullText) and a picker (NullText, then its members) never show `[SetText]`.
- `doc.nullable.editable`: A client falls back to localized None and Specify only when a label is missing, and a non-editable Nullable never needs `[SetText]`: a Nullable is editable when a Request variant or a function's configuration reaches it without passing through a ReadOnly, Hidden or Fixed node ([Nullable choices](../reference/presentation.md#nullable-choices), under the [required answer-label rule](../reference/documentation.md#standard-properties)).
- `doc.nullable.labels-only`: Both change only labels: they neither supply a value nor label false, zero, a present empty string or collection, an unanswered required field, or a selected Nil Union branch.
- `doc.nullable.two-answers` (judgment): Write them as the two answers the person chooses between; `SetText` is the lead-in to the value entered right after it, such as “Died in 2026 on” before a date.
- `doc.initial-names.concise` (judgment): Choose concise, useful, editable initial names for tasks and Endpoint servers. Their ordinary localized labels already provide naming fallbacks; declare `DefaultText` only when the intended initial name differs from that fallback.
- `doc.property.no-substitute`: No standard property substitutes for the owner's ordinary label and description.
- `doc.initial-text.priority-chain`: The remaining priority chain applies only when no persisted value exists and the client is constructing a new value. For a new String or Endpoint-server presentation value left to ordinary initialization by the [shared copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules), an applicable schema `Fixed`/`Default` value has first initialization priority, localized `DefaultText` is next, and the ordinary empty or localized field-label fallback is last. For a new blank task name, localized `DefaultText` has first initialization priority, then the localized function `DisplayName`, then the native function name.

### Answer labels in AI descriptions

Owner: [documentation.md#answer-labels-in-ai-descriptions](../reference/documentation.md#answer-labels-in-ai-descriptions)

- `doc.answer-labels.declared-only.authoring`: Declare answer labels explicitly when AI needs their meanings; [fallback UI captions are not sent](../reference/documentation.md#answer-labels-in-ai-descriptions).

### StringResources prose

Owner: [documentation.md#stringresources-prose](../reference/documentation.md#stringresources-prose)

- `doc.resources.complete-messages` (judgment): Every default-locale value and every translated value must be natural user-facing text and, when formatted, a complete LMF1 message rather than a fragment meant to be concatenated.
- `doc.resources.access-reason` (judgment): A permission or network-access reason identifies the App action or service involved, what access is needed, and the user benefit. “Network access required” is not sufficient.
- `doc.resources.broad-network-reason` (judgment): A concrete `AccessHost` reason explains in ordinary terms why the named service is contacted. Reasons for `{network_access_local}` and `{network_access_external}` explain why broad local or external connectivity is necessary; the reserved key does not excuse vague prose.
- `doc.resources.error-message` (judgment): An error message says what happened and, when possible, what the person can do next. Do not expose stack traces, codes, or diagnostics as the explanation.
- `doc.resources.notification` (judgment): A notification or status message names its actor, object, scope, and material condition. Do not assemble grammar from separately translated fragments.
- `doc.resources.persistent-keys`: Persistent record identity uses the declared fixed-width binary key codec. Human-readable names may use the package's localized formatted resources; never replace a key with rendered text or make its identity depend on locale.
- `doc.resources.stable-keys`: Localized templates stay in the package StringResources catalog. Changing wording does not rename stored keys. Retained legacy data-name tuples follow the [message-format compatibility rule](../reference/message-format.md#data-and-database-names).

### Codebot workflow

Owner: [documentation.md#codebot-workflow](../reference/documentation.md#codebot-workflow)

- `doc.workflow.inventory` (judgment): Inventory every schema-visible declaration and each document node it emits, including both List halves and every Enumeration or Union member.
- `doc.workflow.properties`: Add only documented custom properties, on the correct owner and List side. Add `DefaultText` only when a useful initial value differs from the ordinary fallback. Give every editable Nullable `NullText`, and `SetText` where its control draws a value answer, even when the text equals the fallback ([standard properties](../reference/documentation.md#standard-properties)).
- `doc.workflow.resources`: Write complete, specific default-locale `StringResources` values and verify every referenced permission or host reason exists.
- `doc.workflow.checks` (judgment): Run the language checker and Studio schema validation. Treat failures as source defects; never patch generated JSON.

### Review checklist

Owner: [documentation.md#review-checklist](../reference/documentation.md#review-checklist)

- `doc.review.properties`: Every custom property is defined by a consuming contract, placed on the right owner, and follows the same prose rules. `DefaultText`, `TrueText`, `FalseText`, `NullText`, and `SetText` values are non-empty single lines. Omit properties whose intended text exactly matches the ordinary localized fallback, except the `NullText` and `SetText` an editable Nullable must declare; never fabricate personal or sensitive content to fill a default.

### Developer function-comment invariants

Owner: [function-comments.md#developer-function-comment-invariants](../reference/function-comments.md#developer-function-comment-invariants)

- `fn.standalone` (judgment): It must stand on its own; supporting text derived from types and attributes cannot repair an incomplete or ambiguous function comment.
- `fn.both-contracts` (judgment): A public function must satisfy both contracts; this page adds discovery completeness and developer attribution rather than relaxing the ordinary documentation rules.
- `fn.discovery.embedding`: Plaza calculates exactly one discovery embedding for each public function. Its only natural-language input is the function comment's display name and description. Arguments, types, protocols, charts, attributes, package text, and generated appendices are not added. Consequently, every fact needed to discover the function must appear naturally and truthfully in this comment.

### Source ownership and extraction

Owner: [function-comments.md#source-ownership-and-extraction](../reference/function-comments.md#source-ownership-and-extraction)

- `fn.source.authoritative` (judgment): The source comment is authoritative. Never edit generated schema or localized document JSON to change it.
- `fn.source.required`: Every public function must have a non-empty documentation comment.
- `fn.source.display-name` (judgment): The first ordinary text line is the display name. It must be a concise name, not a sentence or keyword list.
- `fn.source.purpose-sentence` (judgment): The remaining ordinary text is the description. Its first sentence must state the function's main purpose in terms an ordinary user understands.
- `fn.source.comment-syntax`: Rust uses outer rustdoc (`///`) on the function. TypeScript uses a JSDoc block (`/** ... */`) on the function. Language-specific attributes keep their required syntax but are not a substitute for the description.
- `fn.source.same-patch` (judgment): When behavior changes, the source comment must change in the same patch.

### Developer attribution invariant

Owner: [function-comments.md#developer-attribution-invariant](../reference/function-comments.md#developer-attribution-invariant)

- `fn.attribution.required`: Every codebot-generated public-function description must end with a developer attribution so both developer names participate in the function's search text. Use this exact natural-language shape when the developer has a Plaza account:

  ```text
  Developed by Display Name (account_name).
  ```

- `fn.attribution.names` (judgment): `account_name` is the immutable public Plaza account name and `Display Name` is the current public profile display name. A codebot must obtain both from the developer profile and must not substitute a GitHub login, company name, email, package name, or fully qualified name.
- `fn.attribution.no-account`: Before the developer has a Plaza account, the display name may be omitted. Use the developer identity declared by the package:

  ```text
  Developed by account_name.
  ```

- `fn.attribution.ordinary-text`: The attribution is ordinary description text, not a custom document property.
- `fn.attribution.refresh` (judgment): If the public display name changes, regenerate the affected function comments before the next publication. Email never belongs in a function comment or search document.

### Completeness invariant

Owner: [function-comments.md#completeness-invariant](../reference/function-comments.md#completeness-invariant)

- `fn.complete.categories` (judgment): The description must include every user-relevant capability declared or implemented by the function. Include each applicable category and omit categories that do not apply:
- `fn.complete.outcome` (judgment): The outcome or problem the function addresses.
- `fn.complete.setup-choices` (judgment): Choices the user makes during setup, including the meaningful scope of each choice.
- `fn.complete.later-actions` (judgment): Actions the user can take after setup, especially manual controls and overrides.
- `fn.complete.automatic` (judgment): Actions performed automatically and the conditions or information that influence them.
- `fn.complete.information` (judgment): Information the function shows, records, compares, or summarizes, including meaningful graphs and histories.
- `fn.complete.delivered` (judgment): Reminders, warnings, reports, or other information delivered to people.
- `fn.complete.limits` (judgment): Important prerequisites or limits that affect whether the function is suitable for the user's need.
- `fn.complete.direct` (judgment): The function comment must express these capabilities directly. A codebot must not assume that a reader or a search model will reconstruct them from argument names, referenced types, attributes, or generated appendices.

### Actor, action, and scope invariant

Owner: [function-comments.md#actor-action-and-scope-invariant](../reference/function-comments.md#actor-action-and-scope-invariant)

- `fn.actor.user-chooses` (judgment): Use **“The user chooses ...”** for setup choices.
- `fn.actor.user-can` (judgment): Use **“The user can ...”** for direct actions and later adjustments.
- `fn.actor.automatic` (judgment): Name the application or domain actor for automatic behavior, such as **“The sprinkler adjusts ...”** or **“The monitor warns ...”**.
- `fn.actor.object-scope` (judgment): State the action's object and scope, such as **“for each zone”**, **“for the selected patient”**, or **“across all study samples”**.
- `fn.actor.condition` (judgment): State a material condition when it distinguishes the capability, such as **“based on recent rain and forecast weather”**.
- `fn.actor.separate-sentence` (judgment): Use a separate sentence when the actor, capability, or modality changes.
- `fn.actor.active-voice` (judgment): Do not use passive constructions such as “chosen by the user” when active voice is available. Do not use an unclear pronoun such as “it” where a reader or model could confuse a user action with automatic behavior. Preserve meaningful modality: “can,” “must,” and “automatically” are not interchangeable.

### Language invariant

Owner: [function-comments.md#language-invariant](../reference/function-comments.md#language-invariant)

- `fn.language.user-goals` (judgment): Describe user goals and observable behavior, not internal architecture.
- `fn.language.concrete` (judgment): Use concrete domain nouns and verbs. Prefer “shows water use by day” over “provides data.”
- `fn.language.synonyms` (judgment): Include natural synonyms only when they are names people genuinely use for the capability. Do not append keywords or repeat a sentence to influence search.
- `fn.language.no-unprovided` (judgment): Do not mention a capability that the function does not provide.
- `fn.language.no-negations` (judgment): Do not list unrelated unsupported capabilities as negations. A semantic match may still associate the function with the negated words.
- `fn.language.no-identifiers` (judgment): Do not expose source identifiers, fully qualified names, attributes, wire details, or data-structure terminology in user-facing prose.
- `fn.language.concise-complete` (judgment): Keep the description concise, but never shorten it by dropping a material user choice, direct control, automatic behavior, result, or limit.

### Codebot workflow

Owner: [function-comments.md#codebot-workflow](../reference/function-comments.md#codebot-workflow)

- `fn.workflow.inspect` (judgment): Inspect the function implementation, arguments, referenced types, user interactions, graphs, notifications, permissions, relevant constraints, and current developer profile.
- `fn.workflow.verify` (judgment): Verify every statement against source or validated schema evidence. Do not invent behavior from a name or sample.
- `fn.workflow.compare` (judgment): Compare the final comment with the inventory and restore any material fact lost during editing.
- `function-comment.attribution.current-identity`: Append the required developer attribution using the current Plaza account and display names, or the package developer identity before an account exists.
- `function-comment.generated-document.not-source`: Run the normal language and schema checks. Treat a generated document as an output to inspect, never as the source to patch.

### Review checklist

Owner: [function-comments.md#review-checklist](../reference/function-comments.md#review-checklist)

- `fn.review.translatable` (judgment): A translation can preserve every actor, action, scope, condition, and modality without guessing.

### Source documentation

Owner: [presentation.md#source-documentation](../reference/presentation.md#source-documentation)

- `doc.plain-comments-ignored`: Plain code comments are ignored by document extraction.

### AI tool-call descriptions

Owner: [presentation.md#ai-tool-call-descriptions](../reference/presentation.md#ai-tool-call-descriptions)

- `docs.comments.no-assistant-guidance`: The Hub appends the tool-call [value-representation hints](../reference/llm-document-transformation.md#value-representation-hints) to AI tool parameter and response descriptions, from the effective schema attributes, including inherited attributes, and built-in date/time types, and then the field's declared [answer labels](../reference/documentation.md#answer-labels-in-ai-descriptions). AI Import uses the same list. Authored descriptions remain intact. App authors and CodeBots do not repeat this guidance in source comments. Keep comments focused on domain meaning, scope, consequences, and any App-specific limits. This does not remove the requirement for a natural label and complete description on every authored node.

### 10. Localized package documents

Owner: [schema.md#10-localized-package-documents](../reference/schema.md#10-localized-package-documents)

- `docs.custom-property.closed-ownership`: All standard properties are flattened into their owning document objects and translated like other custom properties. They are invalid on other owners; values contain no carriage return or newline after normal parser whitespace removal. Omit a property when its intended text exactly matches the ordinary localized fallback. See [standard properties](../reference/documentation.md#standard-properties) for authoring, initialization, and the distinction between null and other values.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Source and output invariants

Owner: [documentation.md#source-and-output-invariants](../reference/documentation.md#source-and-output-invariants)

- `doc.output.type-member`: A field document Type is a non-negative integer schema-node identifier

### Custom document properties

Owner: [documentation.md#custom-document-properties](../reference/documentation.md#custom-document-properties)

- `doc.property.string-value`: Every document property value is a string
- `doc.property.reserved-names`: DisplayName, Description, List, StringResources, Strings, Enum and DocumentVersion are reserved document property names

### Standard properties

Owner: [documentation.md#standard-properties](../reference/documentation.md#standard-properties)

- `doc.boolean.labels`: `[TrueText]` and `[FalseText]` are optional Boolean field labels, each one non-empty line
- `doc.defaulttext.placement`: `[DefaultText]` is one non-empty line, only on an effective String or EndpointServer field or on a function
- `doc.nullable.texts`: `[NullText]` and `[SetText]` are one non-empty line each, only on a nullable field
- `doc.nullable.required-texts`: An editable Nullable declares `[NullText]`, and `[SetText]` wherever its control draws a value answer, even when the text equals the fallback

<!-- /generated -->
