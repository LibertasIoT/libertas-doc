# Designing the data model

Load this card when a task declares or changes App data: named types, fields
and their attributes, Enumerations and Unions, `EnumSource` choices and keys,
defaults and initialization, Libertas Schema Paths, `CopyFrom` or `CopyTo`,
or `CacheIdentity` drafts; an Endpoint and its protocol also need the
[Endpoint protocol card](endpoint_protocol.md). Its
rules are generated from the [data model reference](../reference/schema.md)
and the [source annotation reference](../reference/annotations.md), which own
them. The [annotation inventory](schema_annotations.md) gives each
annotation's Rust and TypeScript spelling; changing a published protocol or
stored representation also needs the [data-type evolution card](data_type_design.md),
and a field calculation the [calculation card](calculations.md).

## Order of work

1. Choose the types (`types.*`, `types-only.*`, `schema.*`): App-owned types
   exported from the App root, foreign types by their owner's FQN, numeric
   widths and identity scalars.
2. Declare each attribute where it may stand (`attr.*`, `annot.*`), with its
   semantic exclusions and value grammar (`numeric.*`, `format.*`,
   `bitflags.*`, `mime.*`). Chart attributes (`chart.*`, `annot.chart*`) are
   on the [chart card](charts.md) and the [3D chart card](charts_3d.md).
3. Shape choices and alternatives (`union.*`, `key.*`, `enumsource.*`,
   `protocol-variant.*`, `baseobjects.*`, `data-schema.*`).
4. Decide initial values (`init.*`, `defaults.*`, `user.*`, `fixed-user.*`).
5. Write paths and copies (`paths.*`, `copy.*`); the shared copy rules apply
   to ordinary and calculation copies alike. Display Template selectors
   (`display.selector.*`) are on the [Display Template card](display_template.md).
6. Declare draft-recovery metadata (`cache.*`), and keep runtime values and
   localized documents in step (`validation.*`, `docs.*`). Endpoint and
   protocol declarations are on the [Endpoint protocol card](endpoint_protocol.md).
   Then compile with Studio and fix every reported rule.

<!-- generated:codebot-rules task=data-model -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Libertas source annotation reference

Owner: [annotations.md#libertas-source-annotation-reference](../reference/annotations.md#libertas-source-annotation-reference)

- `annot.canonical-spellings-only`: Use only the canonical spellings shown here. Compiler acceptance alone does not establish supported schema syntax.
- `annot.placement-is-authoring` (judgment): The placement column below states the public authoring placement; it is not permission to move an annotation merely because a parser can preserve it for a later Studio error.
- `annot.documentation-invariants`: Every annotated declaration and nested schema-visible node must also follow the [schema documentation invariants](../reference/documentation.md), including the user-friendly label/description, member, variant, and List-item rules.
- `annot.comments-domain-meaning` (judgment): Source comments supply domain meaning without duplicating those generated notes. Rows marked "compiled into the Display Template" are presentation attributes: Studio validates them, compiles them into the package's [Display Template](../reference/presentation.md#display-template) as its lowest-specificity rules, and omits them from the final schema. A `display.json` rule can change their effect per device without editing source.
- `annot.catalog-scope` (judgment): This catalog exhausts schema-parser annotations, not unrelated SDK runtime functions or serialization derives such as `LibertasAvroEncode`, `LibertasAvroDecode`, `VariantIndex`, or `variant_index!`; those remain covered by the public SDK and the language guides.

### Package, discovery, and parser helpers

Owner: [annotations.md#package-discovery-and-parser-helpers](../reference/annotations.md#package-discovery-and-parser-helpers)

- `annot.export.helper-only`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | `@LibertasExport` | `#[libertas_export]` on a public function; `#[derive(LibertasExport)]` on a schema struct/enum | Parser/compiler helper | Enables the language's public App wrapper or helper attributes. It emits no portable `Export` attribute and does not replace the native public/export requirement. | [TypeScript guide](../reference/languages/typescript.md#supported-jsdoc-decorators), [Rust guide](../reference/languages/rust.md#schema-attributes-and-field-presentation) |

- `annot.export.union-content`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | Not applicable | Exact Union rustdoc line `/// #[libertas_export(content = "payload")]` | Rust Union declaration | Emits parser-owned `UnionContentName` representation metadata. Do not invent `tag` or `untagged` forms. | [Rust guide](../reference/languages/rust.md#schema-attributes-and-field-presentation) |

- `annot.const-enum.file`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | File-leading `/** @LibertasConstEnum */` | Not applicable | Dedicated TypeScript file containing one documented exported numeric constant per statement | Treats the constants as members of the numeric Enumeration exposed by a documented exported `typeof` namespace-import alias in the App entry module. | [TypeScript enumerations](../reference/languages/typescript.md#enumerations) |

### Defaults, constraints, and ordinary presentation

Owner: [annotations.md#defaults-constraints-and-ordinary-presentation](../reference/annotations.md#defaults-constraints-and-ordinary-presentation)

- `annot.enum.use-native-enum`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | `@LibertasEnum(A, B, C)` | `#[libertas_enum(A, B, C)]` | Applicable String alias/field or Enumeration declaration supported by that language | Defines a catalog using positional members. The parsers currently cannot attach source documentation to those fabricated members, so codebots must use a documented native numeric enum instead and must not generate this otherwise public compatibility form. TypeScript `values=[...]` is not canonical syntax. | [TypeScript enumerations](../reference/languages/typescript.md#enumerations), [Rust attributes](../reference/languages/rust.md#schema-attributes-and-field-presentation) |

- `annot.enumsource.unmatched-unanswered`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | `@LibertasEnumSource("$.choices")` | `#[libertas_enum_source("$.choices")]` | Unsigned integer field/type; the key's String or integer type when the target List declares a key; or a List of that type | Emits `EnumSource`; the host stores the zero-based index of an item in the resolved same-tree List or, when that List's member declares a `Key` field, the chosen member's key value. A stored value matching no current member is unanswered and never remapped. An unanswered non-Nullable host whose source offers exactly one selectable member takes that member. A List host stores several distinct chosen indices or keys. | [Common field metadata](../reference/presentation.md#common-field-metadata), [EnumSource keys](../reference/schema.md#enumsource-keys), [EnumSource List hosts](../reference/schema.md#enumsource-list-hosts) |

- `annot.combinations-not-implied` (judgment): Attribute combinations and inherited effective metadata are validated by Studio. Do not infer that two annotations can coexist merely because they have separate rows in this catalog.

### External types, Matter descriptors, and Union metadata

Owner: [annotations.md#external-types-matter-descriptors-and-union-metadata](../reference/annotations.md#external-types-matter-descriptors-and-union-metadata)

- `annot.device-type.generated`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | `@LibertasDeviceType("descriptor")` | `#[libertas_device_type("descriptor")]` | Effective `LibertasDevice` field | Emits the Studio-generated physical Matter `DeviceType` descriptor. Do not invent the encoded descriptor manually. | [Matter](../reference/matter/descriptors.md#canonical-generation-and-decoding) |

- `annot.virtual-device-type.generated`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | `@LibertasVirtualDeviceType("descriptor")` | `#[libertas_virtual_device_type("descriptor")]` | Effective `LibertasVirtualDevice` field | Emits the Studio-generated `VirtualDevice` descriptor. Do not invent it manually. | [Matter](../reference/matter/descriptors.md#canonical-generation-and-decoding) |

- `annot.discriminator.typescript-only`:

  | TypeScript canonical source | Rust canonical source | Public placement | Result and meaning | Details |
  | --- | --- | --- | --- | --- |
  | `@LibertasDiscriminator("kind")` | Not applicable | TypeScript discriminated Union when automatic detection is ambiguous | Emits parser-owned `UnionTagName`. Normal TypeScript literal/enum discriminators are detected automatically; Rust uses enum declaration order and native representation. | [Protocol roles](../reference/protocols.md#protocol-roles) |

### Inferred and generated metadata: never invent source annotations

Owner: [annotations.md#inferred-and-generated-metadata-never-invent-source-annotations](../reference/annotations.md#inferred-and-generated-metadata-never-invent-source-annotations)

- `annot.generated.no-source-spelling`: The final schema contains attributes and fields that are inferred from native source or generated from one of the public annotations above. Their presence in schema JSON does not create another public macro or decorator.
- `annot.no-std.native`:

  | Generated or parser-owned result | Source of the result |
  | --- | --- |
  | Rust `no_std=true` | Native crate attribute `#![no_std]`, not a Libertas helper macro. Use it only when the App and all dependencies support that build model. |

- `annot.generated.no-derived-spellings`: Never derive a new `@Libertas...` or `#[libertas_...]` spelling from a schema JSON property.

### Not public or not usable

Owner: [annotations.md#not-public-or-not-usable](../reference/annotations.md#not-public-or-not-usable)

- `annot.removed.never-emit`: Codebots must not emit any spelling in this section.

### Unsupported authoring forms

Owner: [annotations.md#unsupported-authoring-forms](../reference/annotations.md#unsupported-authoring-forms)

- `annot.named-argument-variations`: Named-argument variations do not create additional supported spellings.

### Removed Rust SDK helpers

Owner: [annotations.md#removed-rust-sdk-helpers](../reference/annotations.md#removed-rust-sdk-helpers)

- `annot.rust-sdk-helpers.removed`: The SDK helper allowlists no longer admit `libertas_string`, `libertas_array`, or `libertas_cacheable`, because the owning schema parser does not support those source forms. Use `libertas_size` for size bounds and the documented CacheIdentity behavior where needed. `libertas_discriminator` is also not a Rust schema annotation; Rust uses native enum representation.

### MIME-aware Bytes annotation

Owner: [annotations.md#mime-aware-bytes-annotation](../reference/annotations.md#mime-aware-bytes-annotation)

- `annot.mime-types.semantic-choice` (judgment): Keep unannotated Bytes fields for raw binary protocols; `MimeTypes` is a semantic choice for media values, not a replacement for ordinary Bytes.

### CacheIdentity

Owner: [annotations.md#cacheidentity](../reference/annotations.md#cacheidentity)

- `cache.clients-implement`: Use clients that implement this attribute; a client that does not provides no draft recovery.

### Eligibility and field placement

Owner: [annotations.md#eligibility-and-field-placement](../reference/annotations.md#eligibility-and-field-placement)

- `cache.identity.tuple`: Multiple marked fields form one identity tuple. All must match; they are not alternative identities. The client does not infer identity from field names, Hidden, or CopyFrom.
- `cache.identity.defined-immutable`: Identity values must be defined after initialization; valid null is allowed and differs from undefined. They remain immutable for the editor lifetime, including against user edits and calculation writes. Restoring editable data preserves the freshly initialized identity values.

### Identity, revisions, and application responsibility

Owner: [annotations.md#identity-revisions-and-application-responsibility](../reference/annotations.md#identity-revisions-and-application-responsibility)

- `cache.identity.distinguishing-context` (judgment): The application must supply enough distinguishing context to prevent one workflow's draft from matching another workflow. A workflow identifier is a typical choice.
- `cache.identity.revision-recommendation` (judgment): **Recommendation:** include an application-maintained data/workflow revision when changes to saved data invalidate earlier drafts. For example, a historical page edit may invalidate later pages; changing their identity revision prevents restoring obsolete local drafts. The client does not infer these dependencies.

### Required design principle

Owner: [app-contract.md#required-design-principle](../reference/app-contract.md#required-design-principle)

- `app.design.natural-data-model` (judgment): **Model data in its natural form.** Declare each stored type as what the data is: its real fields, one owner per fact, and a Union where a value is one of several shapes. Never reshape stored data for a screen, a workflow step, or a question order. Human-facing types, the Request and Response payloads a person edits and views, are views of that data: they may wrap it as the person's path needs, for example as a wrapper member within a Union, while the stored model stays as it is.
- `app.design.no-contradiction` (judgment): **The two never conflict.** The goal is human first, and the natural data model serves it: the platform presents the model in its natural form along the person's shortest path, so neither is traded for the other. Where they seem to conflict, the platform is missing a presentation mechanism; raise that gap instead of bending the data or lengthening the person's path. Presentation and interaction adapt to the model through the platform: schema attributes such as `AskOnAdd`, `ReadOnly`, and `Hidden`, and Display Template rules such as `lock`, `flatten`, `promote`, and layouts. When the platform has no way to present a natural model, that is a platform gap to raise, not a reason to bend the data.

### Default or no Default

Owner: [app-contract.md#default-or-no-default](../reference/app-contract.md#default-or-no-default)

- `app.default.decided`: Every field a person answers gets an explicit, reasoned decision, never an accident of its source type or a copied pattern. **No `Default`** when the person must answer: the field stays [undefined](../reference/schema.md#undefined-and-its-sources) and blocks Submit until answered, on a client-built request ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). **A `Default`** only when that value is the answer the person would give without reading, such as a usual value or `$null` where skipping lawfully means null. Never default an answer that has consequences, such as eligibility, an election, consent or a fact the App's rules test: a Default is never the person's confirmation. Decide again whenever the field's question, type or page changes. The rows below apply this decision to each kind of field.

### Standard properties

Owner: [documentation.md#standard-properties](../reference/documentation.md#standard-properties)

- `doc.boolean.semantics`: **A Boolean's `true` and `false` can carry any two meanings the field needs**, such as Accept and Decline: Yes and No are only the fallback captions, `[TrueText]` and `[FalseText]` state any other meaning, and the App's rules read the value under that meaning.
- `doc.boolean.unanswered`: Without a schema Default, a required Boolean starts unanswered. Both choices are unselected; the editor holds an undefined value and refuses submission until one is chosen. It never encodes undefined as false or null.
- `doc.nullable.null-text-rationale` (judgment): This is deliberate: AI CodeBots especially assume null means None, which has caused subtle design flaws; writing `[NullText]` on every question a person answers forces the author to decide what null means there.
- `doc.nullable.null-text-meaning`: **Null can carry any meaning the field needs**: it means None only until `[NullText]` states another meaning, which every Nullable, editable or not, must then declare; a non-editable Nullable shows it wherever its null is displayed.

### Unanswered fields

Owner: [presentation.md#unanswered-fields](../reference/presentation.md#unanswered-fields)

- `field.unanswered`: **Unanswered fields.** A field the person answers that no `Default`, `Fixed` or copy initializes starts [undefined](../reference/schema.md#ordinary-initialization) (unanswered), and a non-Nullable field never holds null. When an editor opens, an undefined field shows only its neutral empty state: its placeholder, such as an unselected choice or "Choose date…", never an error look; an undefined Nullable's text uses the secondary text color. When Next or Submit is blocked, the client marks the first field that blocks it: below that field, an error icon and the missing-answer or validation message, in the platform's error color role and announced politely to assistive technology. It opens any fold or summary page holding the field and scrolls it into view; it does not move focus. The mark clears once that field passes, and it does not move to another field until the next blocked Next or Submit. An undefined field is reported as a missing answer, never as a null value.

### Editing and viewing

Owner: [presentation.md#editing-and-viewing](../reference/presentation.md#editing-and-viewing)

- `tpl.condition.recheck-combined` (judgment): **Re-check a combined condition when it or its data changes.** The client evaluates a condition on the value being displayed or edited, including rows added or removed before any save, so nothing on the server keeps it correct. A condition that combines several fields names each one: a page-is-empty test, for example, counts every List of the page. When you change such a condition, or add, remove, rename, or retype a field or List that it reads or should read, read the whole condition again against the current types, update it, and view each outcome in the Display Template preview. Studio checks that every path resolves and that the expression is Boolean; it cannot tell that a newly added field belongs in the condition.

### Strict questions and data pages

Owner: [protocols.md#strict-questions-and-data-pages](../reference/protocols.md#strict-questions-and-data-pages)

- `workflow.strict-question.required`: **A must-answer question** is a required field with no `Default` on a request the client constructs (an offered action, link or landing request, or a `WorkflowRequest` editor): it starts unanswered, shows no error, and blocks submission. Never model it as a `Nullable` whose `NullText` means "not answered yet", and never send it in an EditRequest.

### Libertas data model reference

Owner: [schema.md#libertas-data-model-reference](../reference/schema.md#libertas-data-model-reference)

- `schema.generated-by-studio`: Generate schemas through Studio rather than editing them.

### Libertas Schema Path

Owner: [schema.md#libertas-schema-path](../reference/schema.md#libertas-schema-path)

- `paths.native-names`: Fields use case-sensitive Native Names, not localized Display Names. Named type references and inheritance are resolved without introducing extra value-tree levels.
- `paths.context.operation-source`: An operation-specific copying source does not implicitly replace the interaction's reference context.

### Shared CopyFrom and CopyTo rules

Owner: [schema.md#shared-copyfrom-and-copyto-rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)

- `copy.alternatives.source-order`: For each annotated receiving node, CopyFrom visits reference objects in context order. Within each object, it evaluates the comma-delimited alternative paths in declaration order. It skips absent/undefined values and stops at the first available value, including valid null. Source order takes priority over path order: exhaust the paths in the first object before trying any path in the next.
- `copy.selection.single-source`: Selection is independent for each annotated receiving node. A selected composite value is projected from that one source; do not merge reference objects or fill its absent members by searching another object.
- `copy.unavailable.no-copy`: With no reference objects or no available path, CopyFrom performs no copy. Ordinary request construction then uses the normal initialization rules below.
- `copy.paths.explicit-qualifiers`: An explicit path may address a node at a different nesting depth or with a different Native Name. Type or Protocol Variant qualifiers identify the permitted schema context; a compatible type alone does not identify the intended value. Within a copied Table, corresponding members retain exact, case-sensitive Native Name matching. Selecting a differently located Table does not implicitly rename its members; such a difference requires an explicit mapping of the relevant node where permitted, or a payload with corresponding member names.
- `copy.projection.native-names`: Both CopyFrom and CopyTo use the same reduced-field projection rules. Match record fields by exact, case-sensitive Native Name and recurse through records and array elements. Source-only fields absent from the destination type MUST be ignored without a copy error; they are not assignments and do not participate in destination compatibility, overlap, or protection checks. Matching fields retain the applicable type, nullability, semantic scalar, and Union identity requirements. This does not make a required destination field optional or permit an incompatible matching field.
- `copy.projection.null-and-validity`: A valid null assigns null to a nullable destination; it does not mean unchanged. A received CopyTo payload must remain valid under its own schema. CopyFrom may project an unfinished editor under the destination's completeness rules.
- `copy.naming.shared-roles` (judgment): **Design rationale: structural matching by field name.** Because copied records match members by Native Name and ignore source-only fields, a Rust or TypeScript App composes values the way a scripting language does: any record with a member of the expected name and a compatible type can supply or receive it, whatever its declaring type. A calculation input can list `@.Business.assets,@.Farm.assets,@.Rental.assets`, and each branch's rows project into the input's element type because every branch names that member `assets`. A field name is therefore part of an App's interface, not only a label. Name fields that play the same role across types deliberately and identically, and fields with different meanings differently; one calculation, projection or alternative list then serves every type that shares the role, with no adapter types.
- `copy.initialization.unaffected-nodes`: CopyFrom and CopyTo override initialization only at the nodes they actually assign. On a newly constructed destination, every unaffected node retains its ordinary Fixed, Default, localized default, nullable, and type initialization. This includes destination-only fields within a copied Table or array element.

### Ordinary initialization

Owner: [schema.md#ordinary-initialization](../reference/schema.md#ordinary-initialization)

- `init.ordinary`: Ordinary initialization may yield a value, valid null, or undefined; it need not make the constructed request complete. Type initialization, which applies to a node with no Fixed, Default, or localized default, is the same on every client. An editable Nullable node is undefined, unanswered like a Union with no branch selected, because its editor asks a two-answer question ([Nullable choices](../reference/presentation.md#nullable-choices)); the reserved Default [`$null`](../reference/schema.md#reserved-default-values) starts it at null, a complete answer, and any other Default starts it at that value. A ReadOnly, Hidden, or Fixed Nullable node, which the person cannot answer, is null, and so is a Nullable node of a Response or SubscriptionData payload. A Table is its fields, each initialized by these rules; a List is empty; a Union has no selected branch. After a Union variant is selected, its payload initializes as its own schema occurrence under [attribute resolution](../reference/schema.md#attribute-resolution). A non-Nullable Table payload constructs its fields even when the outer Union is Nullable. If the payload itself is Nullable, choosing the outer variant leaves that separate null/value question to its own Fixed, Default, or ordinary initialization. Every other required node is undefined, which is unanswered: a String, a number, a Boolean, an Enumeration, Bytes, a DateOnly, a TimeOnly, and a DateTime alike. A client never supplies an empty string, zero, a range bound, or the current time in its place, so a present value always means an answered one. Merely supplying a reference context does not change an unaffected node's initialization.
- `copy.initialization.existing-destination`: On an existing destination, unaffected nodes keep their current values; copying another field does not reset them.

### Undefined and its sources

Owner: [schema.md#undefined-and-its-sources](../reference/schema.md#undefined-and-its-sources)

- `init.undefined-relationships` (judgment): **A node is undefined until it is initialized** by `Fixed`, a `Default`, a copy, a supplied value, or [type initialization](../reference/schema.md#ordinary-initialization) (an empty List, a null non-editable Nullable); otherwise it stays undefined until answered, which is how a must-answer question is asked ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). Undefined cannot be encoded, so no protocol message carries it; only an editor holds it, showing its neutral empty state ([unanswered fields](../reference/presentation.md#unanswered-fields)) and blocking submission while required.
- `init.undefined.construction` (judgment): **Construction:** a `Default`, `$null` included, answers the node; `Fixed` answers and locks it; `CopyFrom` answers only the nodes it assigns.
- `init.undefined.copy-source` (judgment): **Copy source:** `CopyFrom` skips an undefined or absent value for the next alternative and stops at a valid null; with nothing available it copies nothing ([copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)). An undefined required calculation input blocks its calculation; a validation rule that reads undefined is aborted and reports a missing answer.
- `init.undefined.copy-destination` (judgment): **Copy destination:** a successful `CopyTo` answers the destination or replaces its answer, the later assignment winning; it never makes a node undefined, assigns null only to a Nullable, and never overwrites `Fixed` or `CacheIdentity` data ([applying results](../reference/protocols.md#applying-calculation-results)).
- `init.undefined.constrained-by` (judgment): **[`ConstrainedBy`](../reference/protocols.md#backend-constrained-choices):** an undefined non-nullable field with a one-entry allowed set takes that entry, as a Union `Default` selects its variant; an empty set leaves a Nullable field only null; a constructed value outside the set stays a field error. When a result changes a [calculated set](../reference/protocols.md#calculated-allowed-sets), a selection outside it resets to undefined, never null; its `Default` reapplies only if the new set contains it, and the singleton rule follows.

### Shared CopyFrom and CopyTo rules

Owner: [schema.md#shared-copyfrom-and-copyto-rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)

- `copy.union.initial-branch`: CopyFrom on a Union selector may supply its initial branch. Browsing or explicitly choosing another branch does not reapply that selector's CopyFrom; the chosen payload and its descendants retain their own ordinary construction, including their CopyFrom mappings and the current reference context.
- `copy.fixed.member-precedence`: Within a copied Table or array, a copied member takes precedence over a Fixed initializer, but still has to satisfy the Fixed constraint and every other applicable constraint. Copying does not waive destination protections such as the calculation CopyTo prohibition on overwriting Fixed fields.
- `copy.apply.construct-or-merge`: The operation defines how the projected values are applied. CopyFrom constructs its destination; CopyTo merges into an existing destination. Source selection, destination uniqueness, array correspondence, and operation ownership retain their respective contracts.

### Context and syntax

Owner: [schema.md#context-and-syntax](../reference/schema.md#context-and-syntax)

- `paths.context.prevrequest`:

  | Usage | Accepted form | Source or destination context |
  | --- | --- | --- |
  | CopyFrom when constructing a PrevRequest action | The same variant-rooted or root-relative copy syntax | The construction context contains the current edited request as its reference object. |

- `paths.context.loader`:

  | Usage | Accepted form | Source or destination context |
  | --- | --- | --- |
  | Display Template leading-page loader CopyFrom | The same variant-rooted or root-relative copy syntax | Initial load uses the current complete editing Request as the construction reference object, without changing its server-supplied interaction reference context. Subsequent refresh uses the current leading page under the ordinary rules. See [master-detail panes](../reference/presentation.md#master-detail-panes). |

### Typed ancestors and bound occurrences

Owner: [schema.md#typed-ancestors-and-bound-occurrences](../reference/schema.md#typed-ancestors-and-bound-occurrences)

- `paths.binding.invoking-node`: A [link](../reference/protocols.md#links) binds self-or-ancestor from its invoking node, the tapped node or the host field, so `^<TopicRow>` on a link declared by `TopicRow` selects that row. EnumSource and request-field ConstrainedBy bind only occurrences on the host field's ancestry, starting at its parent and including the variant root; the variant root remains eligible when it is the host, which makes primitive variant roots addressable. A strict path pins the schema occurrence; relaxed matching selects the nearest enclosing occurrence of its named type. The selector may omit a suffix to select the bound value itself; otherwise suffixes use ordinary fields and literal zero-based array indices. `$` still selects the complete current request where the context table admits it; calculation paths reject it and write the request root as `^<Variant>`. Ordinary CopyFrom uses each reference object in context order as a candidate variant root.

### Current-data references in validation expressions

Owner: [schema.md#current-data-references-in-validation-expressions](../reference/schema.md#current-data-references-in-validation-expressions)

- `paths.validation.missing-null`: If a path exists, it resolves to its value; otherwise, it resolves to null. Resolve segments over actual values in source order:

### Locator paths

Owner: [schema.md#locator-paths](../reference/schema.md#locator-paths)

- `paths.locator.key-step`: At a List whose member declares a [key](../reference/schema.md#enumsource-keys), a segment may be a **key step**: `=` followed by a literal of the key's type, a JSON string literal (double quotes, with JSON escapes `\"`, `\\` and `\uXXXX`) for a `String` key or an integer literal for an integer key. It selects the member whose key equals that literal exactly. For example, `[incomes/="W2-58"/box1]` and `[accounts/=42/balance]`. Whitespace inside the quotes is part of the key; whitespace around the step is ignored, as for every segment. A position segment (`[incomes/2/box1]`) stays valid at any List, keyed or not.

### Availability, null, and interaction ownership

Owner: [schema.md#availability-null-and-interaction-ownership](../reference/schema.md#availability-null-and-interaction-ownership)

- `paths.context.attribute-defines-syntax` (judgment): Reference-context lookup is a shared schema reference capability. Any schema attribute whose semantics support reading a referenced value may use this context when its contract permits it. Each attribute defines its accepted path or expression syntax, expected value type, and handling of an unavailable reference; this capability does not make every attribute accept a path. The [interaction reference-context rule](../reference/protocols.md#interaction-reference-context) owns the server-supplied context and its replacement.
- `paths.context.no-implicit-fetch`: A reference lookup never performs an implicit server fetch or substitutes task parameters, sibling request values, task-upgrade data, pooled endpoint data, or another interaction's reference context.
- `paths.validation.curly-lookup`: Curly validation references visit reference objects in the same context order. Within each object, they inspect comma-delimited paths in declaration order and return the first non-null value. Missing, undefined, and null alternatives are skipped, continuing into later objects when necessary; no match yields null. They are pure reads. Unlike CopyFrom, a valid null does not stop this lookup. With no reference context, including first-time composition, every such read yields null; authors must handle that absence. CopyFrom follows the distinct [copy-selection and initialization rules](../reference/schema.md#shared-copyfrom-and-copyto-rules). Studio checks their syntax but treats their result type as schema-unknown and nullable because server-supplied reference-object schemas may differ.
- `paths.validation.rule-roots`: Editable-request rules use the interaction's current reference context through curly references. Function rules see the argument Table; a field/type rule is rooted at its owner. A required field can be temporarily unanswered locally, but a valid path expression does not make it optional or permit an incomplete transmitted request. EditRequest construction follows the [interaction reference-context rule](../reference/protocols.md#interaction-reference-context).
- `paths.validity-not-permission` (judgment): A syntactically valid path does not grant permission, create a missing value, prove a runtime index exists, or transfer ownership of a pending reply. The containing attribute defines assignment, validation, and lifecycle behavior. ConstrainedBy, EnumSource, BaseObjects, and calculation rules retain those operation-specific requirements in their owning sections.

### 1. Artifact model

Owner: [schema.md#1-artifact-model](../reference/schema.md#1-artifact-model)

- `schema.artifact.native-name-stable`: The separation is deliberate. `NativeName` remains a stable programmatic name in the schema, while localized `DisplayName`, field `Name`, `Description`, and package `StringResources` values live in documents. Translating presentation text therefore does not break type references or other schema links.

### 2.1 Top-level collections

Owner: [schema.md#21-top-level-collections](../reference/schema.md#21-top-level-collections)

- `schema.types.owned-only`:

  | Field | Meaning |
  | --- | --- |
  | `Types` | Named types whose declarations are owned by this App. A type declared by another package is never copied here, even when it is imported, used, or re-exported locally. |

- `schema.functions.exported`:

  | Field | Meaning |
  | --- | --- |
  | `Functions` | Exported callable functions that are not explicitly ignored. Functions are identified by `NativeName`; parameters are positional and retain source order. |

- `schema.functions.no-return-schema` (judgment): The generated function model contains parameters but no return-schema field. Request/response flows are modeled through endpoint protocol unions and persistent function data through a function data-schema union.

### 2.4 Runtime types

Owner: [schema.md#24-runtime-types](../reference/schema.md#24-runtime-types)

- `types.identity.signed-ids`: System identity scalars (`User`, `Device`, `LanDevice`, `VirtualDevice`, and `Endpoint`) use signed 64-bit IDs: Rust `i64` and TypeScript `bigint`. `User` also represents groups. Durable allocated IDs are positive; negative IDs are runtime dynamic identities and are not durable references. Zero and other sentinels have only their operation-specific meanings. The semantic type already defines the width and signedness; no `NumberType` attribute is needed.
- `types.number.integer-step-default`: Integer nodes default to `NumberStep=1` in parser paths that construct integer constraints.
- `types.number.cross-language-width`: A cross-language contract must use a width supported by both representations.

### 2.5 Structural order and IDs

Owner: [schema.md#25-structural-order-and-ids](../reference/schema.md#25-structural-order-and-ids)

- `schema.order.source-order`: Table fields, function parameters, list elements, and union variants preserve parser/source order. This order is part of the wire contract.

### 3.2 Foreign wire types

Owner: [schema.md#32-foreign-wire-types](../reference/schema.md#32-foreign-wire-types)

- `types.foreign.fqn-grammar`: Both `developer` and `package` use lowercase snake case, separated by exactly one hyphen. `TypeName` is a source-language identifier-like simple name.
- `types.foreign.fqn-external`: For every externally owned reference, this complete FQN is stored directly in `Type`, `Heritages`, `Bitflags`, `ForeignTypeSchema`, `ProtocolSchema`, or `FunctionDataSchema`, depending on the reference site. Every such FQN contributes its package to the direct dependency set used for pre-deployment Hub exploration and the finalized `AppDependencies` closure. An FQN always means external ownership, so its package component must differ from the package that owns the referencing schema; local references use the simple type name.

### Attribute resolution

Owner: [schema.md#attribute-resolution](../reference/schema.md#attribute-resolution)

- `attr.resolution.nearest`: Ordinary attribute lookup tries the local declaration first, corresponding same-name inherited Table fields in declared heritage order second, and the referenced named type last. Resolve inherited fields and named references by this same rule. Attribute-specific contracts govern cumulative [ValidationRules](../reference/validation-rules.md) and [Dynamic](../reference/schema.md#dynamic-schema-properties), direct [Link](../reference/protocols.md#links) declarations, and the [template cascade](../reference/presentation.md#display-template-cascade) and [source-default declaration lookup](../reference/presentation.md#compiled-template).
- `attr.resolution.complete-value`: The first declaration found supplies the whole attribute value. A nearer value replaces a farther value; their strings, lists, or properties are never merged. Where valid for that attribute, an explicit `false`, zero, or empty value still wins. A field's reserved `Default=$null` also wins over a referenced type's value Default; interpreting the winning declaration as null does not resume lookup. Different attribute names resolve independently. With no declaration, the attribute is absent.
- `attr.resolution.host`: The lookup follows schema declaration relationships, not the ancestry of runtime values. An enclosing Table, List, or Union does not automatically donate its attributes to a child. Attribute-specific scope still applies: for example, Hidden suppresses a presentation subtree, and ReadOnly on a List controls its membership under the [collection rule](../reference/presentation.md#common-field-metadata).
- `attr.resolution.union-host`: A Union field and a selected variant's payload are distinct attribute hosts. `Nullable` on the Union permits the outer null answer. It does not declare the selected payload itself Nullable. Construct that payload using the variant field's own attribute resolution, including its referenced type; see [ordinary initialization](../reference/schema.md#ordinary-initialization). Likewise, a Union's Default selects a variant; it is not a Default value for that variant's payload. For a scalar payload with a fixed encoded value, `Fixed` protects that value; it does not lock the outer Union's branch selector.

### 4.3 Value, constraint, and presentation attributes

Owner: [schema.md#43-value-constraint-and-presentation-attributes](../reference/schema.md#43-value-constraint-and-presentation-attributes)

- `attr.nullable`:

  | Attribute | Meaning |
  | --- | --- |
  | `Nullable` | Adds one answer, null, to the type's values. **Null can carry any meaning the field needs**, and its meaning is never assumed: it means its fallback None unless [`NullText`](../reference/documentation.md#standard-properties) states another, such as absence, "unknown" or "Does not apply". It never stands for "not answered yet", which is undefined. Optional source types normalize to this marker. |

- `attr.ask-on-add`:

  | Attribute | Meaning |
  | --- | --- |
  | `AskOnAdd` | Marker value `true`: the person's **Add** asks this List-member value first, and adds no member without it. Only the person's Add; other writers are unaffected. See [add-time values](../reference/presentation.md#add-time-values). |

- `attr.read-only`:

  | Attribute | Meaning |
  | --- | --- |
  | `ReadOnly` | Value is displayed but not user-editable. Initialization, CopyFrom, EditRequest, calculation or link-return CopyTo and draft restoration still write it; only `Fixed` protects a value. See the [ReadOnly rule](../reference/presentation.md#common-field-metadata). |

- `attr.hidden`:

  | Attribute | Meaning |
  | --- | --- |
  | `Hidden` | Value is hidden from ordinary editors. On a chart carrier table, composition, or chart-bearing Union variant wrapper, it also marks direct non-chart fields as preserved transaction context that renderers exclude from chart projection. |

- `attr.time-interval`:

  | Attribute | Meaning |
  | --- | --- |
  | `TimeInterval` | Displays a numeric value as a duration. It does not create a distinct wire type. |

- `format.presentation-only`: `Format` is valid only on the four effective hosts above and may be inherited through a named type. On an ordinary `Number` it follows the [exclusions of numeric semantic attributes](../reference/schema.md#numeric-semantic-exclusions); on a `DateOnly` Number it uses date-pattern grammar. It may coexist with `PhysicalUnit`, in which case it formats only the numeric portion and the unit remains separate localized semantic metadata. Money uses the [exact currency editor](../reference/presentation.md#money). Other editors retain native number/date/time controls and may show a formatted preview. Renderers apply `Format` only at presentation boundaries—labels, summaries, table cells, tooltips, chart ticks, and accessibility text. Parsing user input, Avro payloads, defaults, bounds, sorting, grouping, chart geometry, cache keys, and protocol compatibility always use the raw scalar.
- `bitflags.runtime.allowed-mask`: A field's `Default`, `Fixed`, task configuration value, outgoing runtime value, and received runtime value may contain any combination of declared nonzero bits and no undeclared bit.
- `bitflags.catalog-materialization-and-ui.authoring`: Declare nonzero bits as independent choices; the zero member labels the empty mask, which a required field's editor offers as one [exclusive choice](../reference/presentation.md#bitflags) that clears every flag.
- `numeric.format-coexists`: `Format` may stand with `DateOnly`, `Money`, or `PhysicalUnit`: it then formats the date, the amount, or the numeric portion.

### Reserved Default values and constant references

Owner: [schema.md#reserved-default-values-and-constant-references](../reference/schema.md#reserved-default-values-and-constant-references)

- `defaults.pattern-dollar-literal`: `Pattern` accepts a whole source constant argument through its [source-language syntax](../reference/validation-rules.md#132-full-string-patterns). Inside a quoted pattern, `$` remains regex text; decoded patterns are never scanned for constant references. The emitted attribute contains only the decoded pattern.

### User and group defaults

Owner: [schema.md#user-and-group-defaults](../reference/schema.md#user-and-group-defaults)

- `user.no-implicit-current-user`: `User` stores a user or group ID. Neither a `User` nor a `List<User>` implicitly selects the current user, including when only one picker candidate exists. Without `Default`, ordinary initialization applies, including undefined. Copying retains the [shared initialization precedence](../reference/schema.md#shared-copyfrom-and-copyto-rules).

### Fixed current-user security constraint

Owner: [schema.md#fixed-current-user-security-constraint](../reference/schema.md#fixed-current-user-security-constraint)

- `fixed-user.first-field` (judgment): **Recommended declaration order:** declare one `Fixed("current-user")` field as the first payload field of each Request or SubscriptionRequest variant, and reuse that verified identity in App code. This keeps the identity near the start of the payload after any protocol variant selectors. First position is a recommendation, not a schema requirement; nested protocol variants and identity fields within nested request values remain supported, excluding Lists. If additional fields carry this constraint, the Hub MUST validate every applicable occurrence; validating the first never exempts later fields. Declaration order enables efficient identity checks but does not guarantee that a Hub implementation decodes only one field.
- `fixed-user.hub-verifies`: Before invoking App code, the Hub MUST compare the submitted value with the user identity established by authentication for that invocation. It MUST reject a mismatch, null, an unresolved identity, or malformed input. It MUST NOT repair a submitted mismatch by replacing the value. Client autofill, ReadOnly, Hidden, labels, and a claimed user ID in the payload are never evidence of identity.
- `user.notification-recipients`: For mandatory notification recipients, combine `Default` with static `ReadOnly`: on a scalar User it protects that ID; on a List it protects membership and order, so mark the User item type ReadOnly as well to protect each recipient ID. Follow the [ReadOnly placement rules](../reference/presentation.md#common-field-metadata). An unresolved explicit default blocks submission even when ReadOnly; never silently substitute recipients.

### MIME-aware Bytes

Owner: [schema.md#mime-aware-bytes](../reference/schema.md#mime-aware-bytes)

- `mime.validate-content`: An App must validate actual content when the declared MIME label matters to its processing or security.
- `mime.empty-undefined`: Zero-length `Bytes` means **media undefined**. It is a present and valid wire value, including on a non-`Nullable` field. A server may send it; clients show no selected media. It is distinct from nullable null and from a valid `LMB1` value with zero content bytes. For [ValidationRules](../reference/validation-rules.md), zero-length MIME-aware Bytes evaluates as null; ordinary Bytes retain their existing valuation. A mandatory media field needs an explicit rule such as `[] != null : "IMAGE_REQUIRED";`. `SizeMin` and `SizeMax` run only for defined media and measure content bytes, excluding the LMB1 header. Thus `SizeMin=1` rejects a defined empty file but does not require a file to be selected. Transport limits count the entire sequence.
- `mime.compatibility`: `MimeTypes` changes the semantic Bytes type without changing the Avro primitive. Raw and MIME-aware Bytes are not implicitly compatible. A producer MIME filter is compatible with a recipient only if every exact type it admits is accepted by the recipient; dynamic values are checked against the destination at runtime. Copies preserve the complete envelope; clients do not guess or transcode media from a filename or destination filter. Adding `MimeTypes` to an existing raw field requires a new immutable App version. Legacy mobile-Hub paths retain their raw Bytes behavior.

### 5.1 Tables and inheritance

Owner: [schema.md#51-tables-and-inheritance](../reference/schema.md#51-tables-and-inheritance)

- `types.table.inheritance`: A `Table` is an ordered record. Each child is one field. A derived table can list base types in `Heritages`; inherited children are resolved recursively and added unless the derived table already declares the same `NativeName`.

### 5.2 Lists

Owner: [schema.md#52-lists](../reference/schema.md#52-lists)

- `types.list.element`: A `List` contains one child schema describing every element. The element may be a built-in node, an inline composite, or a named reference. List-level and element-level documentation are distinct even though both originate around the same source declaration.

### 6.3 Function data schemas

Owner: [schema.md#63-function-data-schemas](../reference/schema.md#63-function-data-schemas)

- `data-schema.key-layout`: The key union and its complete reachable layout are ordinary entries in `Types`, including nested unions, ordered fields, discriminator values, numeric representations, byte bounds and nullability. There is no separate index catalog: fields in a key define its identity and ordered scan dimensions. Stored records and concrete index values are runtime data, not schema content. The generated default document includes the key union, its nested types and their field/variant text under the ordinary `Types` document collection. Translations change those labels, never encoded keys.

### 7. Path-based object and choice binding

Owner: [schema.md#7-path-based-object-and-choice-binding](../reference/schema.md#7-path-based-object-and-choice-binding)

- `enumsource.keyed-bounds`: `NumberMin`, `NumberMax`, `NumberStep`, `RangeMin`, and `RangeMax` on a keyed integer host bound the stored key value, never the member's row position. A member whose key is unset (`null`) or an empty `String` is not offered and cannot be selected. `ConstrainedBy` constrains a host's stored value, which for a keyed host is the key: on a keyed scalar EnumSource host its allowed set holds key values of the key's type, `String` or integer, and selects which members may be chosen while every member stays displayable ([keyed allowed sets](../reference/protocols.md#backend-constrained-choices)). It is rejected on an unkeyed (index) EnumSource host and on a List host.

### Enumsource list hosts

Owner: [schema.md#enumsource-list-hosts](../reference/schema.md#enumsource-list-hosts)

- `enumsource.list-host.stale-elements`: Every element must name a selectable member. An element naming none, a stale key included, is unanswered: the host blocks submission with a field error, as a stale scalar host does, and is never remapped or silently dropped.
- `enumsource.list-host.distinct`: Elements are distinct by this contract, so the host needs no `Unique`: a repeated key or index blocks submission with a field error.
- `enumsource.list-host.empty-and-size`: The empty List is a complete answer: none chosen. `SizeMin` and `SizeMax` (`Size(min, max)`) bound the number of chosen keys, so `SizeMin=1` requires at least one.
- `enumsource.list-host.element-bounds`: Numeric bounds and step on the element type bound each stored key or index, as on a scalar host.
- `enumsource.list-host.no-singleton`: The [singleton rule](../reference/protocols.md#backend-constrained-choices) never fills a List host, and a Display Template [`when_single_choice`](../reference/presentation.md#single-choice-rows) rule cannot select one. `ConstrainedBy` does not apply, because the host is not an Enumeration or Union field.

### 8. Types-only and interoperable packages

Owner: [schema.md#8-types-only-and-interoperable-packages](../reference/schema.md#8-types-only-and-interoperable-packages)

- `types.compatible.merged-view.authoring`: Use explicit compatible-package attributes to override the corresponding official attributes by name under the [compatible metadata rules](../reference/schema.md#8-types-only-and-interoperable-packages).

### 9. Static and runtime validation

Owner: [schema.md#9-static-and-runtime-validation](../reference/schema.md#9-static-and-runtime-validation)

- `validation.task-value-phase`: Task-value validation is a separate phase. It applies the generated model to actual values and checks nullability, scoped uniqueness, endpoint base objects, enum membership, numeric type/range/step constraints, string and byte lengths, list size, and table structure. Schema validity does not imply that every runtime value is valid.

### 10. Localized package documents

Owner: [schema.md#10-localized-package-documents](../reference/schema.md#10-localized-package-documents)

- `docs.string-resources-package-only`: Package `StringResources` stores notification, data-name, permission-reason, `FormattedText`, and chart templates by stable package-local key. Function and type documents have no resource table. A protocol peer already knows the task's App package and therefore needs no type-owner metadata or resource FQN.
- `schema.names.identity-versus-localization`: This is why schema `NativeName` values must remain stable while natural localized names belong in the document.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### List item annotations

Owner: [annotations.md#list-item-annotations](../reference/annotations.md#list-item-annotations)

- `annot.list-item-divider`: The List divider assigns following annotations to the item; Studio warns for annotations that also support the List

### Package, discovery, and parser helpers

Owner: [annotations.md#package-discovery-and-parser-helpers](../reference/annotations.md#package-discovery-and-parser-helpers)

- `annot.default-locale.placement`: DefaultLocale goes once in the entry file package documentation; omission means en

### Function-level task declarations

Owner: [annotations.md#function-level-task-declarations](../reference/annotations.md#function-level-task-declarations)

- `annot.permissions.placement`: Permissions resolves one positional constant on a public App function or server Endpoint field
- `annot.function.no-other-attributes`: Functions accept only the public function attributes, never field, protocol-role, chart or Matter annotations

### Defaults, constraints, and ordinary presentation

Owner: [annotations.md#defaults-constraints-and-ordinary-presentation](../reference/annotations.md#defaults-constraints-and-ordinary-presentation)

- `dynamic.annotation.source`: Dynamic accepts one quoted positional string in TypeScript and one Rust string literal

### Eligibility and field placement

Owner: [annotations.md#eligibility-and-field-placement](../reference/annotations.md#eligibility-and-field-placement)

- `cache.identity.placement`: Mark only direct fields of the request root record; nested, array and Union-branch fields cannot carry CacheIdentity

### MIME-aware Bytes in workflow drafts

Owner: [annotations.md#mime-aware-bytes-in-workflow-drafts](../reference/annotations.md#mime-aware-bytes-in-workflow-drafts)

- `cache.mime.not-identity`: MimeTypes cannot occur within a CacheIdentity field

### Common field metadata

Owner: [presentation.md#common-field-metadata](../reference/presentation.md#common-field-metadata)

- `field.ask-on-add.placement`: `AskOnAdd` goes on a field with a value of its own in a List member Table, or on a List whose member has one; never Hidden, ReadOnly, Fixed or OutputOnly

### Bitflags

Owner: [presentation.md#bitflags](../reference/presentation.md#bitflags)

- `bitflags.zero-label.answer`: A Bitflags field is never Nullable; a required one answers none with its zero label

### Shared CopyFrom and CopyTo rules

Owner: [schema.md#shared-copyfrom-and-copyto-rules](../reference/schema.md#shared-copyfrom-and-copyto-rules)

- `copy.fixed.no-copyfrom`: A field cannot combine Fixed and CopyFrom
- `copy.copyto.exactly-one-match`: Exactly one CopyTo destination matches; several matches are rejected before any assignment
- `copy.types.compatible`: CopyFrom and CopyTo require compatible receiving types and preserved operation ownership; runtime checks actual values, bindings and unresolved ambiguity

### Context and syntax

Owner: [schema.md#context-and-syntax](../reference/schema.md#context-and-syntax)

- `paths.context.enumsource-baseobjects`: EnumSource takes one `$`, `@`, `^` or typed-ancestor path; BaseObjects a comma list of `$`, `@` or `^` paths
- `paths.context.llmdoctrans-validation`: LLMDocTrans validation takes one `$`, `@` or `^` path to a Table of the edited Request
- `paths.context.ordinary-copyfrom`: Ordinary CopyFrom takes strict `^<Variant...>`, `$` or `&` alternatives read from the interaction reference context
- `paths.context.link-bound-occurrence`: On a Response or SubscriptionData link target a typed selector names the tapped node's bound occurrence
- `paths.context.link-copy`: Request-editor link CopyFrom and link-return CopyTo bind from the host field's occurrence
- `paths.context.appendto`: AppendTo takes exactly one `Variant.field` naming a root array of an admitting SubscriptionData
- `paths.context.appendindex`: AppendIndex takes exactly one root-field `$.field` control field
- `paths.context.constrainedby`: ConstrainedBy takes one root-relative path, or on a request field a `^` or typed-ancestor path
- `paths.context.reduced-subscriptiondata`: Reduced SubscriptionData CopyTo takes strict `^<StableVariant...>` destinations
- `paths.context.current-data`: Square validation references read the owner's evaluation root or a bound cursor
- `paths.context.external-reference`: Curly validation references take dot-only `{$.path}` alternatives from the reference context
- `paths.host-relative.syntax`: Host-relative paths use `.name` or bracket children; `$` is the root, `@` the host, each `^` an actual parent
- `paths.copy.grammar`: Copy-path indices are literal non-negative integers and identifiers ASCII; no quoted bracket names
- `paths.copy.strict-variant-rooted`: Strict `^<Variant...>` copy paths name every schema level, including selected nested Union branches

### Typed ancestors and bound occurrences

Owner: [schema.md#typed-ancestors-and-bound-occurrences](../reference/schema.md#typed-ancestors-and-bound-occurrences)

- `paths.typed.host-field-selectors`: EnumSource and request-field ConstrainedBy selectors resolve to one parent-relative path for every host occurrence
- `paths.typed.link-bound-occurrence`: Ordinary CopyFrom accepts a bound occurrence only on a Response or SubscriptionData link target

### Current-data references in validation expressions

Owner: [schema.md#current-data-references-in-validation-expressions](../reference/schema.md#current-data-references-in-validation-expressions)

- `paths.validation.parent-prefix`: A leading `^` run ascends from the annotated value, only at the start and never above the data root
- `paths.validation.whitespace`: Whitespace around segments and references is ignored; no index expression is evaluated

### 2.4 Runtime types

Owner: [schema.md#24-runtime-types](../reference/schema.md#24-runtime-types)

- `types.action.unsupported`: The reserved `Action` type is unsupported in App source
- `types.number.widths`: Numbers carry `NumberType`: TypeScript names the width and Rust derives it from the native type

### 3.1 Local named types

Owner: [schema.md#31-local-named-types](../reference/schema.md#31-local-named-types)

- `types.local.root-exported`: Every App-owned type is publicly exported from the App root under exactly its declaration name

### 3.2 Foreign wire types

Owner: [schema.md#32-foreign-wire-types](../reference/schema.md#32-foreign-wire-types)

- `types.compatible.placement`: `ForeignTypeSchema` goes only on a published named compatible type and names a non-compatible official FQN
- `types.compatible.studio-validates`: Studio validates a compatible type's placement, external official target and structural compatibility

### 4. Attribute model

Owner: [schema.md#4-attribute-model](../reference/schema.md#4-attribute-model)

- `attr.known-names-only`: Studio rejects an attribute name that neither this reference nor the annotation reference lists

### Document duplicate identity metadata

Owner: [schema.md#document-duplicate-identity-metadata](../reference/schema.md#document-duplicate-identity-metadata)

- `attr.llmdoctrans.config-shape`: LLMDocTrans config has four ordinary keys, an optional duplicate_identity string and an optional form_date_order string

### 4.2 Function attributes

Owner: [schema.md#42-function-attributes](../reference/schema.md#42-function-attributes)

- `attr.function.singleton`: `Singleton` is the exact marker `true` on a function: at most one task per Hub
- `attr.function.data-error-status`: `DataErrorStatus` is the exact marker `true`, valid only on a function

### 4.3 Value, constraint, and presentation attributes

Owner: [schema.md#43-value-constraint-and-presentation-attributes](../reference/schema.md#43-value-constraint-and-presentation-attributes)

- `attr.default`: `$null` is a Default only on a Nullable field; a Union Default names one of its variants
- `attr.number-bounds`: Numeric bounds and step go on Number or DateTime; DateTime requires `u64`; the step counts from the minimum
- `attr.length-retained`: `LengthMin` and `LengthMax` are retained legacy bounds; current source uses `SizeMin` and `SizeMax`
- `attr.size`: `SizeMin` and `SizeMax` bound a List count, String UTF-8 bytes or Bytes length; a Table is no host
- `attr.unique`: `Unique` goes on a non-composite host with an integer level of at least one; the default is `99`
- `attr.unordered`: `Unordered` goes only on a List
- `attr.multiline`: `Multiline` goes only on a String; its value may contain line breaks, and a String without it contains none
- `attr.secret`: `Secret` goes only on a String, never together with `Multiline`, `Key` or `EnumSource`
- `attr.suppress-label`: `SuppressLabel` goes on a scalar, Table or Union, or on a List whose member is a Table
- `attr.date-only`: `DateOnly` is the marker `true` on an integer Number holding a `yyyyMMdd` calendar date
- `attr.bitflags`: `Bitflags` marks a fabricated Enumeration catalog, or names one on an integer Number
- `attr.mime-types`: `MimeTypes` goes on an effective Bytes host, never with FormattedText or inside a CacheIdentity field
- `attr.physical-unit`: `PhysicalUnit` is one canonical CLDR identifier on a Number and never changes raw values
- `attr.export-no-attribute`: `@LibertasExport` emits no attribute; a schema still carrying `Library=true` is rejected
- `bitflags.catalog-values`: A Bitflags integer host must be wide enough for all declared bits

### Numeric semantic exclusions

Owner: [schema.md#numeric-semantic-exclusions](../reference/schema.md#numeric-semantic-exclusions)

- `numeric.exclusions`: Numeric semantic attributes exclude each other except the pairs the exclusion table permits

### Reserved Default values and constant references

Owner: [schema.md#reserved-default-values-and-constant-references](../reference/schema.md#reserved-default-values-and-constant-references)

- `defaults.reserved.null`: `$null` is a Default only on a Nullable field, never `Fixed`

### User and group defaults

Owner: [schema.md#user-and-group-defaults](../reference/schema.md#user-and-group-defaults)

- `user.default-selectors`: User Default selectors are case-sensitive: a scalar takes one, a List a comma sequence; unknown tokens are errors

### Fixed current-user security constraint

Owner: [schema.md#fixed-current-user-security-constraint](../reference/schema.md#fixed-current-user-security-constraint)

- `fixed-user.current-user-only`: `Fixed` on a User is a Hub-checked security constraint whose only valid value is `current-user`
- `fixed-user.placement`: `Fixed(current-user)` needs a scalar User in a request payload outside Lists; any other use is an error

### MIME-aware Bytes

Owner: [schema.md#mime-aware-bytes](../reference/schema.md#mime-aware-bytes)

- `mime.filter-grammar`: `MimeTypes` is a unique lowercase comma list of exact types or ranges; a field replaces its type's filter

### Dynamic schema properties

Owner: [schema.md#dynamic-schema-properties](../reference/schema.md#dynamic-schema-properties)

- `dynamic.schema.declaration`: Dynamic defines typed operation properties in one schema attribute
- `dynamic.schema.one-definition`: Different Dynamic properties combine across declarations; a duplicate property on one control is an error
- `dynamic.schema.context`: Dynamic expressions resolve against each actual occurrence; List-field Removable resolves against the member

### 5.3 Enumerations

Owner: [schema.md#53-enumerations](../reference/schema.md#53-enumerations)

- `types.enum.default-normalized`: An Enumeration `Default` is a member name or fixed value, normalized to the member's numeric value
- `protocol-variant.placement`: `ProtocolVariant` goes only on an ordinary Enumeration member and names one variant of each consuming protocol

### 5.4 Unions

Owner: [schema.md#54-unions](../reference/schema.md#54-unions)

- `union.tag-field-script-only`: `UnionTagField=true` is ScriptLanguage-only metadata, never emitted for Rust
- `union.discriminator`: Each Union member's `Discriminator` equals its zero-based declaration index; `Fixed` never identifies a member

### 6.3 Function data schemas

Owner: [schema.md#63-function-data-schemas](../reference/schema.md#63-function-data-schemas)

- `data-schema.function-binding`: `FunctionDataSchema` names key and value Unions; external targets use FQNs and add dependencies

### 7. Path-based object and choice binding

Owner: [schema.md#7-path-based-object-and-choice-binding](../reference/schema.md#7-path-based-object-and-choice-binding)

- `enumsource.path-and-host`: `EnumSource` has one path to a same-tree List; without a key the host is an unsigned integer index

### Enumsource keys

Owner: [schema.md#enumsource-keys](../reference/schema.md#enumsource-keys)

- `enumsource.keyed-host`: A keyed target List makes the host store the member's key; the host type equals the key's
- `key.placement`: `Key` is the marker `true` on a direct Table field only
- `key.type`: A key field is a non-Nullable String or integer Number
- `key.unique`: A key field also carries `Unique`
- `key.one-per-table`: A Table has at most one key field, counting inherited fields
- `key.list-member-only`: A keyed Table occurs only as a List member

### Enumsource list hosts

Owner: [schema.md#enumsource-list-hosts](../reference/schema.md#enumsource-list-hosts)

- `enumsource.list-host`: A List EnumSource host has non-Nullable scalar elements of the key or index type

### 8. Types-only and interoperable packages

Owner: [schema.md#8-types-only-and-interoperable-packages](../reference/schema.md#8-types-only-and-interoperable-packages)

- `types.compatible.one-declaration`: A compatible type declares exactly one `ForeignTypeSchema`

### Dynamic property expressions

Owner: [validation-rules.md#dynamic-property-expressions](../reference/validation-rules.md#dynamic-property-expressions)

- `dynamic.expression.grammar`: Dynamic reuses typed expression tokens with Name = expression; assignments

<!-- /generated -->
