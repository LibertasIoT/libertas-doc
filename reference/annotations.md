# Libertas source annotation reference

This is the canonical detailed reference for public source annotations accepted
by the current Libertas schema parsers. <!-- rule: annot.canonical-spellings-only class=hard enforced=no tasks=data-model title="Use only the canonical annotation spellings; compiler acceptance does not establish supported syntax" --> Use only the canonical spellings shown
here. Compiler acceptance alone does not establish supported schema syntax.

The codebot kit carries an
[exhaustive compact projection](../codebot/schema_annotations.md)
of this reference. This page owns the detail; the projection exists to minimize
routine App-generation context.

The language parser owns source spelling, native name and constant resolution,
and extraction. Studio's shared validator owns placement, effective host type,
value grammar, conflicts, protocol reachability, and other portable semantics. <!-- rule: annot.placement-is-authoring class=judgment enforced=no tasks=data-model -->
The placement column below states the public authoring placement; it is not
permission to move an annotation merely because a parser can preserve it for a
later Studio error.

Read the [App contract](app-contract.md) first. Use the
[TypeScript](languages/typescript.md) or [Rust](languages/rust.md) guide for project structure, and
follow each row's topic link for the complete behavior and value vocabulary. <!-- rule: annot.documentation-invariants class=hard enforced=no tasks=data-model -->
Every annotated declaration and nested schema-visible node must also follow the
[schema documentation invariants](documentation.md), including the
user-friendly label/description, member, variant, and List-item rules.

AI Import and the Hub supply [value-representation hints](llm-document-transformation.md#value-representation-hints)
from semantic attributes, and send declared [answer labels](documentation.md#answer-labels-in-ai-descriptions);
the Hub also [renders FormattedText results](presentation.md#ai-tool-call-output). <!-- rule: annot.comments-domain-meaning class=judgment enforced=no tasks=data-model -->
Source comments supply domain meaning without duplicating those generated notes.
Rows marked "compiled into the Display Template" are presentation attributes:
Studio validates them, compiles them into the package's
[Display Template](presentation.md#display-template) as its lowest-specificity
rules, and omits them from the final schema. A `display.json` rule can change
their effect per device without editing source. <!-- rule: annot.catalog-scope class=judgment enforced=no tasks=data-model -->
This catalog exhausts schema-parser annotations, not unrelated SDK runtime
functions or serialization derives such as `LibertasAvroEncode`,
`LibertasAvroDecode`, `VariantIndex`, or `variant_index!`; those remain covered
by the public SDK and the language guides.

## List item annotations

<!-- rule: annot.list-item-divider class=hard enforced=yes tasks=data-model title="The List divider assigns following annotations to the item; Studio warns for annotations that also support the List" -->
On a List declaration, a documentation divider line `----` separates the List
from its item. Documentation, custom properties, and **every annotation** before
the divider apply to the List; those after it apply to the item. The rule is
positional and introduces no new annotation names or encoded value shapes.
Studio validates each annotation against the resulting host: a List-only
annotation below the divider is invalid on a scalar item, and an item-only
annotation above it is invalid on the List.
Studio warns at an annotation below the divider when it also supports the List, so choosing the item instead of the List is explicit.

TypeScript uses JSDoc tags below the divider:

```ts
/** Activities on the statement
 * @LibertasLLMDocField
 * ----
 * @LibertasNumber(type=i64)
 * @LibertasLLMDocField(allowEmpty=false)
 */
activities: number[];
```

Rust uses real attributes after the divider's documentation line, retaining
native source order:

```rust
#[libertas_llm_doc_field]
/// Activities on the statement
/// ----
#[libertas_llm_doc_field(allow_empty = false)]
pub activities: Vec<i64>,
```

The List and item have independent declarations. For the recursive AI Import
projection, an item marked `allowEmpty=false` treats a blank numeric item as
a read error without a Default; the container's marker does not
change the item. Follow the [recursive result and blank-value contract](llm-document-transformation.md#complete-json-projection-and-client-conversion).

## Package, discovery, and parser helpers

| TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
| --- | --- | --- | --- | --- |
| <!-- rule: annot.default-locale.placement class=hard enforced=yes tasks=data-model title="DefaultLocale goes once in the entry file package documentation; omission means en" --> `/// @LibertasDefaultLocale("en-US")` | `//! #[libertas_default_locale("en-US")]` | Leading package/crate documentation in the selected entry file | Emits package `DefaultLocale`. Omission means `en`; all extracted source documentation supplies this locale's default document. | [Documentation and localization](app-contract.md#documentation-and-localization) |
| `/// @LibertasTypesOnly` | `//! #[libertas_types_only]` | Leading package/crate documentation | Emits `TypesOnly=true`. The package has no App functions and publishes every non-ignored exact-name public root type. | [Project and discovery rules](app-contract.md#project-and-discovery-rules) |
|  `/// @LibertasStringResources(NAME, …)` | `//! #[libertas_string_resources(NAME, …)]` | Leading package/crate documentation | Resolves one or more package-owned LMF1 catalog constants and emits the sorted keys of their union as `StringResources` ([catalog list](message-format.md#uses-and-storage)). Identifiers are positional, unquoted and comma-separated. | [String resources and formatted output](presentation.md#string-resources-and-formatted-output) |
|  `@LibertasIgnored` | Exact outer-rustdoc line `/// #[libertas_ignored]` | Public function or public named type | Omits the declaration as a schema root. A published declaration may not reach an ignored type. It takes no arguments. | [Project and discovery rules](app-contract.md#project-and-discovery-rules) |
| <!-- rule: annot.export.helper-only class=hard enforced=no tasks=data-model --> `@LibertasExport` | `#[libertas_export]` on a public function; `#[derive(LibertasExport)]` on a schema struct/enum | Parser/compiler helper | Enables the language's public App wrapper or helper attributes. It emits no portable `Export` attribute and does not replace the native public/export requirement. | [TypeScript guide](languages/typescript.md#supported-jsdoc-decorators), [Rust guide](languages/rust.md#schema-attributes-and-field-presentation) |
| <!-- rule: annot.export.union-content class=hard enforced=no tasks=data-model --> Not applicable | Exact Union rustdoc line `/// #[libertas_export(content = "payload")]` | Rust Union declaration | Emits parser-owned `UnionContentName` representation metadata. Do not invent `tag` or `untagged` forms. | [Rust guide](languages/rust.md#schema-attributes-and-field-presentation) |
| <!-- rule: annot.const-enum.file class=hard enforced=no tasks=data-model --> File-leading `/** @LibertasConstEnum */` | Not applicable | Dedicated TypeScript file containing one documented exported numeric constant per statement | Treats the constants as members of the numeric Enumeration exposed by a documented exported `typeof` namespace-import alias in the App entry module. | [TypeScript enumerations](languages/typescript.md#enumerations) |

## Function-level task declarations

| TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
| --- | --- | --- | --- | --- |
| `@LibertasSingleton` | `#[libertas_singleton]` | Public App function | Emits `Singleton=true`; at most one task for that App function may exist on a Hub. | [Functions, permissions, and singleton tasks](app-contract.md#functions-permissions-and-singleton-tasks) |
| `@LibertasDataErrorStatus` | `#[libertas_data_error_status]` | Public App function | Enables checked Data API status returns; omission keeps Hub-owned termination. | [Data failure contract](protocols.md#explicit-error-status-opt-in) |
| <!-- rule: annot.permissions.placement class=hard enforced=yes tasks=data-model title="Permissions resolves one positional constant on a public App function or server Endpoint field" --> `@LibertasPermissions(NAME)` | `#[libertas_permissions(NAME)]` | Public App function, or a server Endpoint field | Resolves one positional constant containing permissions and emits `Permissions`. Function permissions require nonempty same-key package string-resource reasons. Server Endpoint permissions are downstream requirements. | [Function and Endpoint permissions](protocols.md#function-and-endpoint-permissions) |
| `@LibertasAccessHost("api.example.com,{network_access_local}")` | `#[libertas_access_host("api.example.com,{network_access_local}")]` | Public App function only | Emits `AccessHost`. Every concrete DNS name or reserved local/external grant requires a nonempty same-key package string-resource reason. TCP and UDP are otherwise denied. | [App network access contract](network-access.md) |
|  `@LibertasDataSchema(key=DataKey,value=StoredData)` | `#[libertas_data_schema(key = DataKey, value = StoredData)]` | Public App function only | Requires exactly one key and one value Union reference and emits `FunctionDataSchema=key=DataKey,value=StoredData`. Both references resolve locally or to external App FQNs. Studio marks the value Union `LibertasData=true`. Both languages associate persistence on the function; ordinary native Union declarations remain unchanged. A bare `@LibertasDataSchema` on a Union is unsupported; neither language uses a standalone Union marker. | [Persistent data schemas](protocols.md#persistent-data-schemas) |
| `@LibertasValidationRules('expression : "KEY";')` | `#[libertas_validation_rules(r#"expression : "KEY";"#)]` | Public App function, or any schema data node (including fields, variants, and named types) | Emits semantic `ValidationRules`; field-local, inherited-field, and referenced named-type rules are cumulative and do not replace final structural/mandatory checks. Every pure Boolean expression uses `expression : "KEY";` with a quoted package `StringResources` error-message key. A function rule sees one synthetic Table containing all arguments. The language includes typed first-non-null fallback and no general truthiness. `FinalOnly=true;` or `FinalOnly=false;` is an optional leading directive inside this string, never a separate annotation; omission means `false` (incremental and final). | [ValidationRules](validation-rules.md#1-purpose) |

<!-- rule: annot.function.no-other-attributes class=hard enforced=yes tasks=data-model title="Functions accept only the public function attributes, never field, protocol-role, chart or Matter annotations" -->
No other portable schema attribute is public on a function. In particular,
field presentation, protocol-role, chart, and Matter annotations do not become
function annotations.

## Persistent-data Union declaration

Use the DataSchema entry in [function-level task declarations](#function-level-task-declarations).
The [persistent-data contract](protocols.md#persistent-data-schemas) gives the
complete usage rules.

## Defaults, constraints, and ordinary presentation

| TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
| --- | --- | --- | --- | --- |
| `@LibertasDefault(value)` | `#[libertas_default(value)]` | Schema data field/type | Emits editable initial `Default` data; on a Nullable field the reserved `$null` is the null default for every type, written `#[libertas_default($null)]` in Rust (an unquoted `None` on an `Option<T>` field is a parser error; a quoted `"None"` is the word; [reserved Default values](schema.md#reserved-default-values)); in Rust `$NAME` names a crate `const` ([constant references](languages/rust.md#constant-references)); includes explicit `User`/`List<User>` identity selectors and, on a Union field, the native name of the variant initialization selects. On an Endpoint client, the value instead recommends a `developer-package::function` task process when no eligible server is installed. | [User and group defaults](schema.md#user-and-group-defaults), [Common field metadata](presentation.md#common-field-metadata), [Declaring an Endpoint](protocols.md#declaring-an-endpoint) |
| `@LibertasFixed(value)` | `#[libertas_fixed(value)]` | Schema data field/type | Emits one fixed encoded value. On an Endpoint client, the value instead is a `developer-package::function` task-process identity: the Endpoint selector remains editable, compatible servers are restricted to that process, and its package becomes a dependency. Endpoint `Fixed` cannot coexist with `Default`. Native enum declarations generate member fixed values; Union declarations generate separate Discriminator values. On `LibertasUser`, Fixed is a [Hub-enforced security constraint](schema.md#fixed-current-user-security-constraint); only `current-user` is valid, at a request root or within nested request values outside Lists, optionally Hidden. | [Schema attributes](schema.md#43-value-constraint-and-presentation-attributes), [Declaring an Endpoint](protocols.md#declaring-an-endpoint) |
|  `@LibertasNumber(type=u32, min=..., max=..., step=...)` | `#[libertas_number(min=..., max=..., step=...)]` on a concrete numeric type | Number or DateTime field/type | Emits NumberType and numeric constraints. Unquoted `min`/`max` values emit `NumberMin`/`NumberMax`; quoted Data Validation expressions emit `RangeMin`/`RangeMax`, including current-row bounds such as `max="[^/maximum]"` under the [dynamic range contract](validation-rules.md#31-dynamic-numeric-ranges). `step` remains fixed; in Rust `min`, `max` and `step` accept a `$NAME` [constant reference](languages/rust.md#constant-references). TypeScript requires `type=`: `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`, `u128`, `i128`, `f32`, or `f64`. DateTime requires `u64`. Rust derives type from its native declaration and does not accept a public `type=` parameter. Integer step defaults to 1 when omitted. | [Type model](app-contract.md#type-model), [TypeScript annotations](languages/typescript.md#supported-jsdoc-decorators), [Rust annotations](languages/rust.md#schema-attributes-and-field-presentation) |
|  `@LibertasSize(min=..., max=...)` | `#[libertas_size(min=..., max=...)]` | String, Bytes, or List field/type | Emits inclusive nonnegative `SizeMin`/`SizeMax` (Rust bounds accept a `$NAME` [constant reference](languages/rust.md#constant-references)): List element count, ordinary value String UTF-8 byte count, or Bytes byte count. Keys use static UTF-8/raw byte bounds; explicit equal bounds select exact bytes under the [key rules](protocols.md#storage-schema-declarations). Empty values have size zero; nullable null is handled separately. Tables are not valid hosts. | [Common field metadata](presentation.md#common-field-metadata) |
|  `@LibertasPattern("pattern")` or `@LibertasPattern($CONST)` | `#[libertas_pattern(r"pattern")]` or `#[libertas_pattern($CONST)]` | Effective String field, list element, or named type, including nullable String | Emits `Pattern`; full-string Unicode scalar matching, with no trimming or conversion. TypeScript uses one JSON double-quoted literal or a native String constant; Rust uses one native cooked/raw String literal or String constant. Studio owns shared semantics. | [Patterns and complete escape rules](validation-rules.md#132-full-string-patterns) |
|  `@LibertasUnique` or `@LibertasUnique(level)` | `#[libertas_unique]` or `#[libertas_unique(level)]` | Scalar field inside one or more ancestor Lists | Emits `Unique`; omitted level means `99`, and a positive level selects the ancestor List scope. | [Common field metadata](presentation.md#common-field-metadata) |
| `@LibertasUnordered` | `#[libertas_unordered]` | List field/type | Emits `Unordered=true`; List order has no semantic meaning. | [Common field metadata](presentation.md#common-field-metadata) |
| `@LibertasMultiline` | `#[libertas_multiline]` | String field/type, including Nullable String and List element | Emits `Multiline=true`; line breaks are stored as LF (`\n`). A String without it is single-line. See [editing](presentation.md#common-field-metadata) and [AI Import hints](llm-document-transformation.md#value-representation-hints). | [Schema attributes](schema.md#43-value-constraint-and-presentation-attributes) |
| `@LibertasSecret` | `#[libertas_secret]` | String field/type, including Nullable String and List element; not with Multiline, Key or EnumSource | Emits `Secret=true`; the value is confidential, such as a password, PIN, API key or token. Clients mask it with a Show control; AI Import and AI tool results withhold it. Storage and encoding are unchanged. | [Secret values](presentation.md#secret-values), [Schema attributes](schema.md#43-value-constraint-and-presentation-attributes) |
| `@LibertasAskOnAdd` | `#[libertas_ask_on_add]` | Field with a value of its own in a List member Table, or a List whose member has one | Emits `AskOnAdd=true`; the person's **Add** asks this value first, in a platform-built step, and adds no member without it. Commonly paired with a template `lock`. | [Add-time values](presentation.md#add-time-values) |
| `@LibertasSuppressLabel` | `#[libertas_suppress_label]` | Scalar, struct, or Union field/type; List field/type whose member is a Table | Compiled into the [Display Template](presentation.md#display-template) as `suppress_label`: omit the selected row’s visual label while preserving its value and controls; on a List omit the List-contributed member name and position, preserving its own heading. | [Label-free value rows](presentation.md#suppresslabel-label-free-value-rows) |
| `@LibertasFoldable(initially_open=true)` | `#[libertas_foldable(initially_open = true)]` | Table, Union, or List type or field | Compiled into the [Display Template](presentation.md#display-template) as `fold`. Allows manual folding; unmarked containers remain expanded. The optional `initially_open` is true (the default), false, which starts the value collapsed without disabling folding, or `"nondefault"`, which starts it collapsed unless its value differs from its default. | [Folding](presentation.md#folding-structs-and-unions) |
|  `@LibertasInlineChoices` | `#[libertas_inline_choices]` | Enumeration or Union field or type declaration; not a Bitflags catalog | Compiled into the [Display Template](presentation.md#display-template) as `inline`. Clients show the permitted choices inline as radio controls after ConstrainedBy filtering, arranged by the [inline-choice ladder](presentation.md#inline-choice-ladder); a Union with a data-carrying branch stands one per line, its selected fields below the chosen choice or after the group as [inline choices](presentation.md#inline-choices) places them. A marker on either the field or its type enables it. | [Inline choices](presentation.md#inline-choices) |
| `@LibertasUIIncludeDescription` | `#[libertas_ui_include_description]` | Schema data field/type | Compiled into the [Display Template](presentation.md#display-template) as `include_description`. Clients show the localized long description inline together with the short display name for that UI node. It does not change data or wire encoding. | [Common field metadata](presentation.md#common-field-metadata) |
| `@LibertasReadOnly` | `#[libertas_read_only]` | Schema data field/type | Emits `ReadOnly=true`; clients present the value but do not offer ordinary editing. Every non-user writer (initialization, CopyFrom, EditRequest, calculation or link-return CopyTo, draft restoration) still writes it. | [Common field metadata](presentation.md#common-field-metadata) |
| `@LibertasHidden` | `#[libertas_hidden]` | Schema data field/type | Emits `Hidden=true`; the field remains typed transaction/context data but is hidden from ordinary visualization and editing. | [Common field metadata](presentation.md#common-field-metadata) |
| `@LibertasTimeInterval` | `#[libertas_time_interval]` | Numeric field/type | Emits `TimeInterval=true`; the raw number is a duration in seconds. | [Date and time](presentation.md#date-and-time) |
| `@LibertasDateOnly` | `#[libertas_date_only]` | Explicit integer Number field/type | Emits `DateOnly=true`; every value is a valid decimal Gregorian `yyyyMMdd`, never an epoch or timezone conversion. | [Date and time](presentation.md#date-and-time) |
| `@LibertasFormat("#,##0.00")` | `#[libertas_format("#,##0.00")]` | Number, DateOnly, DateTime, or TimeOnly field/type | Compiled into the [Display Template](presentation.md#display-template) as `format`: one programming-language-neutral Unicode LDML pattern; it changes display only. | [Display formats](presentation.md#display-formats) |
| `@LibertasMoney("USD", 2)` | `#[libertas_money("USD", 2)]` | Signed `i64` Number field/type | Declares exact integer minor units, explicit currency and fixed scale; enables localized currency display and inline entry. | [Money](presentation.md#money) |
| `@LibertasPhysicalUnit("millimeter-per-day")` | `#[libertas_physical_unit("millimeter-per-day")]` | Number field/type | Emits a canonical CLDR core `PhysicalUnit`; it declares the supplied unit and performs no conversion. | [Physical units](presentation.md#physical-units) |
| `@LibertasBitflags` or `@LibertasBitflags(Access)` | `#[libertas_bitflags]` or `#[libertas_bitflags(Access)]` | Bitflag catalog declaration, or integer field selecting that catalog | Emits `Bitflags=true` on the Enumeration catalog or `Bitflags=Access` on an ordinary integer. Values remain integer masks. | [Bitflags](presentation.md#bitflags), [TypeScript Bitflags](languages/typescript.md#bitflag-catalogs-and-integer-fields), [Rust Bitflags](languages/rust.md#bitflag-catalogs-and-integer-fields) |
| <!-- rule: annot.enum.use-native-enum class=hard enforced=no tasks=data-model --> `@LibertasEnum(A, B, C)` | `#[libertas_enum(A, B, C)]` | Applicable String alias/field or Enumeration declaration supported by that language | Defines a catalog using positional members. The parsers currently cannot attach source documentation to those fabricated members, so codebots must use a documented native numeric enum instead and must not generate this otherwise public compatibility form. TypeScript `values=[...]` is not canonical syntax. | [TypeScript enumerations](languages/typescript.md#enumerations), [Rust attributes](languages/rust.md#schema-attributes-and-field-presentation) |
| <!-- rule: annot.enumsource.unmatched-unanswered class=hard enforced=no tasks=data-model --> `@LibertasEnumSource("$.choices")` | `#[libertas_enum_source("$.choices")]` | Unsigned integer field/type; the key's String or integer type when the target List declares a key; or a List of that type | Emits `EnumSource`; the host stores the zero-based index of an item in the resolved same-tree List or, when that List's member declares a `Key` field, the chosen member's key value. A stored value matching no current member is unanswered and never remapped. An unanswered non-Nullable host whose source offers exactly one selectable member takes that member. A List host stores several distinct chosen indices or keys. | [Common field metadata](presentation.md#common-field-metadata), [EnumSource keys](schema.md#enumsource-keys), [EnumSource List hosts](schema.md#enumsource-list-hosts) |
| `@LibertasKey` | `#[libertas_key]` | Exactly one direct non-Nullable String or integer Number field of a Table used only as a List member; the field also carries `Unique` and may be Hidden | Emits `Key=true`. An `EnumSource` host targeting that List stores the member's key instead of its index and must have the key's type. Uniqueness comes from the required `Unique`; a List without a key keeps index storage, and stored values are never migrated. | [EnumSource keys](schema.md#enumsource-keys) |
| `@LibertasFormattedText` | `#[libertas_formatted_text]` | Statically ReadOnly schema data node with effective Bytes type | Emits `FormattedText=true`. The canonical LMF1 tuple follows the owning contract's static placement, request-value sources, and runtime-use restrictions. | [String resources and formatted output](presentation.md#string-resources-and-formatted-output) |
| `@LibertasMimeTypes("image/png,image/jpeg")` | `#[libertas_mime_types("image/png,image/jpeg")]` | Bytes field or named type outside CacheIdentity | Emits a MIME filter for the [LMB1 Bytes contract](schema.md#mime-aware-bytes). Its source grammar, content-size behavior, and incompatibility with FormattedText are validated by Studio. | [MIME-aware Bytes annotation](#mime-aware-bytes-annotation) |
|  `@LibertasLLMDocTrans(instruction="...", validation="@.facts", doc_type="image/*", accept_multiple_inputs=true, duplicate_identity="@.payer_tin,@.recipient_tin,@.amount", form_date_order="MonthDayYear")` | `#[libertas_llm_doc_trans(instruction = "...", validation = "@.facts", doc_type = "image/*", accept_multiple_inputs = true, duplicate_identity = "@.payer_tin,@.recipient_tin,@.amount", form_date_order = "MonthDayYear")]` | Named Table struct declaration used in an editable Request or SubscriptionRequest | Declares one per-occurrence document action. The [instruction argument](llm-document-transformation.md#app-schema-authoring) comes with an optional validation path, MIME filter, multiple-file choice and `duplicate_identity="@.payer_tin,@.recipient_tin,@.amount"`. [Duplicate identity](llm-document-transformation.md#declared-duplicate-identities) identifies one printed paper by selected scalars or whole List/Union values. [`form_date_order`](llm-document-transformation.md#app-schema-authoring), `"MonthDayYear"` or `"DayMonthYear"` with no default, is the order in which every DateOnly result of the form reads a numeric printed date. Direct marked children select scalar or nested composite results, excluding Hidden and Fixed subtrees. A Table whose variants are all `LLMDocTrans` is a classifier: it sorts several files into the enclosed forms and imports each with its own declaration; validation and multiple-file choice are invalid on it. | [Document Schema authoring](llm-document-transformation.md#app-schema-authoring), [categorized import](llm-document-transformation.md#categorized-import) |
|  `@LibertasLLMDocField` or `@LibertasLLMDocField(allowEmpty=false)` | `#[libertas_llm_doc_field]` or `#[libertas_llm_doc_field(allow_empty = false)]` | Direct result child of an LLMDocTrans Table, or a field within a selected composite | Selects any supported Libertas value, including complete nested Tables, Lists and Unions; nested scalar markers override blank conversion. Non-Nullable Number defaults `allowEmpty` to true; other values default false. True requires a typed Schema Default (a variant name on a Union) and a string projection, mapping exact `""` to that Default. `blank=VALUE` (Rust `blank = VALUE`) instead declares the value exact `""` applies, spelled as a Default literal: `$null` on a Nullable field, `0`, or a Union variant such as `ShortTerm`; it is invalid with `allowEmpty`. A Union imports a variant's payload fields only when marked; with none marked in any variant it imports the variant name alone. An unchanged variant keeps its payload; a new one, including a blank variant, is initialized as an editor choice, required unmarked fields left unanswered. | [Complete conversion contract](llm-document-transformation.md#complete-json-projection-and-client-conversion), [declared blank value](llm-document-transformation.md#declared-blank-value) |
|  `@LibertasFormCode("F")` | `#[libertas_form_code("F")]` | Enumeration member, Union variant or nonzero Bitflags flag constant of a type imported by AI Import | Emits `FormCode=F`: the exact code the form prints for that choice or box, used for the permitted-code list sent to the model and for exact matching of its reply; a Bitflags field whose flags declare codes is read as an array of codes ORed into the mask. Nonempty after trimming and unique among sibling members, variants or flags; the zero flag has none. On a Rust flag `const` import `libertas_macros::libertas_form_code`. Label, display and stored value are unchanged. | [Declared form codes](llm-document-transformation.md#declared-form-codes) |
| `@LibertasConstrainedBy("$.allowed_choices")` | `#[libertas_constrained_by("$.allowed_choices")]` | Enumeration or Union request field; Response variant | A hidden integer list, at a root path or (field form) a parent-relative `^` or [typed ancestor](schema.md#typed-ancestors-and-bound-occurrences) `^<Type>` path such as a row's own sibling, constrains field choices or the response NextRequest actions; the list is written by CopyFrom, EditRequest, or a calculation or link-return CopyTo, never by the user. | [Backend-constrained choices](protocols.md#backend-constrained-choices) |
| `@LibertasActionControl("sections")` | `#[libertas_action_control("sections")]` | Hidden direct integer-list field of a displayed Response or SubscriptionData payload with NextRequest | Declares a named control whose current Request membership is the field's original protocol Union discriminants; `Actions` restricts the built-in button. An empty array hides its control, and malformed arrays reject the update. | [Named action controls](protocols.md#named-action-controls) |
|  `@LibertasCopyFrom("$.first,$.fallback")` | `#[libertas_copy_from("$.first,$.fallback")]` | Protocol Request/SubscriptionRequest data | Emits source paths under the [shared copy rules](schema.md#shared-copyfrom-and-copyto-rules); the [context table](schema.md#context-and-syntax) selects the reference context and grammar. | [Shared CopyFrom and CopyTo rules](schema.md#shared-copyfrom-and-copyto-rules) |
| <!-- rule: dynamic.annotation.source class=hard enforced=yes tasks=data-model title="Dynamic accepts one quoted positional string in TypeScript and one Rust string literal" --> `@LibertasDynamic('Editable = expression;')` | `#[libertas_dynamic(r#"Editable = expression;"#)]` | Schema data field/type in an editable request; Addable requires a List; List-field Removable binds to each member | Emits `Dynamic`; independent typed operation properties combine across field/type declarations, with duplicate properties rejected. | [Dynamic](#dynamic) |
| `@LibertasRangeMin("expression")` | `#[libertas_range_min("expression")]` | Number or DateTime field/type | Emits `RangeMin`; one Data Validation bound expression compatible with the host type supplies an inclusive live lower bound for GUI and final validation. Incompatible result types are errors; null means no bound. | [Dynamic numeric ranges](validation-rules.md#31-dynamic-numeric-ranges) |
| `@LibertasRangeMax("expression")` | `#[libertas_range_max("expression")]` | Number or DateTime field/type | Emits `RangeMax`; one Data Validation bound expression compatible with the host type supplies an inclusive live upper bound for GUI and final validation. Incompatible result types are errors; null means no bound. | [Dynamic numeric ranges](validation-rules.md#31-dynamic-numeric-ranges) |

<!-- rule: annot.combinations-not-implied class=judgment enforced=no tasks=data-model -->
Attribute combinations and inherited effective metadata are validated by
Studio. Do not infer that two annotations can coexist merely because they have
separate rows in this catalog. <!-- /rule -->
The [ValidationRules entry](#function-level-task-declarations) also covers
data fields, variants, and named types.

## Endpoint and protocol annotations

| TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
| --- | --- | --- | --- | --- |
| `@LibertasEndpointSchema(Protocol)` | `#[libertas_endpoint_schema(Protocol)]` | `LibertasEndpoint` field | Resolves a local or external Union and emits `ProtocolSchema`. The Endpoint's absence/presence of the server marker determines its side. | [Declaring an Endpoint](protocols.md#declaring-an-endpoint) |
| `@LibertasEndpointServer` | `#[libertas_endpoint_server]` | `LibertasEndpoint` field | Emits `EndpointServer=true`. Without it the Endpoint is a client. | [Declaring an Endpoint](protocols.md#declaring-an-endpoint) |
| `@LibertasEndpointBaseObjects("$.device,$.room")` | `#[libertas_endpoint_base_objects("$.device,$.room")]` | Endpoint field | Emits comma-delimited `BaseObjects` paths that bind the Endpoint instance to configuration objects. | [Declaring an Endpoint](protocols.md#declaring-an-endpoint) |
|  `@LibertasRequest` | `#[libertas_request]` | Protocol Union variant | Emits `Request=true` for one-shot client-to-server input. | [Protocol roles](protocols.md#protocol-roles) |
|  `@LibertasResponse` | `#[libertas_response]` | Protocol Union variant | Emits `Response=true` for a correlated server-to-client result. | [Protocol roles](protocols.md#protocol-roles) |
| `@LibertasError` | `#[libertas_error]` | Protocol Union variant that also has an effective Response role | Emits `Error=true`; clients handle the response out-of-band of the current state transition, preserve the current interaction, and keep retry available. | [Protocol roles](protocols.md#protocol-roles) |
| `@LibertasRevealPath` | `#[libertas_reveal_path]` | Nullable String field of a row type shown read-only, or directly on a Request payload; at most one per type | No argument; emits `RevealPath=true`. On a row, names a field of the editor showing the row, revealed and briefly highlighted on activation without a request. On a Request, the URL-fragment counterpart: remembered when sent, readable by the Hub, revealed on the page the reply lands. | [In-page reveal paths](protocols.md#in-page-reveal-paths), [request reveal paths](protocols.md#request-reveal-paths) |
| `@LibertasLandingPath` | `#[libertas_landing_path]` | Hidden String field directly on a Request or Response payload, at most one per type; no `CopyFrom`; may be OutputOnly on a Request | No argument; emits `LandingPath=true`. Set only by the server; names the page's initial landing field; a remembered request reveal path that resolves takes priority; empty names none. | [Landing paths](protocols.md#landing-paths) |
| `@LibertasErrorFieldPath` | `#[libertas_error_field_path]` | Nullable String field of an Error-marked Response, at most one per variant | No argument; emits `ErrorFieldPath=true`. Optionally locates the rejected Request field for marking after the error dialog. | [Error field path](protocols.md#error-field-path) |
| `@LibertasOutputOnly` | `#[libertas_output_only]` | Statically ReadOnly field or named type used only in Request payloads; Nullable, String, Bytes or List; no `SizeMin` above 0 | No argument; emits `OutputOnly=true`. Clients skip its validation and send its empty value on Submit; display and copies are unchanged. | [Output-only request fields](protocols.md#output-only-request-fields) |
|  `@LibertasSubscriptionRequest` | `#[libertas_subscription_request]` | Protocol Union variant | Emits `SubscriptionRequest=true` for stream establishment or renewal. | [Protocol roles](protocols.md#protocol-roles) |
|  `@LibertasSubscriptionData` | `#[libertas_subscription_data]` | Protocol Union variant | Emits `SubscriptionData=true` for server-to-client stream data. | [Protocol roles](protocols.md#protocol-roles) |
| `@LibertasAccessPrivilege("Read,Write")` | `#[libertas_access_privilege("Read,Write")]` | Request or SubscriptionRequest variant | Emits the alternative-access bitmap (`Read=1`, `Write=2`, both=`3`). Omission also means both; mutating operations must explicitly require Write. | [Request access](protocols.md#request-access) |
| `@LibertasCalculationRequest("A,B")` | `#[libertas_calculation_request("A,B")]` | Protocol request data field | Emits a field's calculation variant list, making the field a trigger of those calculations under the [field calculation contract](protocols.md#field-calculation-requests). | [Field calculation requests](protocols.md#field-calculation-requests) |
| `@LibertasCalculationRoot("A,B")` | `#[libertas_calculation_root("A,B")]` | Root node: any request data field, record type, List member type, Union field or type, or the request's own payload type | Emits `{"Name":"CalculationRoot","Value":"A,B"}` on the node the named calculations are for, under the [calculation root](protocols.md#calculation-root) contract. | [Calculation root](protocols.md#calculation-root) |
| `@LibertasCopyTo("@.orderTotal")` | `#[libertas_copy_to("@.orderTotal")]` | Successful calculation or link-return Response payload field; reduced SubscriptionData variant in the optional App pattern | Emits destination paths under the [shared copy rules](schema.md#shared-copyfrom-and-copyto-rules). [Calculation results](protocols.md#applying-calculation-results), [link returns](protocols.md#link-return), and [reduced subscriptions](protocols.md#reduced-subscriptiondata-design-pattern) define application and lifetime. | [Shared CopyFrom and CopyTo rules](schema.md#shared-copyfrom-and-copyto-rules) |
| `@LibertasAppendTo("Entries.entries")` | `#[libertas_append_to("Entries.entries")]` | Incoming SubscriptionData root array field | Appends ordered complete elements to one admitted working variant's root array. Preserves duplicates; rejects conflicting writers and invalid resulting values atomically. No name-based replacement for annotated fields, including when another variant is named. Requires an initial snapshot. | [Array append](protocols.md#subscriptiondata-array-append) |
| `@LibertasAppendIndex("$.start_index")` | `#[libertas_append_index("$.start_index")]` | Same array field as AppendTo | Excludes the referenced control field from ordinary replacement and CopyTo; optional non-null integer report field checks the exact destination length before append, including empty appends; negative/mismatched positions reject the whole report. The App owns ordering, deduplication, retention, and initial snapshots. | [Array append](protocols.md#subscriptiondata-array-append) |
| `@LibertasDefaultRequest` | `#[libertas_default_request]` | Request variant whose payload needs no user input (Nil, or only hidden/read-only Fixed or Default fields) | Optionally selects the single explicit correlated request for endpoint entry; chart-producing requests are forbidden. | [Default entry](protocols.md#default-request-entry-and-dynamic-actions) |
| `@LibertasNextRequest("A,B")` | `#[libertas_next_request("A,B")]` | Protocol Union variant | Emits the allowed next client variant native names as `NextRequest`. Outside workflows, omission of both NextRequest and WorkflowRequest is a non-transition acknowledgement; workflow completion can be terminal. | [Chaining, caching, and copied values](protocols.md#chaining-caching-and-copied-values) |
| `@LibertasWorkflowRequest("EditProfile")` | `#[libertas_workflow_request("EditProfile")]` | Response variant | Declares the `WorkflowRequest` semi-role. | [WorkflowRequest semi-role](protocols.md#workflowrequest-semi-role) |
| `@LibertasPrevRequest("ReopenProfile,ReopenAddress")` | `#[libertas_prev_request("ReopenProfile,ReopenAddress")]` | Request variant | Lists ordinary Back actions; their CopyFrom source is the current editor and each action must independently validate before sending. Their controls are suppressed in an embedded master-detail trailing pane under the pane contract. | [Workflow interactions](protocols.md#workflow-interactions) |
| `@LibertasLink("OpenTopic", stacked=true)` | `#[libertas_link("OpenTopic", stacked = true)]` | Any field; Table, Union, or Enumeration type declaration | Names one same-protocol target, constructed through CopyFrom with the tapped node or host field bound; the field's complete declaration wins over its type's. Optional Boolean `stacked` defaults to false; omission or false emits exactly `{"Name":"Link","Value":"OpenTopic"}` and preserves ordinary Request invocation and editor link-return CopyTo. True emits exactly `{"Name":"Link","Value":"OpenTopic,stacked=true"}`. The stacked form requires a LinkOnly, fully filled target with no editable input, an ordinary Request with display-only Response replies or a SubscriptionRequest with a live Response snapshot; see the owner for validation, Close, release and lifetime rules. Displayed-value targets must be in that variant's NextRequest; Hidden occurrences are rejected. | [Links](protocols.md#links) |
| `@LibertasLinkOnly` | `#[libertas_link_only]` | Request variant targeted by a declared Link in the same protocol; SubscriptionRequest variant targeted by a stacked Link | Emits `LinkOnly=true`. The target is offered only through declared links: NextRequest and the filtered action set still govern its link rows, while Actions, menus, endpoint landings, and automatic singleton selection never offer it or count it. Studio rejects it on any other node, on an untargeted variant, with DefaultRequest, and with ExcludeUI. Clients predating the marker keep offering it in Actions. | [Link-only requests](protocols.md#link-only) |
| `@LibertasRequestConfirmation` | `#[libertas_request_confirmation]` | Request variant | Emits `{"Name":"RequestConfirmation","Value":"true"}` on the variant, the same marker form as `DefaultRequest` and `LinkOnly`. Marks a critical Request: whenever a person's choice or submission is about to send it, the client first shows a dialog naming the action with Send and Cancel, from every entry point and after a popup or an editor is confirmed. Cancel sends nothing. For the AI assistant, the Hub starts the generated tool's description with "Requires user confirmation." and instructs the assistant, in every turn that offers such a tool, to state the action and its values and to call the tool only after the user confirms in a later message; the Hub does not verify the confirmation. Wire form and validation are unchanged; a program calling the endpoint directly is not affected, and the service still validates every request. Studio rejects it on any other node and on a DefaultRequest. | [Critical requests](protocols.md#request-confirmation) |
| `@LibertasNextResponse("A,B")` | `#[libertas_next_response("A,B")]` | Protocol Union variant | Declares allowed replies on requests, or directional Compatible SubscriptionData on SubscriptionData. | [Chaining, caching, and copied values](protocols.md#chaining-caching-and-copied-values) |
|  `@LibertasProtocolVariant("EditWages")` | `#[libertas_protocol_variant("EditWages")]` | Ordinary Enumeration member | Associates its typed value with one variant of each consuming Endpoint protocol; no navigation or wire change. | [Enum/protocol associations](schema.md#enumeration-protocol-associations) |
|  `@LibertasCacheIdentity` | `#[libertas_cache_identity]` | Direct field of a Request root record | Enables large-page draft recovery using all marked fields as immutable identity; optional app-maintained protocol revision invalidates older drafts. Small requests omit it. | [Workflow draft identity](#cacheidentity) |
| `@LibertasExcludeUI` | `#[libertas_exclude_ui]` | Endpoint server field carrying a resolved `ProtocolSchema`, or reachable protocol `Request`/`SubscriptionRequest` variant | Emits `ExcludeUI=true`; Hub Control omits the server from its UI-device list and LLM tool collection, while generated Control GUI and Hub LLM tool-definition generation omit the operation. Admin, protocol, routing, and wire behavior remain complete. All servers sharing one resolved `ProtocolSchema` root must use the same value. | [Declaring an Endpoint](protocols.md#declaring-an-endpoint) |
| `@LibertasAINavigationPrompt("NAVIGATION_PROMPT")` | `#[libertas_ai_navigation_prompt("NAVIGATION_PROMPT")]` | Protocol Union or page-defining protocol variant | Selects optional App guidance for AI Search through a nonempty package StringResource; guidance only improves interpretation, and availability comes from the task's Go to control. Union guidance precedes page guidance; both contribute. AI selects an eligible choice of the task's Go to control without supplying request fields. | [AI Search](protocols.md#ai-navigation-and-ai-search) |
| `@LibertasLLMToolCall(false)` / `@LibertasLLMToolCall(true)` | `#[libertas_llm_tool_call(false)]` / `#[libertas_llm_tool_call(true)]` | Protocol Union or descendant Request variant | Controls Hub LLM tool-definition generation. The nearest explicit Boolean value along the Union-to-variant path wins; omission defaults to true. `false` on a Union excludes its Request variants until a descendant explicitly sets `true`. It does not change Control UI, protocol messages, or authorization. | [AI tool calls](protocols.md#ai-tool-calls) |
| `@LibertasProtocolConformance("Read & Status")` | `#[libertas_protocol_conformance("Read & Status")]` | Protocol Union, or Union-typed protocol variant | Emits `ProtocolConformance`; client and server expressions independently declare mandatory compatible variants. | [Protocol conformance](protocols.md#protocol-conformance) |

`@LibertasDefault`/`@LibertasFixed` and their Rust counterparts have their
Endpoint-client meanings described in the defaults table. `Permissions` may
also appear on an Endpoint server as described in the function table.

## External types, Matter descriptors, and Union metadata

| TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
| --- | --- | --- | --- | --- |
| `@LibertasForeignType("developer-package::Type")` | `#[libertas_foreign_type(developer-package::Type)]` | Published top-level compatible named type | Emits `ForeignTypeSchema` naming its direct official external App type. The compatible type keeps its own identity and explicit attributes override inherited official attributes. | [External types and App dependencies](app-contract.md#external-types-and-app-dependencies) |
| <!-- rule: annot.device-type.generated class=hard enforced=no tasks=data-model title="DeviceType goes on a LibertasDevice field and carries the Studio-generated descriptor" --> `@LibertasDeviceType("descriptor")` | `#[libertas_device_type("descriptor")]` | Effective `LibertasDevice` field | Emits the Studio-generated physical Matter `DeviceType` descriptor. Do not invent the encoded descriptor manually. | [Matter](matter/descriptors.md#canonical-generation-and-decoding) |
| <!-- rule: annot.virtual-device-type.generated class=hard enforced=no tasks=data-model title="VirtualDeviceType goes on a LibertasVirtualDevice field and carries the Studio-generated descriptor" --> `@LibertasVirtualDeviceType("descriptor")` | `#[libertas_virtual_device_type("descriptor")]` | Effective `LibertasVirtualDevice` field | Emits the Studio-generated `VirtualDevice` descriptor. Do not invent it manually. | [Matter](matter/descriptors.md#canonical-generation-and-decoding) |
| <!-- rule: annot.discriminator.typescript-only class=hard enforced=no tasks=data-model title="Discriminator is TypeScript-only, for a discriminated Union whose automatic detection is ambiguous" --> `@LibertasDiscriminator("kind")` | Not applicable | TypeScript discriminated Union when automatic detection is ambiguous | Emits parser-owned `UnionTagName`. Normal TypeScript literal/enum discriminators are detected automatically; Rust uses enum declaration order and native representation. | [Protocol roles](protocols.md#protocol-roles) |

## Two-dimensional chart annotations

| TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
| --- | --- | --- | --- | --- |
|  `@LibertasChart(mark)` | `#[libertas_chart(mark)]` | Response/SubscriptionData chart payload or composition | Emits a primitive mark or composition as `Chart`. | [Charts](presentation.md#charts) |
|  `@LibertasChartChannel(x, tooltip)` | `#[libertas_chart_channel(x, tooltip)]` | Direct chart data field | Emits its canonical `ChartChannel` roles. | [Charts](presentation.md#charts) |
|  `@LibertasChartScale(id=series, kind=linear, ...)` | `#[libertas_chart_scale(id = series, kind = linear, ...)]` | Scale-bearing chart data field | Emits `ChartScale`; source `kind` is Compiled into the [Display Template](presentation.md#display-template) as the `scale` hint, and the other properties stay in the schema. | [Charts](presentation.md#charts) |
|  `@LibertasChartGuide(target=x, source=scale, ...)` | `#[libertas_chart_guide(target = x, source = scale, ...)]` | Explicit guide binding or guide-label field | Emits `ChartGuide`; `target` and `source` are required. `position` is Compiled into the [Display Template](presentation.md#display-template) as the `guide` hint. | [Charts](presentation.md#charts) |

The complete mark, channel, scale, guide, composition, visible-text, and
hidden-context rules are in [Presentation and localization](presentation.md#charts)
and the normative [client-side chart specification](charts/2d.md).

## Three-dimensional chart annotations

| TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
| --- | --- | --- | --- | --- |
|  `@LibertasChart3D(mark)` | `#[libertas_chart3d(mark)]` | 3D Response/SubscriptionData chart payload or composition | Emits a 3D primitive mark or composition as `Chart3D`. | [3D charts](presentation.md#3d-charts) |
| `@LibertasChart3DChannel(x, z, tooltip)` | `#[libertas_chart3d_channel(x, z, tooltip)]` | Direct 3D chart data field | Emits canonical `Chart3DChannel` roles. | [3D charts](presentation.md#3d-charts) |
| `@LibertasChart3DScale(id=series, kind=linear, ...)` | `#[libertas_chart3d_scale(id = series, kind = linear, ...)]` | Scale-bearing 3D chart data field | Emits `Chart3DScale`; source `kind` is Compiled into the [Display Template](presentation.md#display-template) as the `scale` hint, and the other properties stay in the schema. | [3D charts](presentation.md#3d-charts) |
| `@LibertasChart3DGuide(target=z, source=scale, ...)` | `#[libertas_chart3d_guide(target = z, source = scale, ...)]` | Explicit 3D guide binding or guide-label field | Emits `Chart3DGuide`. | [3D charts](presentation.md#3d-charts) |
|  `@LibertasChart3DView(projection=perspective, ...)` | `#[libertas_chart3d_view(projection = perspective, ...)]` | One physical 3D chart scene | Emits `Chart3DView` camera and projection settings. It does not define a gesture mode. | [3D charts](presentation.md#3d-charts) |

The normative geometry, topology, camera, interaction, fallback, and resource
rules are in [Client-side 3D charting](charts/3d.md).

## Inferred and generated metadata: never invent source annotations

<!-- rule: annot.generated.no-source-spelling class=hard enforced=no tasks=data-model title="Generated or inferred schema attributes have no source annotation of their own" -->
The final schema contains attributes and fields that are inferred from native
source or generated from one of the public annotations above. Their presence in
schema JSON does not create another public macro or decorator.

| Generated or parser-owned result | Source of the result |
| --- | --- |
| `Nullable=true` | TypeScript optional/nullish supported type, or Rust `Option<T>`. There is no public `LibertasNullable` annotation. |
| Rust `NumberType` | Concrete Rust `u*`, `i*`, `f32`, or `f64`. TypeScript obtains it through `LibertasNumber`; never write a direct NumberType annotation. |
| `NumberMin`, `NumberMax`, `NumberStep`, `SizeMin`, `SizeMax` | Emitted components of the public Number and Size annotations. Their schema names are not source spellings. |
| `LengthMin`, `LengthMax` | Retained schema fields from the removed TypeScript String spelling. Current source uses SizeMin/SizeMax through the Size annotation. |
| `Fixed` on enum members; `Discriminator` on Union members; `UnionTagField` on a ScriptLanguage-only native tag field (such as TypeScript/Python; never Rust) | Native enum values and TypeScript discriminated-Union literals. Do not duplicate them with source annotations. |
| `UnionTagName` | Automatic TypeScript discriminator detection or `@LibertasDiscriminator`. There is no direct UnionTagName decorator. |
| `UnionContentName` | The Rust `libertas_export(content = "...")` rustdoc directive. There is no direct UnionContentName attribute. |
| `LibertasProtocol=true` | Injected for a Union reached by an Endpoint `ProtocolSchema`. There is no public LibertasProtocol source marker. |
| `LibertasData=true` | Injected by Studio for the value Union in a function `FunctionDataSchema` binding. Both languages use their function-level DataSchema annotation; neither has a public standalone Union marker. |
| `EnumValues` | Parser metadata produced while constructing an Enumeration from the language's public enum syntax. Do not write it directly. |
| `FullyQualifiedName` and source locations | Transient parser resolution/diagnostic metadata. Studio removes it before final schema publication. |
| `AppDependencies` | Studio deployment planning from resolved external App types and Endpoint-client Fixed packages. Never populate it in App source. |
| <!-- rule: annot.no-std.native class=hard enforced=no tasks=data-model --> Rust `no_std=true` | Native crate attribute `#![no_std]`, not a Libertas helper macro. Use it only when the App and all dependencies support that build model. |

The emitted-name column throughout this document is explanatory. <!-- rule: annot.generated.no-derived-spellings class=hard enforced=no tasks=data-model title="Never derive a source annotation spelling from a schema JSON property" --> Never derive a
new `@Libertas...` or `#[libertas_...]` spelling from a schema JSON property.

## Not public or not usable

<!-- rule: annot.removed.never-emit class=hard enforced=no tasks=data-model title="Never emit a removed or unsupported annotation spelling" -->
Codebots must not emit any spelling in this section.

### Removed annotations

- `@LibertasString` is removed; use `@LibertasSize(min=..., max=...)` for String,
  List, or Bytes bounds. Existing serialized LengthMin/LengthMax values remain
  recognizable, but are not generated by a current public source annotation.
- `@LibertasFunctionDataSchema` is removed; use `@LibertasDataSchema(key=DataKey,value=Type)` on
  the public function. The old bare Union marker is also removed. Studio owns
  the generated LibertasData marker; do not rewrite retained schema bytes.

- `@LibertasInteger` and `@LibertasFloat` are removed source spellings. Use the
  single `@LibertasNumber(type=..., min=..., max=..., step=...)` annotation.
  Existing serialized NumberType/NumberMin/NumberMax/NumberStep fields are
  unchanged; regenerate source artifacts rather than editing retained schemas.

- `@LibertasLibrary` and `#[libertas_library]` are recognized only so Studio can
  report that the `Library` schema attribute was removed.
- `@LibertasUseOriginalDocument` and `#[libertas_use_original_document]` were
  removed. Published compatible types share the official type document
  automatically through `ForeignType`.
- `@LibertasExternalNativeType` was removed. Native external ownership is
  inferred from resolved language imports and exact-name App exports.
- `@LibertasUIHeader` and `#[libertas_ui_header]` (schema attribute
  `TableHeader`) were retired, and Studio rejects them with a message that
  names the replacement: a Display Template
  [`promote`](presentation.md#promotion-flattening-and-union-selectors)
  rule selecting the field to promote, with the same `use_parent_name`
  option, or a [`flatten`](presentation.md#flatten) rule selecting the struct,
  List, or Union to flatten.
- `@LibertasUIName` and `#[libertas_ui_name]` were retired, and Studio rejects
  them with a message that names the replacement. A data-titled row is a
  Display Template [`label`](presentation.md#labels-and-titles) rule on the row's struct
  with its value from a `promote` rule on a child
  ([data-titled rows](presentation.md#data-titled-rows)).

### Unsupported authoring forms

Use the canonical forms in the catalog. In particular:

-  `#[libertas_ignored]` belongs on its exact outer-rustdoc line; it is not a
  standalone SDK macro; see [Rust discovery](languages/rust.md#project-contract-and-library-requirements).
-  TypeScript functions use `@LibertasDataSchema(key=DataKey,value=Type)`; both named arguments are
  required, and a bare Union marker is invalid; see the [function DataSchema contract](protocols.md#persistent-data-schemas).
- Use documented native numeric enums. Fabricated annotation members cannot
  carry required member documentation.
- <!-- rule: annot.named-argument-variations class=hard enforced=no tasks=data-model title="Named-argument variations do not create additional supported annotation spellings" --> Named-argument variations do not create additional supported spellings.

### Removed Rust SDK helpers

<!-- rule: annot.rust-sdk-helpers.removed class=hard enforced=no tasks=data-model -->
The SDK helper allowlists no longer admit `libertas_string`, `libertas_array`,
or `libertas_cacheable`, because the owning schema parser does not support
those source forms. Use `libertas_size` for size bounds and the documented
CacheIdentity behavior where needed. `libertas_discriminator` is also not a
Rust schema annotation; Rust uses native enum representation.

## MIME-aware Bytes annotation

Apply `@LibertasMimeTypes("image/png,image/jpeg")` or
`#[libertas_mime_types("image/png,image/jpeg")]` to a Bytes field or named type
to restrict its media types. The [MIME-aware Bytes contract](schema.md#mime-aware-bytes)
defines the filter, LMB1 encoding, valuation, size, copy, and submission rules. <!-- rule: annot.mime-types.semantic-choice class=judgment enforced=no tasks=data-model -->
Keep unannotated Bytes fields for raw binary protocols; `MimeTypes` is a
semantic choice for media values, not a replacement for ordinary Bytes.

## CacheIdentity

`CacheIdentity` opts large workflow requests into persistent local draft recovery.
TypeScript uses `@LibertasCacheIdentity`; Rust uses `#[libertas_cache_identity]`.
Both emit `CacheIdentity=true`. <!-- rule: cache.clients-implement class=hard enforced=no tasks=data-model --> Use clients that implement this attribute;
a client that does not provides no draft recovery.

### Eligibility and field placement

- <!-- rule: draft.eligibility.small-requests class=hard enforced=no tasks=workflow --> `CacheIdentity` is a Boolean field attribute. A workflow page is cacheable
  only when at least one root-record field has effective `CacheIdentity=true`.
  Use it for large workflow requests where losing completed field edits would
  require substantial re-entry. Small requests must omit CacheIdentity; clients
  do not guess eligibility from payload size or field count.
- <!-- rule: cache.identity.tuple class=hard enforced=no tasks=data-model --> Multiple marked fields form one identity tuple. All must match; they are not
  alternative identities. The client does not infer identity from field names,
  Hidden, or CopyFrom.
- <!-- rule: cache.identity.placement class=hard enforced=yes tasks=data-model title="Mark only direct fields of the request root record; nested, array and Union-branch fields cannot carry CacheIdentity" --> Mark only direct fields of the request variant's root record. A referenced
  record used as that root payload qualifies. Nested record fields, array
  elements, and fields in selectable Union branches cannot carry the attribute.
  Studio validates placement for every request use, including referenced types.
- <!-- rule: cache.identity.defined-immutable class=hard enforced=no tasks=data-model title="CacheIdentity values must be defined after initialization and stay immutable for the editor lifetime" --> Identity values must be defined after initialization; valid null is allowed
  and differs from undefined. They remain immutable for the editor lifetime,
  including against user edits and calculation writes. Restoring editable
  data preserves the freshly initialized identity values.

### Identity, revisions, and application responsibility

Capture the tuple after normal initialization or receipt of EditRequest data,
before user editing. Match every marked field by exact typed equality, including
null, numeric values, bytes, Union selections/payloads, and array order. <!-- rule: cache.identity.distinguishing-context class=judgment enforced=no tasks=data-model --> The
application must supply enough distinguishing context to prevent one workflow's
draft from matching another workflow. A workflow identifier is a typical choice.

<!-- rule: cache.identity.revision-recommendation class=judgment enforced=no tasks=data-model -->
**Recommendation:** include an application-maintained data/workflow revision
when changes to saved data invalidate earlier drafts. For example, a historical
page edit may invalidate later pages; changing their identity revision prevents
restoring obsolete local drafts. The client does not infer these dependencies.

<!-- rule: draft.revisions.application-owned class=hard enforced=no tasks=workflow -->
Protocol changes are allowed. Application data revisions are independent of
protocol/schema versions; tasks need not detect protocol changes. Management
tools guard schema compatibility, and clients reject incompatible cached data
at restoration. Compatible protocol changes alone do not invalidate drafts.

**Optional recommendation:** the application may maintain a protocol revision
and mark it as another CacheIdentity field. Advancing it guarantees rejection
of drafts with the old value even when a protocol change preserves structural
compatibility. Maintaining and advancing this revision is entirely the
application's choice and responsibility. Workflow revision, protocol revision,
or both may be used; the client assigns no special meaning to either field.
Without a distinguishing identity change, semantic changes that preserve the
schema's structure may remain undetectable.

### Draft scope and lifetime

<!-- rule: draft.identity.initialize-before-lookup class=hard enforced=no tasks=client-runtime -->
The full cache key comprises Hub identity, task ID, endpoint ID, request variant
index, and the complete CacheIdentity tuple, within the authenticated account's
storage. The first four components identify the page, not the entire cache key.
Different workflow identities and revisions retain separate drafts. Initialize
the page first, then look up its complete identity directly; unrelated entries
do not need to be scanned or deleted. Garbage collection removes expired drafts.

Persist the latest committed editor state after each change, creating the entry
if absent and updating it otherwise. This includes nested fields, array
operations, Union selection, null/undefined transitions, and atomic calculation
results. Drafts survive process exit and restart. <!-- rule: cache.draft.expiry class=hard enforced=no tasks=client-runtime --> Entries expire one calendar month
after their last committed update. Clients collect expired entries at startup
and reject expired entries before restoration. Reads and restore prompts do
not extend retention. Removing the Hub, task, or
endpoint removes its drafts.

<!-- rule: cache.draft.delete-after-reply class=hard enforced=no tasks=client-runtime -->
Delete a submitted editor's draft only after its successful correlated backend
reply. Transport acceptance alone is insufficient. Retain it on transport or
backend error. <!-- rule: cache.draft.calculation-replies-keep class=hard enforced=no tasks=client-runtime --> Calculation replies never clear the parent draft.

<!-- rule: cache.draft.go-back-and-close class=hard enforced=no tasks=client-runtime -->
When offered under the [pane contract](presentation.md#master-detail-panes),
the workflow's in-app **Go Back** action selects a declared PrevRequest and
voluntarily abandons the current edit. Delete that editor's draft after the
successful correlated, non-error reply to the previous action. Cancelling its
chooser, failing validation, or receiving a transport or backend error retains
the draft. An explicit editor **Close**, including Close in the right-hand
master-detail pane, deletes that editor's draft when the close is accepted.
These actions delete only the departing editor's matching draft; they do not
delete a retained leading editor's draft or other workflow identities.

<!-- rule: cache.draft.system-back-preserves class=hard enforced=no tasks=client-runtime -->
System-level Back by button or gesture, ordinary screen-navigation Back, and
process exit or interruption preserve the committed draft for recovery. They
are separate from the workflow's Go Back action. Closing a field picker or
nested editor returns to its parent and does not discard the parent's draft.
The system Close of a [stacked-link page](protocols.md#links) likewise releases
only that displayed page and preserves the live editor beneath; it is not an
explicit editor Close. Navigating away from the stack follows these existing
draft rules.

EditRequest does not itself delete a draft. If it is a successful reply to a
submission, that correlation still clears the submitted draft. Incoming page
initialization then follows the normal identity comparison and restoration
rules below.

### Restoration and calculations

Initialize the new editor first, including when entered through EditRequest.
Look up the entry for its full identity and verify the stored tuple. If it matches,
ask whether to restore. Declining removes the cache and keeps fresh initialization.
Accepting attempts restoration without submitting the workflow request. Ordinary
rerendering or returning from a field picker does not repeat the prompt.

<!-- rule: cache.restore.writable-values class=hard enforced=no tasks=client-runtime -->
Restoration assigns only the values the user can edit and the destinations
that a calculation or a link return writes; a link-return destination cannot
be recomputed, so it is restored as saved. Every other Hidden, ReadOnly, or Fixed value, including a
cookie and an allowed-set source that no calculation or link return writes, keeps the value
the fresh editor supplied; the draft's copy of it is not applied. A restored
selection outside the fresh allowed set becomes unanswered under the
[reset rule](protocols.md#calculated-allowed-sets) and does not discard the
draft.

<!-- rule: cache.restore.atomic-compatibility class=hard enforced=no tasks=client-runtime -->
Restore atomically against the current schema; a source that a calculation writes is
restored from the draft and recomputed by the seeded calculations. Corrupt
encoding or structural/type incompatibility discards the cache and rolls back
the entire attempt to the fresh editor. Compatibility includes the declared
primitive kind and representation, record fields, List element types and
available Union branches; a value of the wrong kind or an invalid encoding is
not a restorable draft.
Do not partially restore a structurally incompatible draft, coerce incompatible
data, or silently omit incompatible fields. An exact schema version match is
not required.

<!-- rule: cache.restore.local-reset class=hard enforced=no tasks=client-runtime -->
For a structurally compatible draft, check each restored node's own value
constraints, including numeric bounds evaluated in the current editor and
reference context under the [range contract](validation-rules.md#31-dynamic-numeric-ranges).
Replace an invalid value through ordinary initialization at that node,
including its applicable Default or an unanswered value. A descendant's
failure must not reset its containing record, List item or page; preserve every
other restored value. A List that itself violates its size constraint is
reset at that List. These local resets are part of the atomic
restoration and do not discard the compatible draft.

Undefined required fields are legitimate draft state. Preserve undefined
separately from null, false, zero, empty values, bytes, and Union selections.
A committed empty String is assigned, not undefined; if it violates its own
constraint, initialize that String normally, applying its Default when present.
Ordinary initialization may leave a value unanswered or still invalid.
Cross-field ValidationRules keep their normal evaluation, and final
ValidationRules run at Next/Submit; neither resets restored values or discards
the compatible draft. Submission stays blocked until the current editor is
complete and valid. This local recovery does not permit incomplete or invalid
Avro requests or backend persistence of unfinished requests.

<!-- rule: draft.restore.normal-calculation-scheduler class=hard enforced=no tasks=client-runtime -->
After successful restoration, seed calculations once for all current instances,
including array items. Reset destinations through ordinary initialization and
reevaluate using restored inputs. Use the normal ordered scheduler and dependency
propagation; incomplete or invalid requests cannot be sent. Do not restore old
in-flight transactions or automatically submit the workflow editor.

### General field-edit commitment

<!-- rule: edit.commit.finished-only class=hard enforced=no tasks=client-runtime -->
Only finished editing commits a user-entered field value. This applies throughout
client editors, not just caching. Intermediate control text neither replaces
the committed value nor triggers calculations or cache updates. Typing `-` in a
number control therefore does not turn an existing committed number into
undefined. An unassigned field remains undefined until a completed edit assigns
an admissible value.

<!-- rule: edit.commit.submission-resolves class=hard enforced=no tasks=client-runtime -->
Submission must resolve the active edit through this same boundary; it cannot
silently submit an old value while replacement text remains unfinished. Native
controls may use their normal completion actions. Programmatic initialization
and atomic calculation updates retain their transaction boundaries.

### MIME-aware Bytes in workflow drafts

<!-- rule: cache.mime.draft-link class=hard enforced=no tasks=client-runtime -->
For a [MIME-aware Bytes](schema.md#mime-aware-bytes) field, a draft never stores
the media content. It stores a client-local file link when one exists; an
in-memory value received from the server stores only a marker to use the
freshly initialized value after recovery. An explicit clear stores an
undefined marker. The link and its temporary file stay within the client and
are never sent to the Hub. A missing or unreadable link on recovery makes
only that field undefined and reports which field needs a new file; the other
edits restore. If no corresponding fresh value exists for an in-memory marker,
that field is undefined. This is an exception to full-value draft storage,
not a relaxation of identity or schema checks. <!-- rule: cache.mime.not-identity class=hard enforced=yes tasks=data-model title="MimeTypes cannot occur within a CacheIdentity field" --> `MimeTypes` cannot occur
within a `CacheIdentity` field because that identity is persisted. <!-- rule: cache.mime.temp-file-lifetime class=hard enforced=no tasks=client-runtime --> A temporary
file remains while an active editor or unexpired draft refers to it and is
automatically removed after its last reference and a cleanup grace period.

`NextResponse` on SubscriptionData declares [Compatible SubscriptionData](protocols.md#compatible-subscriptiondata), rather than correlated replies.

## Dynamic

`@LibertasDynamic('Editable = [status] == "draft";')` and
`#[libertas_dynamic(r#"Editable = [status] == "draft";"#)]` each emit one
`Dynamic` schema attribute whose value is the extracted assignment string.
Fields, member declarations, and named types use the placement rules in
[dynamic schema properties](schema.md#dynamic-schema-properties).

Rust ordinary strings decode native escapes; raw strings preserve their contents.
Thus `"Editable = [status] == \"draft\";"` and
`r#"Editable = [status] == "draft";"#` carry the same expression text.
TypeScript JSDoc accepts a single- or double-quoted positional string, removes
the outer quotes, trims surrounding expression whitespace, and preserves
backslashes. A backslash protects a matching outer quote during scanning; it
is not decoded as a JavaScript escape. Backticks, unquoted arguments, named
arguments, and multiple arguments are unsupported. Prefer single outer quotes
with double-quoted expression literals. JSON serialization escapes the extracted
text only for transport; it adds no expression decoding pass.

The [expression grammar](validation-rules.md#dynamic-property-expressions) owns
inner strings, assignment delimiters, and Boolean evaluation.
