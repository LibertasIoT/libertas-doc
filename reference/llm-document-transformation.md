# LLM document transformation

LLM document transformation is also called **AI Import**.

A Control client can ask the Hub's configured LLM connector to transform one
or more documents into a JSON value. Each request has one response sent to the
Control session that submitted it. The response repeats the caller's unsigned
64-bit transaction ID, so a client can match concurrent requests without using
the text conversation history.

<!-- rule: doctrans.admission.connector-support class=hard enforced=no tasks=client-runtime -->
Clients offer the action on modern Hubs and hide it on legacy Hubs. The
[Control AI availability rule](protocols.md#ai-navigation-and-ai-search)
applies before choosing documents or transmitting a transformation: an
unavailable action reports an immediate error. The Hub must handle the document
transformation messages and have an OpenAI or Gemini connector whose selected
model accepts the supplied documents and structured output schema. Available
AI does not guarantee those request-specific capabilities; ordinary errors and
timeouts still apply after a request is sent.

## Request

`MSGCtrlLLMDocumentTransformReq` has these fields in wire order:

| Field | Type | Meaning |
| --- | --- | --- |
| `_transactionId` | unsigned 64-bit integer | Caller-chosen correlation ID, returned unchanged. |
| `_documents` | array of `LibertasLLMDocument` | One to eight documents. Each item has one `_bytes` value containing the full [LMB1 envelope](schema.md#mime-aware-bytes). |
| <!-- rule: doctrans.request.nonempty-instruction class=hard enforced=no tasks=client-runtime --> `_instruction` | string | Required, nonempty client task instruction. No separate instruction-size limit. |
| <!-- rule: doctrans.request.validation-object class=hard enforced=no tasks=client-runtime --> `_validationJson` | string containing a JSON object | Facts the documents must confirm. Use `"{}"` when there are no facts. At most 64 KiB. |
| `_outputSchema` | string containing a JSON Schema object | Requested result shape for the model and client. At most 64 KiB. |

<!-- rule: doctrans.request.document-limits class=hard enforced=no tasks=client-runtime -->
The combined encoded documents must not exceed 8 MiB. This operation accepts
LMB1 values with `application/pdf`, `image/png`, `image/jpeg`, or `image/webp`
MIME types. The MIME type must agree with the file signature. The LMB1 value
contains the media bytes; the client does not send a path or file URL.
The Hub allows up to four pending transformations and limits each provider
response body to 2 MiB.

<!-- rule: doctrans.request.facts-versus-shape class=hard enforced=no tasks=client-runtime -->
The task instruction says **what** to extract or transform. The validation
JSON supplies values that must be checked against the documents. The schema
says **which JSON shape** the result must have. These are separate fields;
clients do not interpolate placeholder tags into the instruction.

For example, a client can use this instruction:

<!-- rule: doctrans.extraction.no-guessing class=hard enforced=no tasks=client-runtime -->
> Read the attached employment record. Extract the employee name exactly as
> shown. Do not guess missing or illegible values.

It can send `{"year":"2026"}` as the
validation JSON, and this output schema:

```json
{
  "type": "object",
  "properties": {
    "employee_name": { "type": "string" }
  },
  "required": ["employee_name"],
  "additionalProperties": false
}
```

Validation facts need not be returned as result fields. In this example, the
Hub asks the model to verify `year` from the document and returns only the
employee name when that check succeeds.

<!-- rule: doctrans.transaction.exact-64bit class=hard enforced=no tasks=client-runtime -->
`_validationJson` and `_outputSchema` are serialized **strings** in the
Control message. The examples above show the JSON values before string
serialization. The client must preserve the 64-bit transaction ID without
rounding it through a floating-point number.

<!-- rule: doctrans.request.validate-json-objects class=hard enforced=no tasks=client-runtime -->
The Hub requires `_validationJson` and `_outputSchema` to parse as JSON objects.
It supplies the schema through the configured provider's structured-output
mode: OpenAI uses `json_schema` with `strict: true`, and Gemini uses a
schema-constrained JSON response format. OpenAI requires every object property
in `required` and `additionalProperties: false`; represent a genuinely
Nullable property with a required `type: [T, "null"]`. A schema outside a
provider's supported subset can cause a provider error. The Hub does not
interpret JSON Schema keywords or enforce the schema against the returned
result. The client must check a successful result against its supplied schema
before using it.

<!-- rule: doctrans.nullable.domain-model class=judgment enforced=no tasks=ai-import -->
Nullable must reflect the form's actual data model. App authors must not make
a destination Nullable solely to accommodate document recognition.

## Response and validation

`MSGCtrlLLMDocumentTransformRsp` contains `_transactionId`, `_json`, and
`_error`. On model-reported success, `_error` is empty and `_json` is its
serialized JSON result. On failure, `_json` is empty and `_error`
explains the failure. There is no partial result. A malformed request,
unsupported media type, provider failure, or malformed model response envelope
also produces an error response when the Control session remains connected.

The Hub instructs the model to use the documents as evidence, to check every
validation value, and to report a conflict or unverifiable value as an error.
The Hub forwards the model's JSON result or error without independently
checking the result fields, output schema, or validation facts. Provider
schema mode constrains the supported structure; it does not establish that the
model read the document correctly. The client checks the result shape and
applies a valid result to the editable fields; the user can inspect or edit the
filled Request before submitting it.

The request does not read or change the client's text conversation. Document
transformations can be submitted concurrently and are correlated by the
transaction ID within the originating Control session.

## App Schema authoring

<!-- rule: doctrans.form.declaration class=hard enforced=yes tasks=ai-import title="LLMDocTrans goes on a named Table struct, a form whose direct result children carry LLMDocField" -->
An App declares one document action on a named Table struct with
`LLMDocTrans`. Such a struct is a **form**: mark its direct result children with
`LLMDocField`. <!-- /rule --> A Table whose variants are all `LLMDocTrans` is
a [classifier](#categorized-import) instead, which sorts a pile of documents
into the forms it encloses. <!-- rule: doctrans.action.per-occurrence class=hard enforced=no tasks=ai-import -->
Each occurrence of that struct in an editable Request or SubscriptionRequest
gets its own **AI Import** action, except that
[categorized layout](presentation.md#categorized-layout) centralizes the actions
of its classifier subtree. A screen can therefore contain
several independent forms, including repeated rows of one form type. Selected
files are temporary editor input; they do not require a Bytes field in the App
Request. When a form's row is promoted or suppressed, its action follows the
[row action placement rule](presentation.md#actions-on-a-promoted-or-suppressed-row). <!-- rule: doctrans.action.unavailable-contexts class=hard enforced=no tasks=client-runtime -->
The action is unavailable in internal calculation requests and inside Hidden,
Fixed, CacheIdentity, or ReadOnly subtrees. ReadOnly on a List restricts its
membership, not editing an existing item. <!-- /rule --> <!-- rule: doctrans.validation.every-occurrence class=hard enforced=yes tasks=ai-import title="The validation path resolves to a Table in every editable occurrence; calculation-only and protected occurrences need no unused fact" --> Studio checks the `validation` path
for every editable occurrence of the annotated Table; a missing or non-Table
target in any such occurrence is a schema error. Calculation-only and protected
occurrences do not need an otherwise unused validation fact.

| Argument | Meaning |
| --- | --- |
| <!-- rule: doctrans.args.instruction class=hard enforced=no tasks=ai-import --> `instruction` | Required App-authored task text. Tell the model what to read, what a genuinely blank box means, and when to report an error. The Hub sends it separately from facts and result schema, with no separate instruction-size limit. |
| `validation` | Optional [Libertas Schema Path](schema.md#context-and-syntax) to a Table value in the current Request. `$` is the Request root, `@` is this struct occurrence, and `^` follows actual value-tree parents. The Table's **children**, without its own name, become the validation JSON object. Omission sends `{}`. |
| <!-- rule: doctrans.args.doc-type class=hard enforced=yes tasks=ai-import title="The doc_type filter intersected with PDF, PNG, JPEG and WebP must not be empty" --> `doc_type` | Optional [MIME filter](schema.md#mime-aware-bytes) for selected files. The effective source choices are its intersection with PDF, PNG, JPEG, and WebP; an empty intersection is invalid. Omission accepts all four. Prepared images are sent as JPEG even if selected as PNG or WebP. |
| <!-- rule: doctrans.args.multiple-inputs class=judgment enforced=no tasks=ai-import --> `accept_multiple_inputs` | Optional Boolean, default `false`. One file is the normal case. Set `true` only when a logical document may span several selected files; the Hub limit is eight. |
| `duplicate_identity` | Optional comma-separated self-relative Schema Paths naming the printed value components that together identify one paper, for example `"@.payer_tin,@.recipient_tin,@.amount"`. See [declared duplicate identities](#declared-duplicate-identities). |
| <!-- rule: doctrans.args.form-date-order class=hard enforced=yes tasks=ai-import title="form_date_order is MonthDayYear or DayMonthYear on a form declaration; a form with no DateOnly result field draws a warning" --> `form_date_order` | Optional printed date order of the form: `"MonthDayYear"` or `"DayMonthYear"`, with no default. Every DateOnly field of that form's result, including nested Tables, Lists and Union payloads, reads a numeric printed date with a four-digit year in this order; see [printed dates](#complete-json-projection-and-client-conversion). Declare it once for a form that always prints one order, for example `form_date_order = "MonthDayYear"` on a US form. Any other value is a schema error; a form whose result has no DateOnly field draws a warning, because the declaration has no effect. <!-- /rule --> |

`LLMDocField` has optional `allowEmpty` or `blank`, described under
[blank values](#blank-values-and-output-types). <!-- rule: doctrans.field.allow-empty-default class=hard enforced=no tasks=ai-import --> The `allowEmpty` default is
`true` for a non-Nullable Number and `false` for String, Boolean, and
Enumeration. <!-- rule: doctrans.field.selection class=hard enforced=yes tasks=ai-import title="LLMDocField marks a direct result child or a descendant of a selected composite" --> A marker
on a **direct child** of the annotated Table selects that result field. <!-- /rule --> <!-- rule: doctrans.field.recursive-selection class=hard enforced=no tasks=ai-import --> Within
each selected Table, including a List item Table, marking one or more direct
children selects **only those children** for that Table's result object. If
none of that Table's direct children is marked, all its unprotected children
are selected. Mark each intermediate composite whose descendants are selected.
A nested scalar marker also sets its blank rule.

<!-- rule: doctrans.union.variant-choice class=hard enforced=no tasks=ai-import -->
A selected Union imports its variant choice, and a variant's payload fields
**only when they are marked**. The all-children default above does not apply to
Union payloads: a variant's payload usually holds facts the chosen option leads
to, not values printed on the document. <!-- rule: doctrans.union.marked-variants class=hard enforced=no tasks=ai-import --> Marking one or more direct variants of
a selected Union limits its result schema to those variants; with no marked
variant, all unprotected variants remain available, subject to ordinary choice
constraints. Marked variants must be capable of producing a result.

A variant's payload is either a Table or a single non-Table value. A Table
payload has the same meaning however it is declared (a Rust struct variant
`V { a, b }`, a Rust `W2(W2)`, or a TypeScript interface member): its fields
import only when marked, as above. <!-- rule: doctrans.union.single-value-payload class=hard enforced=no tasks=ai-import --> A single non-Table value, such as a Rust
tuple variant `Allocated(i64)`, has no field to mark; that value is the
variant's one payload field, named by the variant, and it is **always marked**.
A Nil (unit) variant has no payload field.

- When at least one variant has a marked payload field, the result is a
  `variant`/`value` wrapper. `value` contains that variant's marked payload
  fields, following the Table selection rule inside them, and is an empty
  object for a variant with none marked.
- When no variant has a marked payload field, the Union is **variant-only**
  and is projected and converted exactly as an **Enumeration** of its
  available variant names, so a printed code can be imported without the model
  answering the facts that code leads to.

<!-- rule: doctrans.union.same-or-new-variant class=hard enforced=no tasks=client-runtime -->
When the chosen variant equals the current one, the client keeps the variant
and every payload value the result does not supply, in its current state:
an unanswered field stays unanswered and is validated as it was before the
import, so it does not reject the proposal. A different variant is
applied exactly as if the user had chosen it in the editor: its unmarked
payload fields receive normal construction and initialization, and required
ones without a value stay unanswered. The proposal is not rejected for them.
The result is the same state a manual choice of that variant produces:
incremental validation runs as it does after that choice, and every rule a
manual choice would defer (mandatory values, final-only ValidationRules, and
cross-field rules that read an unanswered field) is deferred to Submit's final
validation, which requires them as usual. The editor shows the unanswered
fields for the user to answer. <!-- rule: doctrans.form.mark-a-child class=hard enforced=yes tasks=ai-import title="At least one direct child of an annotated form root Table must carry LLMDocField" -->
At least one direct child of a form's annotated root Table must be marked. Normal editor construction supplies
the bound struct occurrence before the action runs.

**Every supported Libertas value type is supported in result subtrees**,
including nested Tables and Lists, Enumerations, Unions, Dates, money, dynamic
EnumSource, other semantic scalars, and their declared Nullable forms. Hidden,
ReadOnly, Fixed and Secret remain ordinary field attributes. <!-- rule: doctrans.protected.output-exclusion class=hard enforced=no tasks=ai-import --> **Hidden, Fixed and
[Secret](schema.md#43-value-constraint-and-presentation-attributes) fields are
excluded from LLM output**, including their complete subtrees; together they
are protected content. <!-- /rule --> <!-- rule: doctrans.protected.hidden-fixed-excluded class=hard enforced=yes tasks=ai-import title="LLMDocField directly on Hidden, Fixed or Secret content is invalid" --> Placing
`LLMDocField` directly on any of them is invalid. <!-- /rule --> <!-- rule: doctrans.result.required-unavailable class=hard enforced=no tasks=client-runtime --> A selected composite may contain
them: omit those properties from the generated output schema and preserve their
normal values when updating the existing record. New List rows or Union payloads
receive them through normal field initialization, such as Fixed, Default, or
the existing construction rules. If a required value remains unavailable,
normal validation rejects the proposal, except for the unmarked payload fields
of a newly chosen Union variant, described above. The model never supplies it. ReadOnly
results retain their normal visible presentation.
For an existing Table or the same Union variant, omitted fields keep their
current typed values, including visible fields that were outside the result
schema. A newly selected Union variant initializes omitted payload fields by
the normal construction rules. <!-- rule: doctrans.list.new-rows class=hard enforced=no tasks=client-runtime --> An extracted List creates new rows through normal initialization; it must not
copy Hidden identities from old rows by position. <!-- rule: doctrans.protected.composite-omitted class=hard enforced=yes tasks=ai-import title="LLM output omits composites containing only protected content; marking one directly is invalid" --> A composite containing only
protected content is omitted, including a List with a protected item type.
Marking such a field directly is invalid; no null placeholders are generated.

The client derives standard JSON Schema recursively from the selected result
fields and only the types reachable from them: object properties, arrays and
items, enum choices, variant alternatives, null, and references for recursive
types. It does not send the App's full schema. Declared constraints still apply. <!-- rule: doctrans.result.schema-limits class=hard enforced=no tasks=ai-import -->
This feature adds no arbitrary Table
depth, List length, or integer-width restriction; actual schema and wire limits
apply, including the 64 KiB output-schema limit. A configured provider that
rejects a schema returns a normal request error.

The following TypeScript source illustrates independent W2 and W4 actions:

```typescript
interface W2Facts { year: string; }

/**
 * @LibertasLLMDocTrans(validation="@.expected", instruction="Read the W2 exactly. Check the supplied facts against the form. Return an empty string for a clearly blank tips box; return decimal numeric text for a legible amount. Report illegible values as errors.", doc_type="application/pdf,image/*")
 */
interface W2 {
    /** @LibertasHidden */
    expected: W2Facts;
    /** @LibertasLLMDocField */
    employee_name: string;
    /** @LibertasNumber(type=f64)
     * @LibertasDefault(0)
     * @LibertasLLMDocField */
    tips: number;
    /** @LibertasSize(max=16)
     * @LibertasLLMDocField */
    boxes: W2Box[];
}

interface W2Box {
    label: string;
    /** @LibertasNumber(type=f64)
     * @LibertasDefault(0) */
    amount: number;
}

/** @LibertasLLMDocTrans(instruction="Read the W4 exactly. Return an empty string for a clearly blank dependents box. Report illegible values as errors.") */
interface W4 {
    /** @LibertasNumber(type=u16)
     * @LibertasDefault(0)
     * @LibertasLLMDocField */
    dependents: number;
}

/** @LibertasRequest */
interface TaxRequest { w2: W2; w4: W4; }
```

The Rust spellings are `#[libertas_llm_doc_trans(instruction = "...",
validation = "@.expected", doc_type = "image/*",
accept_multiple_inputs = true)]` on the struct and
`#[libertas_llm_doc_field(allow_empty = true)]` or
`#[libertas_llm_doc_field(blank = 0)]` on a direct field. Omit the
field argument to use its type-dependent default. A marker on a nested scalar
inside a selected composite overrides only that scalar's blank rule. Rust and TypeScript emit the
same portable Schema attributes.

<!-- rule: doctrans.validation.hidden-facts class=hard enforced=no tasks=ai-import -->
The selected validation Table may be Hidden and may receive its value through
normal Request construction, CopyFrom, or EditRequest. Its children may be
complete typed values using the same recursive JSON projection below. An absent
or incomplete required value blocks the action. Declared Nullable facts retain
their null meaning. <!-- rule: doctrans.validation.no-secret class=hard enforced=yes tasks=ai-import title="An LLMDocTrans validation Table contains no Secret value" --> The validation facts are sent to the model, so
Studio rejects a validation Table with a `Secret` value anywhere in its subtree. <!-- /rule --> <!-- rule: doctrans.validation.fact-names class=hard enforced=no tasks=ai-import --> Fact
names need no relationship to result names: the W2 example verifies `year`
without returning `year`. The model checks those facts against the document;
the Hub does not independently verify them.

### Blank values and output types

<!-- rule: doctrans.schema.selected-properties-required class=hard enforced=no tasks=client-runtime -->
Every selected result property, including each selected property in a nested Table, is
required in the generated strict output JSON Schema; Hidden, Fixed and Secret
subtrees are omitted. A List is a JSON array
whose item schema is derived recursively. Every Table object has exactly its
selected children and `additionalProperties: false`; unselected children do
not appear in the model output. The client checks List
length against its Schema bounds, converts every item, and replaces the selected
List as one typed subtree. It merges selected Table fields
into the existing Table or same Union variant as one editor change. <!-- rule: doctrans.blank.number-string class=hard enforced=no tasks=ai-import -->
A non-Nullable Number
normally uses a **JSON string** result because
the form's box might be blank even though the App's data field is required.
The model returns printed numeric text for a legible amount and exact `""`
for a clearly blank box. The client strictly converts nonempty text to the
destination Number type. Exact `""` applies the field's effective, typed
Schema `Default`. Thus `tips: number` with `@LibertasDefault(0)` stays required
and stores zero for a blank tips box. The schema sent to the model does not
contain a JSON Schema `default` keyword or expose the App's Default value.

<!-- rule: doctrans.blank.allow-empty-requires-default class=hard enforced=yes tasks=ai-import title="allowEmpty=true requires a real Schema Default on a non-Nullable scalar represented as a JSON string" -->
`allowEmpty=true` requires a real Schema `Default` and a non-Nullable scalar
whose document representation is a JSON string. An implicit empty initialization or
localized DefaultText does not count. <!-- rule: doctrans.blank.string-nonempty class=hard enforced=no tasks=ai-import --> A String with default `allowEmpty=false`
requires a nonempty result. <!-- rule: doctrans.blank.allow-empty-hosts class=hard enforced=yes tasks=ai-import title="Boolean rejects allowEmpty=true; Table, List and Nil do not use allowEmpty; a Union permits true only with a Default variant" --> Boolean results use JSON Boolean and cannot use
`allowEmpty=true`. Table, List, and Nil fields do not use `allowEmpty`; a
Union may use `allowEmpty=true` when its Schema `Default` names one of its
variants. Fixed
fields are outside the model's result. <!-- rule: doctrans.blank.enumeration-names class=hard enforced=no tasks=ai-import --> An Enumeration uses a JSON string for
the printed codes described in the [conversion table](#complete-json-projection-and-client-conversion). The client maps an exact offered code to its
declared `Fixed` integer value, or to its ordinal when no `Fixed` is declared.
The model never invents a code from its meaning or supplies an arbitrary integer.
Explicit `allowEmpty=true` also permits exact `""` and applies the effective
Enumeration Default. <!-- rule: doctrans.blank.no-invented-default class=judgment enforced=no tasks=ai-import --> If a Number has no meaningful Default, an App author may
set `allowEmpty=false` only when the form establishes that its box cannot be
blank; its output then uses JSON `integer` or `number` as appropriate and a
blank box is an error. Otherwise, leave that field out of automatic extraction
until its blank semantics are known. Do not invent a zero Default for this
purpose.

<!-- rule: doctrans.blank.endpoint class=hard enforced=yes tasks=ai-import title="Endpoint results cannot enable allowEmpty=true" -->
An Endpoint's process-provisioning `Default` does not supply an Endpoint ID, so
Endpoint results cannot enable `allowEmpty=true`.

<!-- rule: doctrans.authoring.nullable-domain-choice class=hard enforced=no tasks=ai-import -->
Nullable remains a domain-model choice, never a document-recognition trick.
CodeBots must not make a field Nullable solely to support this action. <!-- rule: doctrans.nullable.null-clears class=hard enforced=no tasks=ai-import --> A truly
Nullable result permits JSON `null` for a genuinely blank value; `null` clears
the destination. A non-null Nullable Table uses its selected child object shape;
a non-null Nullable List uses its selected item shape. A Nullable Number uses JSON string or null and converts its
non-null numeric text strictly. Nullable Enumeration uses an offered printed code or null. <!-- rule: doctrans.nullable.no-allow-empty class=hard enforced=yes tasks=ai-import title="allowEmpty is inapplicable to Nullable fields; blank remains applicable" -->
`allowEmpty` is inapplicable to Nullable fields; `blank` is not.

##### Nullable fields with no permitted choice

<!-- rule: doctrans.choices.empty-permitted-set class=hard enforced=no tasks=client-runtime -->
When the current constraints leave a Nullable Union, Enumeration, identity, or
EnumSource field with an empty permitted set, its result schema admits exactly
`null` (the model is offered only `null`) and the import proceeds; any other
value is rejected. A non-Nullable field with an empty set refuses the action.

#### Declared blank value

<!-- rule: doctrans.blank.declared-value class=hard enforced=no tasks=ai-import title="LLMDocField blank=VALUE declares, as a Default literal, the value a blank box applies" -->
`LLMDocField(blank=VALUE)` declares the value that an exact `""` result applies
when the field's box is blank on the document. `VALUE` is spelled exactly like
a `Default` literal for that field: `$null` on a Nullable field, a number
literal on a Number, a quoted string, an Enumeration member name, or a Union
variant name, with `$NAME` constant references wherever Default allows them.
For example, `blank=$null` clears a Nullable field when its box is blank,
`blank=0` stores zero, and `blank=ShortTerm` on a Union chooses that variant.
The portable `LLMDocField` attribute value is `blank:` followed by the literal
in its portable Default encoding, such as `blank:$null`, `blank:0`, or
`blank:ShortTerm`; the other portable values are `auto` for the bare marker,
`true`, and `false`.

<!-- rule: doctrans.blank.valid-values class=hard enforced=yes tasks=ai-import title="blank needs a selected scalar, Enumeration or Union field and a value valid for it; Bytes and identities take only $null" -->
`blank` is valid on a selected scalar, Enumeration, or Union field, Nullable
or not, and its value must be valid for that field: `$null` requires
Nullable, a member or variant must exist (a Union variant must not be Hidden
or Fixed), and a number or text must fit the declared type and bounds. Bytes
and identity fields (User, Device, LanDevice, VirtualDevice, Endpoint) accept
only `blank=$null`. <!-- rule: doctrans.blank.not-with-allow-empty class=hard enforced=yes tasks=ai-import title="blank cannot combine with allowEmpty" --> `blank` implies that the field accepts `""`, so it cannot
be combined with `allowEmpty`; <!-- /rule --> <!-- rule: doctrans.blank.default-shorthand class=hard enforced=no tasks=ai-import --> `allowEmpty=true` remains the shorthand for a
blank that applies the field's Schema `Default`. A field with neither keeps the
rules above. Studio rejects an invalid `blank` at the marker's source location.

<!-- rule: doctrans.schema.exclude-blank-default class=hard enforced=no tasks=client-runtime -->
A field that accepts `""`, through `allowEmpty=true` or `blank`, adds it to its
output schema: exact `""` joins an `enum` for EnumSource and permitted choices; Enumeration
and variant-only Union already use printed-text strings. A Boolean is JSON Boolean or `""`, a String
drops its nonempty requirement, a Number uses its string form, and a Union with
a marked payload field accepts `""` in place of its `variant`/`value` wrapper.
The schema sent to the model never exposes the blank value or the Default.
The client applies `""` exactly as if the user had typed or chosen that value.
On a Union it chooses the blank variant (the Default variant under
`allowEmpty=true`) as a manual choice does: the current variant keeps its
payload, and a different variant initializes its payload by the normal
construction rules, with required payload fields left unanswered and deferred
to Submit as described for a [different variant](#app-schema-authoring).

<!-- rule: doctrans.blank.optional-text class=hard enforced=no tasks=ai-import -->
For ordinary optional text whose domain meaning is “blank when not supplied,”
use a non-Nullable String with an empty Schema Default and explicit
`allowEmpty=true`. In Rust, combine `#[libertas_default("")]` with
`#[libertas_llm_doc_field(allow_empty = true)]`; the bare field marker still
defaults to `false` for String. Permit length zero in its size bounds. Describe
the manual answer as “Leave it empty if absent” and tell the model to return
exact `""` for genuinely missing text. Reserve JSON `null` for truly Nullable
destinations. <!-- rule: doctrans.blank.unreadable-is-error class=judgment enforced=no tasks=ai-import --> Unreadable text is an error, not evidence that the text is absent.
Follow the [type-selection guidance](app-contract.md#user-facing-and-non-user-facing-types)
when manual input and an internal record use different representations.

<!-- rule: doctrans.import.printed-values class=hard enforced=no tasks=ai-import -->
AI Import reads the printed document. Return code letters as printed, dates as
printed, amounts as printed, and each address as one text block with its printed
line breaks. Do not expand codes into their meanings, normalize dates or amounts,
add an unprinted country, include a telephone number in an address, or repeat a
separately selected party name in its address. Keep identifiers as text, including
leading zeros. The client owns deterministic conversion to the declared stored
types; App instructions identify the printed boxes rather than teach storage
encodings.

<!-- rule: doctrans.convert.strict class=hard enforced=no tasks=client-runtime -->
The client checks the complete returned structure and converts each printed
scalar using the destination's declared semantics. Missing properties, extra
properties, incorrect containers and non-scalar substitutions reject the
proposal. A scalar that cannot be converted unambiguously stays unanswered in
the editor, with its original printed text and conversion explanation visible
beside the field. It never becomes null, zero, a Default or a guessed answer.
Other successfully converted fields remain available for review. The unanswered
state is local editor state and cannot enter an encoded request. Normal field,
constraint and cross-field validation governs the converted values and final
submission; rules that depend on an unanswered value wait for an answer.

<!-- rule: doctrans.conversion.failure-no-singleton class=hard enforced=no tasks=client-runtime -->
A failed printed conversion remains unanswered even when a constrained field has
only one available choice. Ordinary automatic singleton selection cannot replace
the failed reading; a successful manual answer or related linked answer clears
its local conversion feedback.

### Complete JSON projection and client conversion

<!-- rule: doctrans.conversion.ordinary-semantic-codecs class=hard enforced=no tasks=client-runtime -->
JSON Schema describes the JSON representation. The client converts that
representation into the destination's actual Libertas type with its ordinary
value codecs and validation rules; JSON primitive types do not replace the
App's semantic types.

<!-- rule: doctrans.schema.descriptions.authoring class=hard enforced=no tasks=ai-import -->
Supply locating labels and domain import instructions. The model reads printed values; do not require it to interpret a code or convert to a storage representation. The platform owns [schema generation and conversion](#complete-json-projection-and-client-conversion).

<!-- /rule -->

<!-- rule: doctrans.schema.descriptions class=judgment enforced=no tasks=client-runtime -->
Generated nodes carry their locating label and source description, except a
`$ref` whose definition carries them. Omit enum choice meanings and
storage-representation hints when
the client can convert the printed value. Keep dynamic candidate descriptions
when the returned choice depends on the current editor snapshot, declared answer
labels, and guidance necessary to distinguish a genuinely blank box. The omissions
shorten the output schema without dropping fields; the 64 KiB limit applies.

| Libertas value | Model JSON and client conversion |
| --- | --- |
| String | JSON string; preserve text, applying the declared blank rule. |
| Boolean | JSON Boolean; no string truthiness or numeric coercion. |
| <!-- rule: doctrans.conversion.number-exactness class=hard enforced=no tasks=client-runtime --> Number | Decimal numeric string when blank conversion or exact wide-integer representation requires it; otherwise JSON integer/number. Parse the complete value, preserve integer precision, and enforce native width, range, and step. A value with `PhysicalUnit: percent` returned as printed text may end in one percent sign after optional whitespace (`10 %`, `10%`); the client removes it and reads the number, which is already in percent units (50 means 50 percent). |
| <!-- rule: doctrans.conversion.printed-date class=hard enforced=no tasks=client-runtime --> DateOnly | Printed date string; validate the complete Gregorian date and convert to the ordinary `yyyyMMdd` typed value. Accept an explicit year-first date, an English month name, or a numeric date with only one valid month/day interpretation; an ambiguous date stays unanswered, without inferring an order or a century. When the form declares [`form_date_order`](#app-schema-authoring), a numeric date with a four-digit year is read in that order instead and must still be a valid Gregorian date, otherwise it stays unanswered; year-first and month-name dates are read as before, and a two-digit year stays unanswered. The model still returns the date as printed. |
| DateTime | RFC 3339 string with an explicit offset; convert to the ordinary epoch-second instant without guessing a timezone. |
| TimeOnly | `HH:mm:ss` string; convert to seconds from local midnight under the normal TimeOnly rules. |
| <!-- rule: doctrans.conversion.money-precision class=hard enforced=no tasks=client-runtime --> Money | Printed amount string in the declared currency; the client recognizes a complete decimal amount, comma groups of three digits, a matching currency marker, a leading sign or enclosing parentheses for a negative amount. Convert exactly to integer minor units using the declared scale. Ambiguous punctuation, a conflicting currency, excess fractional precision and overflow leave the value unanswered. |
| TimeInterval | Decimal seconds string; convert to the declared numeric duration type and bounds. |
| <!-- rule: doctrans.conversion.printed-code class=hard enforced=no tasks=client-runtime --> Enumeration | A printed code is the member's or variant's declared [form code](#declared-form-codes) when present, otherwise the member name; nothing is inferred from labels. The code maps to that member's Fixed value or ordinal; the returned code, after trimming surrounding whitespace, must equal exactly one member's code. The JSON schema accepts printed text, including an unrecognized code, rather than forcing a choice from an `enum`; the permitted-code snapshot is listed without its meanings. Code meanings are not included in the output schema. No client guesses a code by interpreting its description; an unknown or ambiguous code stays unanswered. |
| <!-- rule: doctrans.conversion.enumeration-values class=hard enforced=no tasks=client-runtime --> Dynamic EnumSource | Canonical decimal index string constrained by `enum` to the currently permitted indices; for a List whose member declares a [key](schema.md#enumsource-keys), the member's key as a string (decimal for an integer key) constrained to the current keys instead. Include the corresponding current choice values in schema descriptions. Resolve the returned index or key through that same choice snapshot and normal constraint rules, and store the index or the typed key. A [List host](schema.md#enumsource-list-hosts) is a JSON array of these strings; a repeated element rejects the proposal, and `Size` applies. |
| <!-- rule: doctrans.conversion.bitflags-mask class=hard enforced=no tasks=client-runtime --> Bitflags | Without a [declared flag code](#declared-form-codes): decimal mask string with the declared catalog described in the schema; convert exactly and reject undeclared bits. |
| <!-- rule: doctrans.conversion.bitflags-codes class=hard enforced=no tasks=client-runtime --> Bitflags with declared flag codes | When any flag of the catalog declares a [form code](#declared-form-codes): a JSON array of the printed codes of the flags marked on the form, listed as a permitted-code snapshot without meanings; an empty array marks none. Each element, after trimming surrounding whitespace, must equal exactly one flag's declared code; the mask is the bitwise OR of the matched flags' values. A non-array, a non-string element, an unknown code or a repeated code leaves the whole field unanswered, never a partial mask. Valid combinations remain the App's ValidationRules. |
| User, Device, LanDevice, VirtualDevice, Endpoint | Decimal identity string; convert without precision loss and apply the ordinary identity/selector constraints. Include the current permitted candidates and their meanings when the editor provides a choice set. |
| Bytes | Base64 string; decode to Bytes and enforce ordinary byte constraints, including LMB1 or formatted-value rules when annotated. |
| <!-- rule: doctrans.conversion.table-preserve-omitted class=hard enforced=no tasks=client-runtime --> Table | JSON object with every selected child required and no extra properties; recursively convert children while preserving omitted values in an existing Table or unchanged Union variant. |
| List | JSON array; recursively convert every element and enforce its declared size and element constraints. |
| <!-- rule: doctrans.conversion.union-marked-payload class=hard enforced=no tasks=client-runtime --> Union with a marked payload field | An `anyOf` of variant objects, each with required `variant` and `value` properties and no extras. `variant` is that branch's canonical name; `value` holds that variant's marked payload fields (an empty object when it has none). Convert through the ordinary Union discriminator and payload codec. |
| Union with no marked payload field (variant-only) | As printed Enumeration codes resolved through its available variants. Keep the current variant and payload when it is unchanged; otherwise select it as the editor does, with normal payload initialization. |
| Nil | JSON null. |
| <!-- rule: doctrans.conversion.nullable-domain-choice class=hard enforced=no tasks=ai-import --> Nullable | Add null to that value's JSON schema; a returned null clears the typed value. Never add Nullable merely for recognition. |

<!-- rule: doctrans.conversion.physical-unit-semantics class=hard enforced=no tasks=client-runtime -->
Physical units retain their ordinary semantics; they do not change the
underlying value type. <!-- rule: doctrans.schema.no-template class=hard enforced=no tasks=ai-import --> Display Template presentation, including a `format`
pattern, never reaches the model. <!-- /rule --> <!-- rule: doctrans.schema.hidden-fixed-excluded class=hard enforced=no tasks=client-runtime --> Hidden, Fixed and Secret values never enter the
model's output schema or its returned result; normal initialization and
validation continue to govern them. <!-- rule: doctrans.blank.semantic-number-defaults class=hard enforced=no tasks=ai-import -->
Number annotations such as DateOnly, Money, TimeInterval, EnumSource, and
Bitflags keep the Number default of `allowEmpty=true`; their typed Default is
applied locally for exact `""`. Other semantic scalar types default to
`allowEmpty=false` and may explicitly enable it with a meaningful Default.

<!-- rule: doctrans.enum-source.snapshot class=hard enforced=no tasks=client-runtime -->
Dynamic EnumSource is resolved from the bound Request when the request is
prepared. Its current choices, allowed-choice constraints, and relevant
reference values belong to the pending transaction snapshot. If they change
before application, discard the result as stale. A choice cannot
be silently reinterpreted against a newer list.

#### Value-representation hints

<!-- rule: doctrans.schema.printed-hints-omitted class=hard enforced=no tasks=client-runtime -->
The table below records the value-representation hints for AI Import and for
[AI tool-call descriptions](presentation.md#ai-tool-call-descriptions). AI Import
omits the Money, DateOnly and ordinary numeric-string hints of its column: those
values are printed text under the [client conversion rule](#complete-json-projection-and-client-conversion).
Tool calls retain their ordinary typed argument representation. <!-- rule: doctrans.schema.hints-derived class=hard enforced=no tasks=client-runtime --> Each hint follows from the
node's effective schema attributes, including inherited ones, and its built-in
type; App authors never write it. Every applicable hint is one line, in table
order, after the node's text and choice list. The words are fixed English; an
angle-bracketed part comes from the schema or its document. A hint states the
JSON representation only: it never changes storage or encoding, supplies a
value, or lists choices beyond those `enum` already offers.

| Applies to | AI Import hint | Tool-call hint |
| --- | --- | --- |
| Money Number | `Money: <currency>, scale <scale>; decimal amount in major currency units as a string, with '.' as the decimal separator and at most <scale> decimal places; no currency symbol or grouping separators.` | `Money: <currency>, scale <scale>; integer minor units (<one major unit in minor units> = <currency> 1). Use the exact integer, without a currency symbol or decimal amount.` |
| DateOnly Number | `DateOnly: calendar date string in YYYY-MM-DD format, e.g. "2026-09-20"; no timezone conversion.` | Same |
| TimeInterval Number | `TimeInterval: duration in seconds as a decimal string; e.g. 5 minutes = "300".` | `TimeInterval: numeric duration in seconds; e.g. 5 minutes = 300.` |
| Number in its string form with no hint above and no Bitflags or EnumSource | `Number: decimal number as a string, without grouping separators or a currency sign.` | None: the JSON type is `integer` or `number`. |
| PhysicalUnit Number | `PhysicalUnit: <unit>. The number is expressed in this unit; no automatic unit conversion.`, followed by ` 50 means 50 percent.` for `percent` | Same |
| Bitflags Number without declared flag codes | `Bitflags: integer mask as a decimal string; combine declared flags using bitwise OR. Undeclared bits are invalid.` | `Bitflags: integer mask; combine declared flags using bitwise OR. Undeclared bits are invalid.` |
| Bitflags Number with declared flag codes | `Bitflags: an array of the printed codes of every flag marked on the form, each listed once; an empty array when none is marked.` | As without declared flag codes |
| DateTime | `DateTime: RFC 3339 date-time string with an explicit UTC offset or Z, e.g. "2026-09-20T14:30:00Z"; identifies an instant.` | Same |
| TimeOnly | `TimeOnly: time string in HH:MM:SS format, e.g. "14:30:00"; no date component.` | Same |
| EnumSource Number or String | Keyed List: `EnumSource: the chosen item's key from the choices; never its index or label.` Otherwise: `EnumSource: the chosen item's zero-based index from the choices; never its label.` | Keyed List: `EnumSource: the Key field (<key>) value of the chosen item in the List at <path> in the current data tree; use that key, not the item's index or label.` Unkeyed List: `EnumSource: zero-based index into the List at <path> in the current data tree; use the item's index, not its label or value.` List not resolved: `EnumSource: the chosen item of the List at <path> in the current data tree; use the item's Key field value if its member Table declares one, otherwise its zero-based index; not its label.` |
| EnumSource List host | Keyed List: `EnumSource list: an array of distinct chosen items' keys from the choices; never labels; an empty array chooses none.` Otherwise: `EnumSource list: an array of distinct chosen items' zero-based indexes from the choices; never labels; an empty array chooses none.` | `EnumSource list: an array of distinct chosen items of the List at <path> in the current data tree; each element is <element>; never labels; an empty array chooses none.`, where `<element>` is `the item's Key field (<key>) value`, `the item's zero-based index`, or, when the List is not resolved, `the item's Key field value if its member Table declares one, otherwise its zero-based index` |
| Unordered List | `Unordered: List order has no semantic meaning; duplicates are not prohibited by this attribute.` | Same |
| `Multiline` String | `Multiline: text may span several lines; keep the document's line breaks as \n.` | Same |
| String without `Multiline`, EnumSource or an identity type | `Single line: no line breaks; join the document's lines with one space.` | Same |
| `Secret` String | None: AI Import excludes it as [protected content](#app-schema-authoring). | `Secret: confidential; a result carries {"withheld":"Secret"} in place of the value. Supply a value only when the person gives it.` |
| Bytes | `Bytes: base64-encoded bytes.` | None: the schema declares `contentEncoding`. |
| User, Device, LanDevice, VirtualDevice, Endpoint | `Identity: exact decimal ID as a string.` | None: the schema lists names. |

A Bitflags mask hint in either form is followed by one line per declared flag,
`<member> = <value>`, with ` (<Display Name>): <Description>` appended when
the catalog document lists that member. In the tool-call hint, `<one major unit
in minor units>` is 1 followed by `<scale>` zeros, such as `100` for `USD/2`.

### Declared form codes

<!-- rule: doctrans.form-code.declaration class=hard enforced=yes tasks=ai-import title="FormCode is a nonempty code on an Enumeration member, Union variant or nonzero Bitflags flag, unique among its siblings" -->
An Enumeration member, a Union variant or a nonzero Bitflags flag may declare
`FormCode`, the exact code the form prints for that choice or box:
`#[libertas_form_code("F")]` in Rust, `@LibertasFormCode("F")` in TypeScript.
On a Rust flag `const`, inside `bitflags!` or a manual `impl`, import the
attribute (`use libertas_macros::libertas_form_code;`) and write
`#[libertas_form_code("7")]` above the constant. The zero flag, the catalog's
"None of these" label, never declares one. AI Import lists that code in the
permitted-code snapshot sent to the model and matches the model's reply to it
exactly after trimming surrounding whitespace, wherever the choice appears in
a result, including inside Nullable values and Union variants. The code is
nonempty after trimming and unique among the sibling members, variants or
flags. It changes neither the choice's label, its display nor its stored value.
A Bitflags field whose catalog declares flag codes is read as an array of
codes under the [Bitflags conversion rule](#complete-json-projection-and-client-conversion);
a flag without a code cannot be matched there. Without any declared flag code,
Bitflags keep the mask representation. <!-- /rule -->

### Array import

A file often holds several forms of one type: separate forms for several
accounts, or a consolidated statement. <!-- rule: doctrans.array.inside-list class=hard enforced=no tasks=ai-import --> When the annotated struct occurrence
lies anywhere inside an editable List item, directly or through nested Tables
and Union payloads, its **AI Import** action imports an array. The struct type
of the occurrence the action is on decides what is imported: only forms of that
type.

- <!-- rule: doctrans.array.strict-forms-root class=hard enforced=no tasks=client-runtime --> The output schema is a JSON object with one required property, `forms`: an
  array whose every element is that struct's selected result object, as for a
  single import, with at least one element. The root stays an object because
  structured-output providers (OpenAI) require one.
  The client appends to the instruction that the model returns one element per
  distinct form in the files, that the files may contain duplicate copies of a
  form which count once, and ignores forms of other types and pages that are
  not forms. <!-- rule: doctrans.array.first-replaces class=hard enforced=no tasks=client-runtime -->
- The first element replaces the occurrence the action is on, exactly as a
  single import does. <!-- rule: doctrans.array.append-new-items class=hard enforced=no tasks=client-runtime -->
- Each further element is **appended** as a new item of the nearest enclosing
  List. The new item is constructed by normal initialization; every Union on
  the path from that item down to the struct selects the same variant as in the
  current item, as a manual choice does, and the element is then applied to the
  new occurrence as a single import. Other fields of the new item (for example
  its owner) keep their normal initialization. Existing items are never
  changed or removed. <!-- rule: doctrans.array.duplicate-instruction class=judgment enforced=no tasks=ai-import -->
- The App's `instruction` names what a duplicate looks like for its form (for
  example a tax form's Copy B, Copy C and Copy 2 printed on one page); the
  client's appended text stays generic.
- Duplicates are dropped: a file may hold several copies of one form, though
  not always. After conversion the client
  drops an element to be appended whose [declared duplicate identity](#declared-duplicate-identities)
  equals another element's or an existing item's at the same struct position.
  Without a declaration, the comparison uses exact equality of all imported values. The review states how many
  duplicates were skipped. <!-- rule: doctrans.array.all-or-nothing class=hard enforced=no tasks=client-runtime -->
- Each element is converted and validated like a single result, and the whole
  proposal is one editor change: if any element fails, or the appended items
  would exceed the List's declared size, nothing is applied. <!-- rule: doctrans.array.single-outside-list class=hard enforced=no tasks=ai-import -->
- A struct occurrence outside any List, or inside a List whose membership is
  ReadOnly, keeps the single import.

### Declared duplicate identities

<!-- rule: doctrans.duplicate-identity.paths class=hard enforced=yes tasks=ai-import title="A form's duplicate_identity names distinct imported scalar or whole List/Union values through self-relative Table paths" -->
A form may declare `duplicate_identity` on `LLMDocTrans`: a nonempty,
comma-separated list of distinct [Schema Paths](schema.md#context-and-syntax)
rooted at `@`, the form itself. Each path passes through Table fields and ends
at a selected imported scalar, List or Union. A List or Union endpoint is one
whole typed value: the path cannot traverse into it, and matching never pairs
its individual items. A path cannot leave the form, end at a Table or Nil, or
name Hidden, Fixed or unselected content. A classifier cannot declare it.
Choose printed values that together identify one paper. For example, a QSBS
statement can declare `duplicate_identity="@.lots"` for its complete imported
List of lots; an estate credit entry can declare
`duplicate_identity="@.code,@.amount"`, where code may be a Union and amount a
scalar. <!-- /rule -->

<!-- rule: doctrans.duplicate-identity.equality class=hard enforced=no tasks=ai-import,client-runtime title="Declared typed identity equality identifies one paper despite differences in other imported fields; undeclared forms keep exact equality" -->
Within the same form and struct position, converted identity values compare
as an ordered typed tuple, without text normalization. A whole List compares
its complete ordered value, including length and every typed item; a whole
Union compares its selected variant and complete typed payload. Changing an
item, its order, a variant or its payload makes that component unequal. Null is a value; a
null containing Table supplies null for its selected descendant. An unanswered
local identity value does not equal a supplied null answer. An appended
form whose tuple equals an existing item's or an earlier imported form's is
one duplicate, even when other imported values differ. Keep the existing or
first imported paper unchanged and include the skipped copy in the review's
duplicate count. Without a declaration, retain exact imported-value equality.
The first result of an array import still replaces its current occurrence;
all further elements and categorized imports use this duplicate rule. <!-- /rule -->

Conversion and validation still follow the array import's
[all-or-nothing rules](#array-import), including a copy that is discarded.

The portable metadata remains the `LLMDocTrans` attribute, with optional
`duplicate_identity` in its JSON config; [the schema shape](schema.md#document-duplicate-identity)
owns that representation. Rust and TypeScript use the same named argument.

### Categorized import

<!-- rule: doctrans.categorized.independent-transforms class=hard enforced=no tasks=client-runtime -->
A person often holds a pile of different documents at once, such as W-2s,
several 1099s and a K-1. One transformation over every form at once is
impractical: one instruction cannot carry every form's instructions well, and
one result schema with every form's fields exceeds provider schema limits and
degrades accuracy. A **categorized import** first sorts the files into
documents of known kinds, then imports each document with its own kind's
declaration.

<!-- rule: doctrans.classifier.definition class=hard enforced=no tasks=ai-import -->
**Classifier rule.** An `LLMDocTrans` Table is a **classifier** if and only if
all its variants are `LLMDocTrans` structs. Its variants are its import
positions: each selected field, each selected List's item, and, in an item
Table, its one marked form or each variant of its one marked Union (the item's
other fields, such as an owner, keep their normal initialization). A variant may itself be a classifier, so
classifiers nest, typically one root classifier with one classifier per
**category**. Every other `LLMDocTrans` Table is a **form**, with the meaning
described above; an `LLMDocTrans` a form encloses keeps only its own action. <!-- rule: doctrans.classifier.no-marked-child class=hard enforced=no tasks=ai-import --> Selection is the
ordinary rule above (the marked direct children, or all unprotected children
when none is marked); a form's requirement to mark at least one direct child
does not apply to a classifier.

```rust
/// Documents you may have
#[libertas_llm_doc_trans(instruction = "Sort each document into its kind. ...")]
pub struct DocumentCenter {
    /// Wages
    pub wages: WagesCategory,
    /// Interest and dividends
    pub interest_dividends: InterestDividendsCategory,
}

/// Wages
#[libertas_llm_doc_trans(instruction = "Wage statements from an employer. ...")]
pub struct WagesCategory {
    pub items: Vec<WageItem>,
}

pub struct WageItem {
    /// Owner
    pub owner: Owner,               // normal initialization
    /// Document
    #[libertas_llm_doc_field]
    pub document: WageDocument,
}

pub enum WageDocument {
    /// Form W-2
    /// Wage and Tax Statement from an employer.
    W2(W2),                         // W2 is an LLMDocTrans form
    /// Form 8919
    /// Uncollected Social Security and Medicare tax on wages.
    Form8919(Form8919),             // Form8919 is an LLMDocTrans form
}
```

An editor usually draws a classifier with many categories in the
[categorized layout](presentation.md#categorized-layout): only the categories
that hold a document, plus one control that adds any kind.

This schematic Rust omits derives and the form structs. The root classifier is a
[Table, not a Union](#app-schema-authoring), because a Union holds one variant at a time while the
editor holds every category at once.

<!-- rule: doctrans.classifier.instruction class=judgment enforced=no tasks=ai-import -->
**Arguments.** On a classifier, `instruction` (required) says what counts as one
document, what to leave out, and how to recognize duplicate copies; the client
appends the generic classification text below.
`doc_type` filters the selectable files as for a form. <!-- rule: doctrans.classifier.file-selection class=hard enforced=no tasks=ai-import --> A classifier always
accepts several files; <!-- /rule --> <!-- rule: doctrans.classifier.multiple-files class=hard enforced=yes tasks=ai-import title="A classifier takes no validation path or accept_multiple_inputs = true" --> a `validation` path or `accept_multiple_inputs = true` is
invalid on it (the emitted defaults are accepted). <!-- rule: doctrans.classifier.form-arguments class=hard enforced=no tasks=ai-import --> Each form keeps its own `validation`, used when its kind is imported; its
`doc_type` and `accept_multiple_inputs` govern only its own action.

**Assembled instruction.** A classification request sends one instruction: the
classifier's own `instruction`, then, in declaration order, each enclosed
classifier's native name and `instruction`, nested the same way down to the
categories. Native names keep the assembled bytes independent of locale; the
localized display names name the kinds (below). A root classifier's rules for a category therefore reach the model
when the root sorts a pile. Leaf forms' instructions are not included; they
belong to the import step. <!-- rule: doctrans.classifier.assembled-instruction class=hard enforced=no tasks=client-runtime --> The assembled instruction is decided by the schema
alone, and the platform generates it; an App author does not budget it.

**Kinds.** A classifier's kinds are **all the forms under it**: every
non-classifier `LLMDocTrans` reached through its variants and through every
nested classifier. Traversal stops at a form, so declarations a form encloses are
not kinds. <!-- rule: doctrans.classifier.kind-constraints class=hard enforced=no tasks=ai-import --> A kind is offered only while the Union's ordinary choice
constraints (Hidden, Fixed, `ConstrainedBy`) permit it for a new item at its
position. A kind inside a List is a **list kind**, and the nearest List
enclosing it is its **category List**. A form that is a classifier's variant
itself, outside any List, is a **single-document kind**, such as a passport or a
birth certificate: the Request holds one occurrence of it. <!-- rule: doctrans.classifier.kind-names class=judgment enforced=no tasks=ai-import --> A kind is named for the model by its
variant's (or, for a one-kind List, its struct's) localized display name and
description, with the display names of the classifiers between the importing
classifier and the kind. Write them to say exactly what the document is, such as
"Form 1099-S for the sale of your main home". The root classifier offers every
leaf kind; a category classifier offers its own.

<!-- rule: doctrans.classifier.item-shapes class=hard enforced=yes tasks=ai-import title="A List under a classifier holds a Union of forms, a Table with one marked Union or form, or a form" -->
**Item shapes.** A List under a classifier holds one of:

- a Union, whose variants with a form payload are kinds;
- a Table with exactly one direct child marked `LLMDocField` that is a Union
  (read the same way) or a form (a one-kind List); its other fields, such as an
  owner, keep their normal initialization; or
- a form struct, making a one-kind List.

**Studio validation.** The shared validator decides the classifier rule from
the emitted schema. A classifier must be a [named Table](#app-schema-authoring) <!-- rule: doctrans.classifier.placement class=hard enforced=yes tasks=ai-import title="A classifier is used in an editable Request or SubscriptionRequest and has at least one kind" --> used in an editable
Request or SubscriptionRequest, with at least one kind, <!-- /rule --> without a [`validation`
path or `accept_multiple_inputs = true`](#categorized-import), and with every enclosed List of an [allowed item
shape](#categorized-import). <!-- rule: doctrans.classifier.readonly-list-and-single-kinds class=hard enforced=yes tasks=ai-import title="Studio refuses a ReadOnly classifier List and warns when a single-document kind encloses import declarations" --> Studio refuses a classifier List whose membership is ReadOnly, and warns
when a single-document kind itself encloses import declarations (it is imported
as one form; a record or Nil variant in a category Union causes this). Each failure is reported at the
declaration's source location.

**Action.** Every classifier occurrence offers an **AI Import** action, except
that [categorized layout](presentation.md#categorized-layout) gives its outermost
classifier the sole import action and manual document chooser for that subtree.
The action is placed as the
[presentation contract](presentation.md#actions-on-a-promoted-or-suppressed-row)
describes. Its availability is AI Import's: absent when AI is unavailable under
the [Control AI availability rule](protocols.md#ai-navigation-and-ai-search),
disabled while any calculation is pending or deferred, and unavailable in
internal calculation requests and inside Hidden, Fixed, CacheIdentity, or
ReadOnly subtrees. Outside categorized layout, each category List keeps its
ordinary **Add**: a one-kind List appends its struct, and a multi-kind List opens
its variant chooser.

**Workflow.**

1. **Select.** The person takes pictures or opens files, several at once, and
   can add more before starting. Images are
   prepared as described under [client workflow](#client-workflow); PDF bytes
   are unchanged.
2. **Classify.** The client sends one transformation per batch of at most
   eight files and 8 MiB, the request limits above. Its output schema is:

   ```json
   {
     "type": "object",
     "properties": {
       "documents": { "type": "array", "items": {
         "type": "object",
         "properties": {
           "kind": { "type": "string", "enum": ["…offered kinds…"] },
           "parts": { "type": "array", "items": {
             "type": "object",
             "properties": {
               "file": { "type": "integer" },
               "pages": { "type": "array", "items": { "type": "integer" } }
             },
             "required": ["file", "pages"], "additionalProperties": false } }
         },
         "required": ["kind", "parts"], "additionalProperties": false } },
       "not_imported": { "type": "array", "items": {
         "type": "object",
         "properties": {
           "file": { "type": "integer" },
           "pages": { "type": "array", "items": { "type": "integer" } }
         },
         "required": ["file", "pages"], "additionalProperties": false } }
     },
     "required": ["documents", "not_imported"], "additionalProperties": false
   }
   ```

<!-- rule: doctrans.categorized.model-page-ownership class=hard enforced=no tasks=client-runtime -->
   `kind` is an enumeration with one unique name per offered kind; its
   `description` lists every kind with the name text described under
   **Kinds**, as an Enumeration's choices are described. `file` is the
   one-based position of a file in that request, and `pages` its one-based
   pages; an image has one page. The appended generic text says that the result
   has one element per distinct document, that duplicate copies of a document
   count once, and that pages belonging to no offered kind are listed in
   `not_imported` instead. The import itself does not open or count the files: it
   forwards them as selected and relies on the model's page numbers; a document
   whose pages do not exist fails its import and becomes Enter by hand, and so
   does a document whose part names a file the request does not contain or no
   pages; an unusable `not_imported` entry is listed as given. One bad entry never
   ends the import. Only the
   transport of a [client provider](llm-client-requests.md#provider-mapping) may
   convert a file for a provider that cannot take it, such as a PDF sent as one
   PNG per page; that conversion never decides the import's pages.
3. <!-- rule: doctrans.files.never-rewrite-pdf class=hard enforced=no tasks=client-runtime --> **Import.** For each classified document, the client sends that kind's own
   transformation: the kind's instruction, its [array import](#array-import)
   output schema, and its `validation` facts resolved in a new item of its
   category List constructed by normal initialization. The request carries the
   files the document's pages lie in, and the instruction names those pages,
   for example "Read only pages 3 to 4 of file 2". Clients never split or
   rewrite a PDF. Classification and import requests run concurrently, with at
   most four of an import's transformations pending at once, the Hub's limit.
4. **Progress.** The blocking busy overlay with **Cancel** of the
   [client workflow](#client-workflow) shows determinate text, such as
   "Sorting 12 files…" and then "Reading 5 of 9 documents…".
5. **Review.** Before anything changes the editor, one sheet lists one row per
   document, grouped by category:
   - **Ready**: the kind and a short summary of its imported values, such as
     "W-2 · Acme Inc." (see the summary rule below);
   - **Enter by hand**: classified, but its import failed (unreadable pages, a
     provider error or timeout, a result that fails conversion or validation); a list kind is added with its kind selected and its payload in
     normal initialization; a single-document kind's occurrence stays unchanged;
   - **Replaces your current entry**: a single-document kind whose occurrence
     is no longer in normal initialization; its checkbox starts unchecked;
   - **Not imported**: each file and its pages that the classification lists in
     `not_imported`, and the count of documents dropped as duplicates.

<!-- rule: doctrans.categorized.review-summary class=hard enforced=no tasks=client-runtime,display-template title="A Ready row's summary is the category List's compiled `summary` paths read from the imported item; without them, the first three answered scalars of the import, never a Boolean, an empty value or a field named year" -->
**Ready summary.** When the compiled [Display Template](presentation.md#summary-array-items)
gives the kind's category List a `summary` layout, the row summary is that
rule's `paths` read from each imported item, in path order, joined by " · "
(an empty value contributes nothing). Otherwise the summary is the first three
answered scalars of the import in declaration order, skipping Booleans, null
and empty values, and a field named `year`, `tax_year` or `calendar_year`,
which is the paper's period and does not identify it. A document that imports
several forms shows each form's summary, separated by "; ". The summary is
at most 160 characters.

<!-- rule: doctrans.categorized.apply.authoring class=hard enforced=no tasks=ai-import -->
Rely on the normal [import apply and review flow](#categorized-import); importing changes the editor without submitting the App request.

<!-- /rule -->

   Each Ready and Enter-by-hand row has a checkbox, checked by default, so a
   misclassified document can be left out. **Add selected** adds the
   checked documents; **Cancel** discards them all. <!-- rule: doctrans.categorized.apply class=hard enforced=no tasks=client-runtime -->
6. **Apply.** Applying is one editor change. A checked single-document kind is
   applied to its occurrence as a single import; a second document of the same
   single-document kind is listed as **Not imported** (only one is allowed).
   **Selection order** decides: documents are ordered by the position of their
   first file and first page in the person's selection, never by when a reply
   arrives, so "second" is the later one in that order, and the review's rows and
   the appended items follow the same order.
   Each checked list-kind document is **always appended** as a new item of its
   category List (the import is started from a classifier, outside any document,
   so there is no current item to replace), its variant selected as a manual choice
   does, and its result applied under the [array import](#array-import) rules:
   the first form fills the new item, further forms of that kind are appended
   with the same variant, and duplicates of another imported form or of an
   existing item at the same position are dropped, using the form's
   [declared identity](#declared-duplicate-identities) when present and exact
   imported-value equality otherwise. Existing items are never
   changed; a filled single-document occurrence changes only when its
   **Replaces your current entry** row is checked. If any List's declared size would be exceeded, nothing is applied
   and the sheet names that List.
7. **After.** The Request is not submitted; the person reviews and saves as
   usual.

The categorized import is one blocking action under the
[pending and stale rules](#client-workflow), through its review: Cancel,
navigation, or a disconnect discards every pending reply and leaves the editor
unchanged. An error or timeout of any classification request ends the import,
is reported, and leaves the editor unchanged; the person can retry. An import
step's failure makes only that document Enter by hand.

Without AI, every step above is absent; **Add** and each item's editor work as
usual.

## Client workflow

The model request itself, including a client's own provider account, follows
[LLM requests](llm-client-requests.md).

<!-- rule: doctrans.image.resize-with-quality-floor class=hard enforced=no tasks=client-runtime -->
The **AI Import** action (Chinese: **AI导入**) first offers **Take a picture**
using ordinary camera capture when the effective MIME filter accepts camera JPEG
and a camera is available,
and **Open a file**. Each action selects one file by default, or up to eight in
user-selected order when enabled. Captured photos use the same preparation and
validation as selected image files. Clients decode and orient selected images,
flatten transparency on white, and JPEG-encode them at no more than
**3,000,000 content bytes each**. Image preparation must keep reducing an
oversized image's dimensions until its JPEG fits. JPEG quality stays at or above
70, and the source is never upscaled. An image that cannot be decoded or encoded reports an input
error. PDF bytes remain unchanged. Clients check the combined LMB1 limit of
8 MiB before sending.

The client uses a session-unique unsigned 64-bit transaction ID. A pending
action belongs to its originating Control session, editor, struct occurrence,
selected files, validation value, and result-field state. A cancel, timeout,
navigation, or disconnect, or a change to a relevant value while it is pending
(for example an arriving calculation result), makes its reply stale. Only the matching
pending reply can change the editor.

<!-- rule: doctrans.import.waits-for-calculations class=hard enforced=no tasks=client-runtime -->
Like Submit, AI Import is disabled while any calculation is pending or deferred and becomes available when the calculations settle, so no calculation result can arrive during a transformation and make its reply stale. <!-- /rule --> AI Import is blocking. While its transformation is pending, the client shows a
busy overlay over the whole editor with a **Cancel** action; the editor and its
actions accept no input, so no edit can make the pending reply stale and no
second transformation or Submit can start. The busy state ends, and the editor
accepts input again, when the matching reply arrives (a result is applied as
above; an error is reported), when the user cancels, when the timeout elapses,
or when the Control session disconnects. Cancel, timeout, and disconnect
discard the pending action, so a late reply is ignored; an error, a timeout, or
a disconnect is reported to the user, who can retry. Navigating away from the
editor also discards it. On success, the client checks the JSON
shape, strictly converts each value, validates the complete candidate, and
applies all selected fields in one normal editor change, including values
supplied by a Schema Default. Ordinary drafts and calculations remain active. <!-- rule: doctrans.import.no-submit class=hard enforced=no tasks=client-runtime -->
AI Import does not submit the Request. An invalid structure or failed transaction validation leaves the editor
unchanged. A scalar conversion failure follows the [unanswered-field rule](#complete-json-projection-and-client-conversion). The user can review and correct the filled fields before
submitting the Request.
