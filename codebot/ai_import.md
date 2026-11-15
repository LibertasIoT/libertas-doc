# Designing AI Import

Load this card when a page's values come from a document: declaring
`LLMDocTrans` or `LLMDocField`, a classifier, validation facts or blank values,
or adding a field to an importing page. Its rules are generated from the
[AI design obligations](../reference/app-contract.md#ai-access-in-every-design-step),
the [document transformation contract](../reference/llm-document-transformation.md)
and the [LLM request contract](../reference/llm-client-requests.md), which own
them. The [annotation inventory](schema_annotations.md) gives each
annotation's Rust and TypeScript spelling.

## Order of work

1. Decide which fields are import targets, and keep values only the person
   supplies outside what an import replaces (`app.import.*`,
   `doctrans.nullable.*`).
2. Declare the form or classifier and its arguments (`doctrans.form.*`,
   `doctrans.action.*`, `doctrans.args.*`, `doctrans.validation.*`,
   `doctrans.classifier.*`).
3. Select the result fields and Union variants (`doctrans.field.*`,
   `doctrans.union.*`, `doctrans.protected.*`).
4. Declare each selected field's blank behavior (`doctrans.blank.*`).
5. Check List, array and categorized imports and the limits
   (`doctrans.array.*`, `doctrans.categorized.*`, `doctrans.request.*`,
   `doctrans.result.*`, `doctrans.schema.*`, `doctrans.import.*`, `llm.*`),
   then compile with Studio and fix every reported rule.

<!-- generated:codebot-rules task=ai-import -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### AI access in every design step

Owner: [app-contract.md#ai-access-in-every-design-step](../reference/app-contract.md#ai-access-in-every-design-step)

- `app.import.printed-fields` (judgment): Every field whose value is printed on that document or its attached statements is an import target, including fields in sections that appear only conditionally.
- `app.import.records-not-targets` (judgment): A value that only the person's own records hold is not an import target; its description says where to find it.
- `app.import.classify-new-fields` (judgment): A new field on an importing page is classified when it is added.
- `app.import.preserve-person-data`: Nothing an import can set may hold data only the person supplies: a composite on an import path is not Nullable, because a null result replaces it, and a Union whose variant the import chooses carries only document values, because a variant change discards the old payload; keep person-supplied values in a record the import does not select.

### Request

Owner: [llm-document-transformation.md#request](../reference/llm-document-transformation.md#request)

- `doctrans.nullable.domain-model` (judgment): Nullable must reflect the form's actual data model. App authors must not make a destination Nullable solely to accommodate document recognition.

### App Schema authoring

Owner: [llm-document-transformation.md#app-schema-authoring](../reference/llm-document-transformation.md#app-schema-authoring)

- `doctrans.action.per-occurrence`: Each occurrence of that struct in an editable Request or SubscriptionRequest gets its own **AI Import** action, except that [categorized layout](../reference/presentation.md#categorized-layout) centralizes the actions of its classifier subtree. A screen can therefore contain several independent forms, including repeated rows of one form type. Selected files are temporary editor input; they do not require a Bytes field in the App Request. When a form's row is promoted or suppressed, its action follows the [row action placement rule](../reference/presentation.md#actions-on-a-promoted-or-suppressed-row).
- `doctrans.args.instruction`:

  | Argument | Meaning |
  | --- | --- |
  | `instruction` | Required App-authored task text. Tell the model what to read, what a genuinely blank box means, and when to report an error. The Hub sends it separately from facts and result schema, with no separate instruction-size limit. |

- `doctrans.args.multiple-inputs` (judgment):

  | Argument | Meaning |
  | --- | --- |
  | `accept_multiple_inputs` | Optional Boolean, default `false`. One file is the normal case. Set `true` only when a logical document may span several selected files; the Hub limit is eight. |

- `doctrans.field.allow-empty-default`: The `allowEmpty` default is `true` for a non-Nullable Number and `false` for String, Boolean, and Enumeration.
- `doctrans.field.recursive-selection`: Within each selected Table, including a List item Table, marking one or more direct children selects **only those children** for that Table's result object. If none of that Table's direct children is marked, all its unprotected children are selected. Mark each intermediate composite whose descendants are selected. A nested scalar marker also sets its blank rule.
- `doctrans.union.variant-choice`: A selected Union imports its variant choice, and a variant's payload fields **only when they are marked**. The all-children default above does not apply to Union payloads: a variant's payload usually holds facts the chosen option leads to, not values printed on the document.
- `doctrans.union.marked-variants`: Marking one or more direct variants of a selected Union limits its result schema to those variants; with no marked variant, all unprotected variants remain available, subject to ordinary choice constraints. Marked variants must be capable of producing a result.
- `doctrans.union.single-value-payload`: A single non-Table value, such as a Rust tuple variant `Allocated(i64)`, has no field to mark; that value is the variant's one payload field, named by the variant, and it is **always marked**. A Nil (unit) variant has no payload field.
- `doctrans.protected.output-exclusion`: **Hidden, Fixed and [Secret](../reference/schema.md#43-value-constraint-and-presentation-attributes) fields are excluded from LLM output**, including their complete subtrees; together they are protected content.
- `doctrans.result.schema-limits`: This feature adds no arbitrary Table depth, List length, or integer-width restriction; actual schema and wire limits apply, including the 64 KiB output-schema limit. A configured provider that rejects a schema returns a normal request error.
- `doctrans.validation.hidden-facts`: The selected validation Table may be Hidden and may receive its value through normal Request construction, CopyFrom, or EditRequest. Its children may be complete typed values using the same recursive JSON projection below. An absent or incomplete required value blocks the action. Declared Nullable facts retain their null meaning.
- `doctrans.validation.fact-names`: Fact names need no relationship to result names: the W2 example verifies `year` without returning `year`. The model checks those facts against the document; the Hub does not independently verify them.

### Blank values and output types

Owner: [llm-document-transformation.md#blank-values-and-output-types](../reference/llm-document-transformation.md#blank-values-and-output-types)

- `doctrans.blank.number-string`: A non-Nullable Number normally uses a **JSON string** result because the form's box might be blank even though the App's data field is required. The model returns printed numeric text for a legible amount and exact `""` for a clearly blank box. The client strictly converts nonempty text to the destination Number type. Exact `""` applies the field's effective, typed Schema `Default`. Thus `tips: number` with `@LibertasDefault(0)` stays required and stores zero for a blank tips box. The schema sent to the model does not contain a JSON Schema `default` keyword or expose the App's Default value.
- `doctrans.blank.string-nonempty`: A String with default `allowEmpty=false` requires a nonempty result.
- `doctrans.blank.enumeration-names`: An Enumeration uses a JSON string for the printed codes described in the [conversion table](../reference/llm-document-transformation.md#complete-json-projection-and-client-conversion). The client maps an exact offered code to its declared `Fixed` integer value, or to its ordinal when no `Fixed` is declared. The model never invents a code from its meaning or supplies an arbitrary integer. Explicit `allowEmpty=true` also permits exact `""` and applies the effective Enumeration Default.
- `doctrans.blank.no-invented-default` (judgment): If a Number has no meaningful Default, an App author may set `allowEmpty=false` only when the form establishes that its box cannot be blank; its output then uses JSON `integer` or `number` as appropriate and a blank box is an error. Otherwise, leave that field out of automatic extraction until its blank semantics are known. Do not invent a zero Default for this purpose.
- `doctrans.authoring.nullable-domain-choice`: Nullable remains a domain-model choice, never a document-recognition trick. CodeBots must not make a field Nullable solely to support this action.
- `doctrans.nullable.null-clears`: A truly Nullable result permits JSON `null` for a genuinely blank value; `null` clears the destination. A non-null Nullable Table uses its selected child object shape; a non-null Nullable List uses its selected item shape. A Nullable Number uses JSON string or null and converts its non-null numeric text strictly. Nullable Enumeration uses an offered printed code or null.

### Declared blank value

Owner: [llm-document-transformation.md#declared-blank-value](../reference/llm-document-transformation.md#declared-blank-value)

- `doctrans.blank.declared-value`: `LLMDocField(blank=VALUE)` declares the value that an exact `""` result applies when the field's box is blank on the document. `VALUE` is spelled exactly like a `Default` literal for that field: `$null` on a Nullable field, a number literal on a Number, a quoted string, an Enumeration member name, or a Union variant name, with `$NAME` constant references wherever Default allows them. For example, `blank=$null` clears a Nullable field when its box is blank, `blank=0` stores zero, and `blank=ShortTerm` on a Union chooses that variant. The portable `LLMDocField` attribute value is `blank:` followed by the literal in its portable Default encoding, such as `blank:$null`, `blank:0`, or `blank:ShortTerm`; the other portable values are `auto` for the bare marker, `true`, and `false`.
- `doctrans.blank.default-shorthand`: `allowEmpty=true` remains the shorthand for a blank that applies the field's Schema `Default`. A field with neither keeps the rules above. Studio rejects an invalid `blank` at the marker's source location.
- `doctrans.blank.optional-text`: For ordinary optional text whose domain meaning is “blank when not supplied,” use a non-Nullable String with an empty Schema Default and explicit `allowEmpty=true`. In Rust, combine `#[libertas_default("")]` with `#[libertas_llm_doc_field(allow_empty = true)]`; the bare field marker still defaults to `false` for String. Permit length zero in its size bounds. Describe the manual answer as “Leave it empty if absent” and tell the model to return exact `""` for genuinely missing text. Reserve JSON `null` for truly Nullable destinations.
- `doctrans.blank.unreadable-is-error` (judgment): Unreadable text is an error, not evidence that the text is absent. Follow the [type-selection guidance](../reference/app-contract.md#user-facing-and-non-user-facing-types) when manual input and an internal record use different representations.
- `doctrans.import.printed-values`: AI Import reads the printed document. Return code letters as printed, dates as printed, amounts as printed, and each address as one text block with its printed line breaks. Do not expand codes into their meanings, normalize dates or amounts, add an unprinted country, include a telephone number in an address, or repeat a separately selected party name in its address. Keep identifiers as text, including leading zeros. The client owns deterministic conversion to the declared stored types; App instructions identify the printed boxes rather than teach storage encodings.

### Complete JSON projection and client conversion

Owner: [llm-document-transformation.md#complete-json-projection-and-client-conversion](../reference/llm-document-transformation.md#complete-json-projection-and-client-conversion)

- `doctrans.schema.descriptions.authoring`: Supply locating labels and domain import instructions. The model reads printed values; do not require it to interpret a code or convert to a storage representation. The platform owns [schema generation and conversion](../reference/llm-document-transformation.md#complete-json-projection-and-client-conversion).
- `doctrans.conversion.nullable-domain-choice`:

  | Libertas value | Model JSON and client conversion |
  | --- | --- |
  | Nullable | Add null to that value's JSON schema; a returned null clears the typed value. Never add Nullable merely for recognition. |

- `doctrans.schema.no-template`: Display Template presentation, including a `format` pattern, never reaches the model.
- `doctrans.blank.semantic-number-defaults`: Number annotations such as DateOnly, Money, TimeInterval, EnumSource, and Bitflags keep the Number default of `allowEmpty=true`; their typed Default is applied locally for exact `""`. Other semantic scalar types default to `allowEmpty=false` and may explicitly enable it with a meaningful Default.

### Array import

Owner: [llm-document-transformation.md#array-import](../reference/llm-document-transformation.md#array-import)

- `doctrans.array.inside-list`: When the annotated struct occurrence lies anywhere inside an editable List item, directly or through nested Tables and Union payloads, its **AI Import** action imports an array. The struct type of the occurrence the action is on decides what is imported: only forms of that type.
- `doctrans.array.duplicate-instruction` (judgment): The App's `instruction` names what a duplicate looks like for its form (for example a tax form's Copy B, Copy C and Copy 2 printed on one page); the client's appended text stays generic.
- `doctrans.array.single-outside-list`: A struct occurrence outside any List, or inside a List whose membership is ReadOnly, keeps the single import.

### Declared duplicate identities

Owner: [llm-document-transformation.md#declared-duplicate-identities](../reference/llm-document-transformation.md#declared-duplicate-identities)

- `doctrans.duplicate-identity.equality`: Within the same form and struct position, converted identity values compare as an ordered typed tuple, without text normalization. A whole List compares its complete ordered value, including length and every typed item; a whole Union compares its selected variant and complete typed payload. Changing an item, its order, a variant or its payload makes that component unequal. Null is a value; a null containing Table supplies null for its selected descendant. An unanswered local identity value does not equal a supplied null answer. An appended form whose tuple equals an existing item's or an earlier imported form's is one duplicate, even when other imported values differ. Keep the existing or first imported paper unchanged and include the skipped copy in the review's duplicate count. Without a declaration, retain exact imported-value equality. The first result of an array import still replaces its current occurrence; all further elements and categorized imports use this duplicate rule.

### Categorized import

Owner: [llm-document-transformation.md#categorized-import](../reference/llm-document-transformation.md#categorized-import)

- `doctrans.classifier.definition`: **Classifier rule.** An `LLMDocTrans` Table is a **classifier** if and only if all its variants are `LLMDocTrans` structs. Its variants are its import positions: each selected field, each selected List's item, and, in an item Table, its one marked form or each variant of its one marked Union (the item's other fields, such as an owner, keep their normal initialization). A variant may itself be a classifier, so classifiers nest, typically one root classifier with one classifier per **category**. Every other `LLMDocTrans` Table is a **form**, with the meaning described above; an `LLMDocTrans` a form encloses keeps only its own action.
- `doctrans.classifier.no-marked-child`: Selection is the ordinary rule above (the marked direct children, or all unprotected children when none is marked); a form's requirement to mark at least one direct child does not apply to a classifier.
- `doctrans.classifier.instruction` (judgment): **Arguments.** On a classifier, `instruction` (required) says what counts as one document, what to leave out, and how to recognize duplicate copies; the client appends the generic classification text below. `doc_type` filters the selectable files as for a form.
- `doctrans.classifier.file-selection`: A classifier always accepts several files;
- `doctrans.classifier.form-arguments`: Each form keeps its own `validation`, used when its kind is imported; its `doc_type` and `accept_multiple_inputs` govern only its own action.
- `doctrans.classifier.kind-constraints`: A kind is offered only while the Union's ordinary choice constraints (Hidden, Fixed, `ConstrainedBy`) permit it for a new item at its position. A kind inside a List is a **list kind**, and the nearest List enclosing it is its **category List**. A form that is a classifier's variant itself, outside any List, is a **single-document kind**, such as a passport or a birth certificate: the Request holds one occurrence of it.
- `doctrans.classifier.kind-names` (judgment): A kind is named for the model by its variant's (or, for a one-kind List, its struct's) localized display name and description, with the display names of the classifiers between the importing classifier and the kind. Write them to say exactly what the document is, such as "Form 1099-S for the sale of your main home". The root classifier offers every leaf kind; a category classifier offers its own.
- `doctrans.categorized.apply.authoring`: Rely on the normal [import apply and review flow](../reference/llm-document-transformation.md#categorized-import); importing changes the editor without submitting the App request.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### App Schema authoring

Owner: [llm-document-transformation.md#app-schema-authoring](../reference/llm-document-transformation.md#app-schema-authoring)

- `doctrans.form.declaration`: LLMDocTrans goes on a named Table struct, a form whose direct result children carry LLMDocField
- `doctrans.validation.every-occurrence`: The validation path resolves to a Table in every editable occurrence; calculation-only and protected occurrences need no unused fact
- `doctrans.args.doc-type`: The doc_type filter intersected with PDF, PNG, JPEG and WebP must not be empty
- `doctrans.args.form-date-order`: form_date_order is MonthDayYear or DayMonthYear on a form declaration; a form with no DateOnly result field draws a warning
- `doctrans.field.selection`: LLMDocField marks a direct result child or a descendant of a selected composite
- `doctrans.form.mark-a-child`: At least one direct child of an annotated form root Table must carry LLMDocField
- `doctrans.protected.hidden-fixed-excluded`: LLMDocField directly on Hidden, Fixed or Secret content is invalid
- `doctrans.protected.composite-omitted`: LLM output omits composites containing only protected content; marking one directly is invalid
- `doctrans.validation.no-secret`: An LLMDocTrans validation Table contains no Secret value

### Blank values and output types

Owner: [llm-document-transformation.md#blank-values-and-output-types](../reference/llm-document-transformation.md#blank-values-and-output-types)

- `doctrans.blank.allow-empty-requires-default`: allowEmpty=true requires a real Schema Default on a non-Nullable scalar represented as a JSON string
- `doctrans.blank.allow-empty-hosts`: Boolean rejects allowEmpty=true; Table, List and Nil do not use allowEmpty; a Union permits true only with a Default variant
- `doctrans.blank.endpoint`: Endpoint results cannot enable allowEmpty=true
- `doctrans.nullable.no-allow-empty`: allowEmpty is inapplicable to Nullable fields; blank remains applicable

### Declared blank value

Owner: [llm-document-transformation.md#declared-blank-value](../reference/llm-document-transformation.md#declared-blank-value)

- `doctrans.blank.valid-values`: blank needs a selected scalar, Enumeration or Union field and a value valid for it; Bytes and identities take only $null
- `doctrans.blank.not-with-allow-empty`: blank cannot combine with allowEmpty

### Declared form codes

Owner: [llm-document-transformation.md#declared-form-codes](../reference/llm-document-transformation.md#declared-form-codes)

- `doctrans.form-code.declaration`: FormCode is a nonempty code on an Enumeration member, Union variant or nonzero Bitflags flag, unique among its siblings

### Declared duplicate identities

Owner: [llm-document-transformation.md#declared-duplicate-identities](../reference/llm-document-transformation.md#declared-duplicate-identities)

- `doctrans.duplicate-identity.paths`: A form's duplicate_identity names distinct imported scalar or whole List/Union values through self-relative Table paths

### Categorized import

Owner: [llm-document-transformation.md#categorized-import](../reference/llm-document-transformation.md#categorized-import)

- `doctrans.classifier.multiple-files`: A classifier takes no validation path or accept_multiple_inputs = true
- `doctrans.classifier.item-shapes`: A List under a classifier holds a Union of forms, a Table with one marked Union or form, or a form
- `doctrans.classifier.placement`: A classifier is used in an editable Request or SubscriptionRequest and has at least one kind
- `doctrans.classifier.readonly-list-and-single-kinds`: Studio refuses a ReadOnly classifier List and warns when a single-document kind encloses import declarations

### Document duplicate identity metadata

Owner: [schema.md#document-duplicate-identity-metadata](../reference/schema.md#document-duplicate-identity-metadata)

- `attr.llmdoctrans.config-shape`: LLMDocTrans config has four ordinary keys, an optional duplicate_identity string and an optional form_date_order string

<!-- /generated -->
