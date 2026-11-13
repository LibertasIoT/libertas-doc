# Rust App rules

Load this card when writing or changing a Rust App: its Cargo project and
`src/lib.rs`, schema types and their derives, exports, Rust attribute spelling
and placement, SDK runtime calls, persistent data, string resources, and native
build metadata. Its rules are generated from the
[Rust manual](../reference/languages/rust.md), which owns them; the
[Rust quick guide](rust.md) gives the framing and the
[annotation inventory](schema_annotations.md) the complete spelling table.
Rules shared by every language stay in their owning cards and are linked from
the manual: the semantics of each attribute, calculations, validation rules,
presentation, documentation and protocols.

## Order of work

1. Set up the library, SDK pins and discovery roots (`rust.project.*`,
   `rust.sdk.*`, `rust.dependencies.*`, `rust.discovery.*`,
   `rust.types-only.*`).
2. Shape the schema types and exports (`rust.shape.*`, `rust.exports.*`,
   `rust.scope.*`, `rust.types.*`, `rust.enum.*`, `rust.union.*`,
   `rust.foreign-type.*`).
3. Document every schema-visible declaration in rustdoc (`rust.doc.*`).
4. Spell each attribute exactly (`rust.attr.*`, `rust.number.*`,
   `rust.size.*`, `rust.pattern.*`, `rust.const.*`, `rust.bitflags.*`,
   `rust.array.*`, `rust.enum-source.*`, `rust.enum-attr.*`,
   `rust.llm-doc.*`, `rust.matter.*`, `rust.presentation.*`,
   `rust.formatted-text.*`, `rust.chart.*`, `rust.chart3d.*`,
   `rust.ai.*`).
5. Declare the endpoint protocol (`rust.endpoint.*`, `rust.protocol.*`,
   `rust.conformance.*`, `rust.calc.*`, `rust.workflow.*`, `rust.link.*`,
   `rust.choices.*`, `rust.append.*`, `rust.validation.*`).
6. Implement the task: functions, permissions, hosts, string resources, data
   and runtime (`rust.singleton.*`, `rust.permissions.*`,
   `rust.access-host.*`, `rust.string-resources.*`, `rust.data.*`,
   `rust.codec.*`, `rust.tokio.*`).
7. Test and build (`rust.testing.*`, `rust.build.*`), then compile with Studio
   and fix every reported rule.

<!-- generated:codebot-rules task=rust-app -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Project contract and library requirements

Owner: [rust.md#project-contract-and-library-requirements](../reference/languages/rust.md#project-contract-and-library-requirements)

- `rust.sdk.rustdoc-normative`: For runtime SDK behavior that is not part of schema extraction, follow the public rustdoc comments in [`crates/libertas/src/lib.rs`](https://github.com/LibertasIoT/libertas-rs/blob/main/crates/libertas/src/lib.rs) and the runtime implementation. In particular, the `LibertasEndpoint` rustdoc is normative for endpoint subscription and signaling behavior.
- `rust.sdk.identity-aliases`: The SDK represents device, endpoint, LAN-device, user, task, and peer identities with `i64` aliases. Durable allocated IDs are positive; negative IDs identify runtime dynamic objects and are not durable references. Retain peer handles unchanged, including negative handles. Transaction and timer IDs remain 32-bit.
- `rust.sdk.rebuild-for-hub` (judgment): Rebuild Apps with this SDK for the matching Hub runtime; existing task records can be retained while tasks are disabled for the update.
- `rust.project.libertas-crate`: **Global types**: Use the `libertas` crate to access native platform types.
- `rust.project.cargo-check`: **Standard Rust**: Your library must compile successfully (`cargo check --lib` should pass) before the UI schemas can be generated.
- `rust.project.scan-entry`: **Entry point**: Schema scanning always starts from the application library root, `src/lib.rs`.

### Optional Tokio support

Owner: [rust.md#optional-tokio-support](../reference/languages/rust.md#optional-tokio-support)

- `rust.tokio.feature`: SDK revisions that provide the `tokio` Cargo feature expose the coroutine API as `libertas::tokio`. Keep the project's SDK pin and enable that feature when using this API. The default core remains a single-threaded `no_std` SDK with no Tokio dependency compiled; the App selects its own Tokio executor and its required executor, I/O, and timer features.
- `rust.tokio.dispatcher-startup`: During normal synchronous App startup, attach a `Dispatcher` and bind typed Avro commands to complete synchronous SDK handlers, or register a typed endpoint through `Dispatcher::endpoint`. Share the resulting `Service` and `Endpoint` handles with coroutines. Awaited calls use a bounded queue and individual replies; the dispatcher executes SDK operations on the host thread.
- `rust.tokio.host-callbacks-never-block`: Keep direct core SDK calls in those handlers or normal host callbacks. Never block the host callback loop waiting for a dispatched call.
- `rust.tokio.shutdown`: The App retains ownership of its workers and domain recovery. Await `Handle::shutdown_requested`, stop and join the workers, then call `Handle::complete_shutdown`.
- `rust.tokio.data-semantics`: A handler containing several Data operations retains their ordinary Data semantics; it does not create a new storage transaction.

### Matter SDK dependency

Owner: [rust.md#matter-sdk-dependency](../reference/languages/rust.md#matter-sdk-dependency)

- `rust.matter.sdk-dependency`: Add the Matter SDK only when the App uses Matter, preserving the project's existing branch, tag, or revision policy:

  ```toml
  [dependencies]
  libertas-matter = { git = "https://github.com/LibertasIoT/libertas-matter.git", default-features = false, features = ["alloc"] }
  ```

- `rust.dependencies.preserve-pins`: Do not silently change an existing dependency pin while making an unrelated App change. Exact runtime APIs and feature availability come from the selected SDK revision.

### Schema discovery and shape rules

Owner: [rust.md#schema-discovery-and-shape-rules](../reference/languages/rust.md#schema-discovery-and-shape-rules)

- `rust.discovery.public-functions`: Every public function declared in the root `src/lib.rs` is emitted in `Functions` unless its rustdoc contains the exact no-argument `/// #[libertas_ignored]` parser directive.
- `rust.discovery.export-wrapper-not-root`: `#[libertas_export]` is the SDK's function wrapper for consuming parser-only argument attributes during Rust compilation; it is not what makes a public function visible to the schema parser.
- `rust.discovery.minimum-transitive-types`: In an ordinary App, those functions are the only roots. The parser publishes the minimum transitive set of non-builtin types reached from function arguments and schema-reference attributes. Derives and schema attributes on an otherwise unreachable type do not publish it.
- `rust.discovery.types-only-export-roots`: In a `TypesOnly` App, every public package-owned named type exported from the root `lib.rs` is automatically a root unless it carries the no-argument `/// #[libertas_ignored]` directive.
- `rust.shape.union-variants`: In a Union enum, a tuple variant may contain exactly one payload; a named variant becomes an inline `Table`, and a unit variant becomes `Nil`.
- `rust.shape.wrapper-types`: `Option<T>` adds `Nullable=true`, `Box<T>` preserves the inner schema shape, `Vec<u8>` is `Bytes`, and other `Vec<T>` values are `List` with one item schema.

### Documentation and localization

Owner: [rust.md#documentation-and-localization](../reference/languages/rust.md#documentation-and-localization)

- `rust.doc.package-comments`: The `//!` comments define the **Package-level** documentation.
- `rust.doc.package-comments-placement`: They should only be placed at the very top of the application library root, `src/lib.rs`.
- `rust.doc.item-comments`: Standard Rustdoc comments (`///`) are used for documenting structs, enums, traits, and properties.
- `rust.doc.short-document`: **Short Document (First Line)**: The very first line of the comment is extracted as the **Short Document**, which is used as the Short Name or Display Name in the UI.
- `rust.doc.long-document`: **Long Document (Remaining Lines)**: Any remaining lines after the first line are extracted as the **Long Document**, which serves as the detailed Description field in the UI or LLM tool call description. Long descriptions use HTML-style flow: one logical CR/LF break becomes exactly one space, while two or more logical breaks become one LF paragraph break. CRLF counts as one break, so leave a blank rustdoc line only when a paragraph break is intended.
- `rust.doc.text-before-directives`: Put ordinary label/description text before extension blocks and put both before parser-directive rustdoc lines.
- `rust.doc.code-comments-ignored`: Standard double-slash `//` comments are **completely ignored**. Use them freely for internal code documentation.
- `rust.doc.replace-example-identity` (judgment): Replace the example Plaza identity before publication.
- `rust.doc.parameter-rustdoc`: Put parameter rustdoc directly on each typed parameter of a `#[libertas_export]` function.
- `rust.attr.no-inference-from-schema`: Do not infer an authoring attribute from generated schema metadata or a parser-internal alias.

### Native global types

Owner: [rust.md#native-global-types](../reference/languages/rust.md#native-global-types)

- `rust.types.native`: When defining properties and function parameters, utilize the provided native Libertas types. These types allow the platform to understand the specific semantic meaning of your data (which automatically translates into the correct UI controls like device pickers, date/time selectors, or specialized inputs):
- `rust.types.endpoint`: **`LibertasEndpoint`**: Represents a protocol endpoint for interactions with humans, AI, and other software.
- `rust.types.datetime`: **`LibertasDateTime`**: A timestamp. In the UI, this automatically displays in the user's local timezone.
- `rust.types.time-only`: **`LibertasTimeOnly`**: Provides a 24-hour time selector in the UI.
- `rust.types.bytes`: **`Vec<u8>`**: Maps to the built-in `Bytes` schema type for binary data.
- `rust.types.user`: **`LibertasUser`**: Presents a system user or user group picker. Use explicit [User and group defaults](../reference/schema.md#user-and-group-defaults) for initial suggestions; the picker does not infer the current user.
- `rust.types.device`: **`LibertasDevice`**: Presents a physical or logical device picker.
- `rust.types.virtual-device`: **`LibertasVirtualDevice`**: Presents a virtual device picker.
- `rust.types.lan-device`: **`LibertasLanDevice`**: Presents a WiFi or Ethernet network device picker.
- `rust.types.mime-types`: For a MIME-aware `Vec<u8>` field, use `#[libertas_mime_types("image/png,image/jpeg")]`. The parser emits `MimeTypes`;
- `rust.types.mime-helpers`: Apps can use `libertas::encode_mime_bytes("image/png", content)` to create a defined value and `libertas::decode_mime_bytes(bytes)` to inspect one; an empty `Vec<u8>` is the valid media-undefined value. Keep unannotated `Vec<u8>` fields for raw binary protocols.

### Matter status values

Owner: [rust.md#matter-status-values](../reference/languages/rust.md#matter-status-values)

- `rust.matter.status-types`: `libertas-matter` exposes client `MatterStatus` as a raw `u32`; zero means success. Virtual-device response APIs use `IMStatusCode`, or `MatterResponseStatus` for a standard cluster status:

  ```rust
  request.respond_status(IMStatusCode::InvalidAction);
  ```

### Struct composition

Owner: [rust.md#struct-composition](../reference/languages/rust.md#struct-composition)

- `rust.types.struct-composition`: You can build reusable models by nesting or composing public structs. A field whose type is another named struct becomes a reference to that named schema type. The referenced type retains its own fields, attributes, and documents; field-level attributes and comments belong to the reference node. Rust structs do not have schema inheritance, and composing one struct inside another does not copy or flatten the referenced definition into the owner.

### Enumerations (no payloads)

Owner: [rust.md#enumerations-no-payloads](../reference/languages/rust.md#enumerations-no-payloads)

- `rust.enum.enumerations`: Standard Rust enums without payloads will present to the user as a simple dropdown or selection list. A member can carry `#[libertas_protocol_variant("EditWages")]` under `#[derive(LibertasExport)]` to declare an [enum/protocol association](../reference/schema.md#enumeration-protocol-associations).

### Optional types

Owner: [rust.md#optional-types](../reference/languages/rust.md#optional-types)

- `rust.types.separate-input-output-records` (judgment): Separate user input, displayed output, and internal records when choosing types. `pub` makes a declaration available to schema discovery; it does not mean that people edit every occurrence. Follow the shared [type-selection guidance](../reference/app-contract.md#user-facing-and-non-user-facing-types).
- `rust.types.optional-text-string`: For ordinary optional text, use `String` with an explicit empty Default and size bounds that permit zero. Required text uses a positive minimum and can remain unanswered until the person enters it:

  ```rust
  /// Student name
  /// Describes the parts of a student's name that a school records.
  #[derive(LibertasExport, LibertasAvroDecode)]
  pub struct Student {
      /// First name
      /// Enter the student's first name as recorded by the school.
      #[libertas_size(min = 1, max = 120)]
      pub first_name: String,

      /// Middle name
      /// Enter the student's middle name. Leave it empty if absent.
      #[libertas_default("")]
      #[libertas_size(min = 0, max = 120)]
      pub middle_name: String,
  }
  ```

- `rust.types.size-min-not-default`: `SizeMin=0` alone would not initialize a String to `""`; Rust's `String::default()` is not a Schema Default either.
- `rust.types.option-for-absence` (judgment): Use `Option<T>` when absence is a distinct, valid domain answer. For example:

  ```rust
  /// Reported payment
  /// Describes the amount known for one payment.
  #[derive(LibertasExport, LibertasAvroDecode)]
  pub struct ReportedPayment {
      /// Payment amount
      /// Enter the known payment amount, including zero when no money was paid.
      /// Omit the amount when it is unknown.
      #[libertas_money("USD", 2)]
      pub amount: Option<i64>,
  }
  ```

- `rust.types.retain-nullable-text` (judgment): Retain Nullable text when absence must differ from a present answer, including an optional identifier with `#[libertas_size(min = 9, max = 10)]`: absence is valid, but any supplied identifier must meet that length constraint.
- `rust.types.option-string-minimum`: `Option<String>` with a minimum of 1 is valid. The size rule applies to the present String; it rejects `Some("")`, not `None`. Prefer the simpler blank String for ordinary optional text when this distinction is unnecessary.

### Labeled decisions and internal absence

Owner: [rust.md#labeled-decisions-and-internal-absence](../reference/languages/rust.md#labeled-decisions-and-internal-absence)

- `rust.types.labeled-decisions` (judgment): A yes-or-no decision whose “yes” carries details can stay an `Option<T>`: its editor is a two-answer choice labelled by `[NullText]` and `[SetText]`, and with no Default the person must choose (see [Nullable choices](../reference/app-contract.md#nullable-choices)). Use a named Union when there are three or more answers or the negative answer carries data. Its localized labels explain what the person chooses, and a selected payload branch exposes the relevant input. Use an Enumeration when neither choice needs a payload. For example:

  ```rust
  /// Email contact preference
  /// Choose whether email contact is wanted and where messages should be sent.
  #[derive(LibertasExport, LibertasAvroDecode)]
  #[libertas_inline_choices]
  pub enum EmailContact {
      /// No email contact
      /// Choose this when email messages are not wanted.
      NoEmail,

      /// Receive email
      /// Choose this to receive messages at the supplied email address.
      Receive {
          /// Email address
          /// Enter the address where messages should be sent.
          #[libertas_size(min = 1, max = 254)]
          address: String,
      },
  }
  ```

- `rust.types.default-not-confirmation` (judgment): Defaulting a Union to its empty branch also removes the requirement to make a new choice; a Default is not proof of confirmation.
- `rust.types.meaningful-choice-labels`: Native names such as `Nothing` and `Something` are permitted, but their comments must provide meaningful standalone choice labels.
- `rust.types.blank-absence-boundary` (judgment): For an internal email address that only needs presence or absence, `Option<String>` avoids an unnecessary named choice type. If an input String uses blank for absence while an existing record uses `Option<String>`, map at the boundary, for example:

  ```rust
  fn record_text(input: String) -> Option<String> {
      if input.is_empty() { None } else { Some(input) }
  }

  fn editor_text(stored: Option<String>) -> String {
      stored.unwrap_or_default()
  }
  ```

- `rust.types.blank-absence-policy` (judgment): Use this mapping only when exact empty text means absence in that domain; trimming whitespace would be a separate policy. Keep a reusable type when its meaning and constraints fit both uses.
- `rust.types.foldable-not-union` (judgment): If the goal is only to collapse extra details, use `#[libertas_foldable]` on a struct: [folding](../reference/presentation.md#folding-structs-and-unions) keeps its values, while selecting another Union branch changes the answer. Follow the shared [representation-change review](../reference/app-contract.md#reviewing-a-representation-change) for validation, fallbacks, grouping, import, and published compatibility.

### Enum and variant documentation

Owner: [rust.md#enum-and-variant-documentation](../reference/languages/rust.md#enum-and-variant-documentation)

- `rust.doc.variant-precedence`: When a Union enum variant wraps a named type, documentation on the variant takes precedence over documentation on that named type. Attributes from the variant and referenced type are combined.

### Schema attributes and field presentation

Owner: [rust.md#schema-attributes-and-field-presentation](../reference/languages/rust.md#schema-attributes-and-field-presentation)

- `rust.attr.argument-forms`: In particular, `libertas_endpoint_schema`, `libertas_copy_from`, `libertas_device_type`, `libertas_virtual_device_type`, and `libertas_foreign_type` take positional values rather than key-value arguments ([Endpoint forms](../reference/annotations.md#endpoint-and-protocol-annotations), [Matter and external-type forms](../reference/annotations.md#external-types-matter-descriptors-and-union-metadata)).
- `rust.string-resources.package-level`: `libertas_string_resources` is package-level source metadata written in the leading inner documentation as `//! #[libertas_string_resources(CONST_NAME, …)]` ([shared contract](../reference/message-format.md#uses-and-storage)). Each name resolves through the crate's ordinary scope, including module paths and `pub use` re-exports, to a literal `&[(&str, &str)]` constant; the parser follows modules whatever their `cfg`, so catalog modules may be `#[cfg(test)]`.

### General field attributes

Owner: [rust.md#general-field-attributes](../reference/languages/rust.md#general-field-attributes)

- `rust.attr.default-form`: `#[libertas_default(value)]` is the canonical input-default form. The named `#[libertas_default(default = value)]` variation remains parser compatibility syntax and codebots must not generate it.
- `rust.attr.default-values`: Enumeration member names or numeric defaults follow the [shared enumeration contract](../reference/schema.md#53-enumerations). User and User-list defaults use the [identity-selector contract](../reference/schema.md#user-and-group-defaults), for example `#[libertas_default("current-user")]`.
- `rust.attr.fixed`: `#[libertas_fixed(value)]` emits one fixed encoded value ([shared contract](../reference/schema.md#43-value-constraint-and-presentation-attributes)).
- `rust.attr.read-only`: `#[libertas_read_only]` emits `ReadOnly=true`.
- `rust.attr.output-only`: `#[libertas_output_only]` emits `OutputOnly=true`
- `rust.attr.hidden`: `#[libertas_hidden]` emits `Hidden=true`.
- `rust.llm-doc.trans-and-field`: For document-assisted Request entry, place `#[libertas_llm_doc_trans(instruction = "...", validation = "@.facts", doc_type = "image/*", accept_multiple_inputs = true)]` ([shared contract](../reference/llm-document-transformation.md#app-schema-authoring))
- `rust.llm-doc.blank-text-agreement`: The field comments and instruction agree on missing text; JSON `null` is reserved for Nullable destinations.
- `rust.llm-doc.numeric-blank-default`: A non-Nullable numeric field defaults to `allow_empty = true` and therefore needs a meaningful `#[libertas_default(value)]` for a truly blank form box.
- `rust.llm-doc.blank-literal`: To apply a specific value instead, declare it with a Default literal: `#[libertas_llm_doc_field(blank = 0)]`, `blank = $null` on an `Option` field, or `blank = ShortTerm` naming an enum variant.
- `rust.attr.exclude-ui`: `#[libertas_exclude_ui]` is deliberately narrower than `libertas_hidden`. It emits `ExcludeUI=true`
- `rust.attr.llm-tool-call`: `#[libertas_llm_tool_call(false)]` on a protocol enum excludes its Request variants from Hub LLM tool definitions ([shared contract](../reference/schema.md#45-union-endpoint-and-data-attributes)).

### Promotion, flattening, labels, and titles

Owner: [rust.md#promotion-flattening-labels-and-titles](../reference/languages/rust.md#promotion-flattening-labels-and-titles)

- `rust.presentation.suppress-label`: `#[libertas_suppress_label]` suppresses a scalar, struct, or Union row’s visual label while retaining its value and controls.

### Inline descriptions

Owner: [rust.md#inline-descriptions](../reference/languages/rust.md#inline-descriptions)

- `rust.presentation.include-description`: `#[libertas_ui_include_description]` on a schema data field or type is compiled into the [Display Template](../reference/presentation.md#display-template) as its `include_description` hint. Clients keep the localized short display name and also render the localized long description inline in that UI node. The marker is client-only presentation metadata and does not change the typed value or wire encoding.

### Matter device descriptor attributes

Owner: [rust.md#matter-device-descriptor-attributes](../reference/languages/rust.md#matter-device-descriptor-attributes)

- `rust.matter.device-type`:

  | Rust attribute | Required effective host | Emitted schema attribute |
  | --- | --- | --- |
  | `#[libertas_device_type("encodedDescriptor")]` | `LibertasDevice` | `DeviceType=encodedDescriptor` ([shared contract](../reference/matter/descriptors.md#schema-mapping)) |

- `rust.matter.virtual-device-type`:

  | Rust attribute | Required effective host | Emitted schema attribute |
  | --- | --- | --- |
  | `#[libertas_virtual_device_type("encodedDescriptor")]` | `LibertasVirtualDevice` | `VirtualDevice=encodedDescriptor` ([shared contract](../reference/matter/descriptors.md#schema-mapping)) |

- `rust.matter.descriptor-argument`: The meaningful argument is one positional string. The parser removes the Rust string quotes and stores its contents verbatim; it does not decode or validate the descriptor. Studio's Device Type Editor and Virtual Device Type Editor are the canonical generators. They produce the versioned Base64 attribute value:

  ```text
  Attribute.Value = Base64(uvarint32(formatVersion) || descriptorBody)
  ```

  ```rust
  /// Matter lighting choices
  /// Describes the physical and virtual lights used for one lighting task.
  #[derive(LibertasExport, LibertasAvroDecode)]
  pub struct MatterConfiguration {
      /// Room light
      /// The person chooses the physical light controlled by this task.
      #[libertas_device_type("BQEBAYACAQYD")]
      pub light: LibertasDevice,

      /// Optional virtual light
      /// The person may choose a virtual light to mirror the room light; leaving
      /// it empty means that no virtual light is used.
      #[libertas_virtual_device_type("BQEBgAIBAQYBBAAAAQAAAwABAgA=")]
      pub virtual_light: Option<LibertasVirtualDevice>,
  }
  ```

### Numeric modifiers

Owner: [rust.md#numeric-modifiers](../reference/languages/rust.md#numeric-modifiers)

- `rust.number.explicit-bounds-replace-defaults`: Explicit bounds and step replace the corresponding native defaults, including through `Option<T>` and `Box<T>`; omitted settings retain their native defaults.
- `rust.number.time-interval`: `#[libertas_time_interval]`: In the Task Editor, this triggers a specialized `HH:MM` time interval input for users.
- `rust.number.date-only`: `#[libertas_date_only]`: No-argument marker
- `rust.bitflags.forms`: `#[libertas_bitflags]` / `#[libertas_bitflags(TypeName)]`: The no-argument form projects a struct or `bitflags!` declaration into a localized Enumeration catalog. The one-type form projects that catalog onto an integer field without changing its integer wire value.

### Bitflag catalogs and integer fields

Owner: [rust.md#bitflag-catalogs-and-integer-fields](../reference/languages/rust.md#bitflag-catalogs-and-integer-fields)

- `rust.bitflags.declaration`: Put no-argument `#[libertas_bitflags]` on a struct declaration. It may be a plain struct with associated integer constants or a declaration inside the popular [`bitflags!`](https://docs.rs/bitflags/latest/bitflags/macro.bitflags.html) macro. The declaration may remain private because the parser fabricates a schema-only `Enumeration`; it is never the runtime or wire type.

### Size constraints

Owner: [rust.md#size-constraints](../reference/languages/rust.md#size-constraints)

- `rust.size.attribute`: `#[libertas_size(min = value, max = value)]`: Restricts size ([shared contract](../reference/schema.md#43-value-constraint-and-presentation-attributes))

### Constant references

Owner: [rust.md#constant-references](../reference/languages/rust.md#constant-references)

- `rust.const.reference`: A `$NAME` names a `const` item of your crate, so one value is written once. The parser evaluates the constant while it reads the source and emits its literal, so the published schema is unchanged:

  ```rust
  pub const MAX_WAGES: i64 = 1_000_000 * 100;
  pub const LABEL_LIMIT: u32 = 120;

  #[derive(LibertasExport, LibertasAvroDecode)]
  pub struct Wages {
      #[libertas_number(min = 0, max = $MAX_WAGES)]
      #[libertas_validation_rules("[] <= $MAX_WAGES : \"ABOVE_MAXIMUM\";")]
      pub amount: i64,
      #[libertas_size(max = $LABEL_LIMIT)]
      pub label: String,
  }
  ```

- `rust.const.substitution-sites`: In rule text (`libertas_validation_rules`, `libertas_range_min`, `libertas_range_max`, and quoted `min`/`max` arguments of `libertas_number`) it is replaced wherever it appears outside a double-quoted string literal. Path attributes (`libertas_enum_source`, `libertas_constrained_by`, `libertas_copy_from`, `libertas_copy_to`) are never touched: there `$` is the request root of the [Libertas Schema Path](../reference/schema.md#libertas-schema-path), and a `$` in rule text not followed by a name, as in `{$.limits.maximum}`, stays as written.
- `rust.const.in-scope`: **In Rust scope.** In a static bound (`min`, `max` or `step` of `libertas_number`, `min` or `max` of `libertas_size`), the reference must also resolve as a Rust path where the field is declared: import the constant with `use`, or write `$crate::module::NAME`. The SDK's Avro decoder enforces the bound by comparing against that same constant, converted exactly to the field's type, so a name that is out of scope or converts lossily is a compile error.

### Enumerations declared by attribute

Owner: [rust.md#enumerations-declared-by-attribute](../reference/languages/rust.md#enumerations-declared-by-attribute)

- `rust.enum-attr.compatibility-only`: The parser can turn a public type alias into an `Enumeration` with `#[libertas_enum(...)]`. For compilable Rust, put this parser-only declaration on an exact rustdoc line because the SDK does not export `libertas_enum` as a standalone attribute macro for type aliases. This positional form cannot attach source documentation to each fabricated member, so it cannot satisfy the codebot documentation contract. It is parser-supported compatibility syntax, not a form that codebots may generate.
- `rust.enum-attr.prefer-native-enum`: Because those fabricated members have no per-member source documentation syntax, a codebot must prefer a documented native Rust enum for every authored reusable choice catalog and must not generate the positional form until such syntax exists:

  ```rust
  /// Account role
  /// Classifies the responsibilities assigned to one account.
  #[derive(LibertasExport, LibertasAvroDecode)]
  pub enum Role {
      /// Administrator
      /// Allows the account to manage people and shared settings.
      Admin,

      /// Standard member
      /// Allows the account to use shared services without managing accounts.
      User,

      /// Guest
      /// Allows the account to use only the services explicitly shared with it.
      Guest,
  }
  ```

### Array attributes

Owner: [rust.md#array-attributes](../reference/languages/rust.md#array-attributes)

- `rust.array.unordered`: `#[libertas_unordered]`: Marks an array as an unordered set, affecting how arrays are merged or displayed.
- `rust.array.unique`: `#[libertas_unique(level)]`: Emits the `Unique` ancestor level ([shared contract](../reference/schema.md#43-value-constraint-and-presentation-attributes))
- `rust.array.key`: `#[libertas_key]`: No-argument marker emitting `Key=true` ([shared contract](../reference/schema.md#enumsource-keys))

### Union content naming

Owner: [rust.md#union-content-naming](../reference/languages/rust.md#union-content-naming)

- `rust.union.content-name`: The parser supports `UnionContentName` through the exact parser-directive rustdoc line shown below. The outer `#[libertas_export]` compiler macro is function-only, so do not apply that spelling directly to an enum. See the [canonical rule](../reference/annotations.md#package-discovery-and-parser-helpers).

### Data-backed enum indexes

Owner: [rust.md#data-backed-enum-indexes](../reference/languages/rust.md#data-backed-enum-indexes)

- `rust.enum-source.attribute`: Use `#[libertas_enum_source("path")]`
- `rust.enum-source.stable-key`: To store a stable identity instead, make the list's member a struct and mark one non-`Option` `String` or integer field of it with `#[libertas_key]` and `#[libertas_unique]`.

### Functions, persistent data, and resources

Owner: [rust.md#functions-persistent-data-and-resources](../reference/languages/rust.md#functions-persistent-data-and-resources)

- `rust.types-only.non-type-exports`: In a `TypesOnly` package, exact-name public re-exports of schema types are roots. Re-exported constants, statics, and helper functions are not schema types and do not become type roots. Unresolved re-exports and invalid schema types still report errors; non-type exports need no exclusion annotation.

### Public functions

Owner: [rust.md#public-functions](../reference/languages/rust.md#public-functions)

- `rust.discovery.ignored-type`: The same directive may be placed on a public named type to omit that type as a root ([shared contract](../reference/languages/rust.md#schema-discovery-and-shape-rules)).
- `rust.singleton.attribute`: Use the no-argument `#[libertas_singleton]` attribute ([shared contract](../reference/schema.md#42-function-attributes))

### Persistent data schemas

Owner: [rust.md#persistent-data-schemas](../reference/languages/rust.md#persistent-data-schemas)

- `rust.data.error-status-attribute`: For the opt-in, import `libertas_macros::libertas_data_error_status` and mark the public task function with `#[libertas_data_error_status]` ([shared contract](../reference/schema.md#42-function-attributes)).
- `rust.data.checked-calls`: Checked calls use the `try_` method prefix and return `Result<T, DataError>`; ordinary methods retain default failure behavior, even on opted-in tasks.
- `rust.data.context`: Use `libertas::data::Data::<Keys, Values>::current()` for the synchronous context, or bind it with `libertas::tokio::Dispatcher::data` under the optional `tokio` feature. See the [transactional API](../reference/protocols.md#transactional-key-value-api) for targets, batches, ranges, transaction ownership, and limits.
- `rust.data.schema-attribute`: Every public task function declares both storage unions with `#[libertas_data_schema(key = DataKey, value = StoredData)]` ([shared contract](../reference/protocols.md#storage-schema-declarations)).
- `rust.data.key-sizes`: Key byte fields use the existing `#[libertas_size(max = N)]`, or explicit equal `min` and `max` for an exact-length payload.
- `rust.data.endpoint-schema-independent`: `#[libertas_endpoint_schema(Type)]` retains its independent protocol type reference. Neither declaration is a standalone marker on an enum.

### Encoding buffers

Owner: [rust.md#encoding-buffers](../reference/languages/rust.md#encoding-buffers)

- `rust.codec.sdk-encodes` (judgment): Apps pass typed keys and values to the storage and endpoint APIs. The SDK handles encoding and buffer selection internally; Apps do not need to call codecs or manage serialization buffers.
- `rust.codec.custom-key-codecs`: For custom codec implementations, `KeyCodec::encode_key_uninit` accepts an exactly sized `&mut [core::mem::MaybeUninit<u8>]` and returns initialized bytes on success. `encode_key_to_buffer` appends directly into InlineByteBuffer spare capacity; failure preserves the previously initialized prefix. The field counterpart is `KeyFieldCodec::encode_field_uninit`. Initialized-slice methods remain available. Generated `encoded_len()` selects a constant width in O(1). Manual key codecs implement `write_key` or `write_field` using KeyWriter, which checks complete initialization of each declared field.
- `rust.codec.custom-avro-encoders`: Manual Avro encoders implement `fn avro_encode(&self, buffer: &mut (impl libertas::ByteBuffer + ?Sized))`. ByteBuffer supplies `push` and `extend_from_slice`, with implementations for Vec and InlineByteBuffer; derives support both automatically. The generic method means AvroEncode is not a trait-object interface. Existing Vec calls retain their behavior and wire bytes; explicitly dereference buffer guards, for example `&mut *guard`. The low-level `to_key_bytes()` and derived `to_avro()` convenience methods continue returning Vec. These customization interfaces are independent of ordinary App API usage.

### Permissions

Owner: [rust.md#permissions](../reference/languages/rust.md#permissions)

- `rust.permissions.endpoint-server`: The same attribute may be placed on a built-in Endpoint field marked `#[libertas_endpoint_server]` ([shared contract](../reference/annotations.md#function-level-task-declarations)).

### Network host access

Owner: [rust.md#network-host-access](../reference/languages/rust.md#network-host-access)

- `rust.access-host.attribute`: Use `#[libertas_access_host("host1,host2")]` on a function to declare the DNS hostnames its task process needs to contact ([shared contract](../reference/network-access.md#declaration)).

### Endpoint schema

Owner: [rust.md#endpoint-schema](../reference/languages/rust.md#endpoint-schema)

- `rust.endpoint.schema-attribute`: Apply `#[libertas_endpoint_schema(MySchemaUnion)]` to a `LibertasEndpoint` field to define its application payload. The argument is positional, not `schema=...`. A simple name may be unquoted; a qualified Rust path may be quoted ([shared contract](../reference/protocols.md#declaring-an-endpoint)).
- `rust.endpoint.process-identity`: `#[libertas_default("developer-package::function")]` does not filter existing candidates; when none exists, it recommends provisioning that process ([shared contract](../reference/schema.md#43-value-constraint-and-presentation-attributes)).

### Protocol conformance during configuration

Owner: [rust.md#protocol-conformance-during-configuration](../reference/languages/rust.md#protocol-conformance-during-configuration)

- `rust.conformance.placement`: `#[libertas_protocol_conformance("VariantA & (VariantB | !VariantC)")]` declares variants required from the opposite endpoint when the annotated Union is used as a protocol ([shared contract](../reference/schema.md#61-endpoint-protocol-validation)).
- `rust.conformance.variant-precedence`: When placed on a Union-typed variant, the variant's expression becomes the conformance evaluation for the nested Union in that declaration context. This allows separate parent variants that use the same nested Union type to declare different requirements. The variant expression takes precedence over a `ProtocolConformance` expression on the referenced Union declaration for that specific use.

### Runtime checks for optional variants

Owner: [rust.md#runtime-checks-for-optional-variants](../reference/languages/rust.md#runtime-checks-for-optional-variants)

- `rust.conformance.runtime-check`: At runtime, the client retains the server's protocol declaration, while the server does not retain the client's declaration. Before sending a request that uses an optional payload, the client must inspect the retained server declaration. For a nested Union payload, it must verify every variant along the declaration path.
- `rust.conformance.variant-index`: Derive `VariantIndex` and use `variant_index!` from the public [`libertas_macros` crate](https://github.com/LibertasIoT/libertas-rs/tree/main/crates/libertas_macros) to obtain each zero-based declaration index as a compile-time constant. Use those constants with the runtime server-declaration check instead of embedding numeric ordinals:

  ```rust
  use libertas_macros::{variant_index, VariantIndex};

  /// Service actions
  /// Describes the routine and optional actions available from this service.
  #[derive(LibertasExport, LibertasAvroDecode, VariantIndex)]
  #[libertas_protocol_conformance("Ping | Optional")]
  pub enum ServiceProtocol {
      /// Check service availability
      /// Asks whether the service is currently available.
      #[libertas_request]
      Ping,

      /// Additional service action
      /// Offers an additional diagnostic or calibration action when the service
      /// supports it.
      #[libertas_protocol_conformance("ReadDiagnostics")]
      Optional(OptionalRequest),
  }

  /// Additional service actions
  /// Describes optional diagnostic and calibration actions.
  #[derive(LibertasExport, LibertasAvroDecode, VariantIndex)]
  pub enum OptionalRequest {
      /// Read service diagnostics
      /// Asks for the service's current diagnostic observations.
      #[libertas_request]
      ReadDiagnostics,

      /// Calibrate the service
      /// Asks the service to calibrate using the supplied calibration level.
      #[libertas_request]
      Calibrate(u8),
  }

  const OPTIONAL_INDEX: usize = variant_index!(ServiceProtocol::Optional);
  const CALIBRATE_INDEX: usize = variant_index!(OptionalRequest::Calibrate);
  ```

### Protocol roles and context

Owner: [rust.md#protocol-roles-and-context](../reference/languages/rust.md#protocol-roles-and-context)

- `rust.endpoint.base-objects`: `#[libertas_endpoint_base_objects("path1,path2")]` filters the endpoint's base-object context using one positional comma-delimited path string.
- `rust.protocol.error-marker.authoring`: Declare `#[libertas_error]` beside `#[libertas_response]` for a typed error result; [out-of-band error handling](../reference/protocols.md#protocol-roles) preserves the current interaction.
- `rust.protocol.error-field-path`: Optionally mark a Nullable String field (`Option<String>`) with the no-argument `#[libertas_error_field_path]` to emit `ErrorFieldPath=true`.
- `rust.protocol.reveal-path`: A read-only row type or a Request payload may mark one `Option<String>` field with the no-argument `#[libertas_reveal_path]` to emit `RevealPath=true`, and a Request or Response payload may mark one `#[libertas_hidden]` `String` field with the no-argument `#[libertas_landing_path]` to emit `LandingPath=true`.
- `rust.protocol.access-privilege`: On `#[libertas_request]` or `#[libertas_subscription_request]`, add `#[libertas_access_privilege("Read")]`, `#[libertas_access_privilege("Write")]`, or `#[libertas_access_privilege("Read,Write")]` to restrict which end users may invoke that runtime operation. The parser emits the language-neutral integer bitmap `AccessPrivilege=1`, `2`, or `3` ([shared contract](../reference/protocols.md#request-access)).
- `rust.protocol.next-request-response`: Use `#[libertas_next_request("...")]` and `#[libertas_next_response("...")]` to name allowed next variants. One comma-delimited string may name multiple targets ([shared contract](../reference/protocols.md#entering-and-continuing-a-workflow)).
- `rust.calc.calculation-request`: `#[libertas_calculation_request("CalculatePrice,CalculateShipping")]` is a field-only helper accepted by `LibertasExport`. Its one comma-delimited string lists the protocol Request variants that use the field.
- `rust.calc.calculation-root`: `#[libertas_calculation_root("CalculatePrice")]` marks the node a calculation is for; its comma-delimited string names the calculations it roots.
- `rust.calc.copy-to`: `#[libertas_copy_to("@.orderTotal")]` assigns a successful calculation response field into the originating request editor, inside the root's subtree.
- `rust.calc.copy-from`: `#[libertas_copy_from("$.path1,$.path2")]`
- `rust.workflow.request-and-prev`: `#[libertas_workflow_request("EditProfile")]` declares the [WorkflowRequest semi-role](../reference/protocols.md#workflowrequest-semi-role).
- `rust.link.link-only`: `#[libertas_link_only]`, with no argument,
- `rust.protocol.request-confirmation`: `#[libertas_request_confirmation]`, with no argument,

### Semantic validation rules

Owner: [rust.md#semantic-validation-rules](../reference/languages/rust.md#semantic-validation-rules)

- `rust.validation.rules-attribute`: Use `#[libertas_validation_rules("...")]` ([shared contract](../reference/validation-rules.md#3-rule-set))
- `rust.validation.range-attributes`: Use `#[libertas_range_min("expression")]` and `#[libertas_range_max("expression")]` on a Number or `LibertasDateTime` field or named type when its inclusive range comes from current data or the reference context:

  ```rust
  #[libertas_range_min("{$.limits.minimum}")]
  #[libertas_range_max("{$.limits.maximum}")]
  pub type Percentage = i32;
  ```

- `rust.validation.range-bound-syntax`: Each attribute contains one Data Validation bound expression compatible with the host type without a message key, rule-pair semicolon, or `FinalOnly`. A Boolean predicate such as `[] <= 100` is invalid; use `100` to set that maximum on an ordinary Number. Date controls require their respective semantic date types. The [dynamic range contract](../reference/validation-rules.md#31-dynamic-numeric-ranges) defines static/runtime type rejection, compatible date values, first-response absence, and enforcement in both the GUI and final validation.

### Formatted runtime text

Owner: [rust.md#formatted-runtime-text](../reference/languages/rust.md#formatted-runtime-text)

- `rust.formatted-text.marker`: Use the no-argument `#[libertas_formatted_text]` marker
- `rust.formatted-text.bytes`: Formatted bytes are the canonical resource key plus ordered, typed `LibertasMessageArgument` values; build them with `libertas_formatted_text`. Localized templates use zero-based LMF1, and runtime payloads never contain resource templates.
- `rust.formatted-text.no-base64-string`: Do not use a `String` host containing Base64. Rendering would only decode that String back to the same byte tuple, adding a pointless encode/decode cycle and providing no benefit. Keeping formatted values as Bytes creates a sharp type distinction from literal text, making developer mistakes less likely. App code keeps the value as `Vec<u8>`; if a generic JSON transport represents Bytes as Base64, that is only the transport codec and never the field's semantic type.
- `rust.formatted-text.sdk-function`: The public SDK function `libertas_formatted_text(resource_name, arguments)` returns the canonical `Vec<u8>`. App code ordinarily does not decode it; clients resolve and render the LMF1 template at the presentation boundary.

### Chart presentation

Owner: [rust.md#chart-presentation](../reference/languages/rust.md#chart-presentation)

- `rust.chart.attributes`: Chart metadata is part of the ordinary endpoint protocol schema. It adds no payload bytes or chart-specific message. Apply `#[libertas_chart(mark)]` ([shared contract](../reference/charts/2d.md#31-chart-metadata-belongs-to-returned-data))
- `rust.chart.tuple-variant-divider`: For a List carried directly by a tuple variant, put the container text above the variant's `----` divider and the item text below it. The parser assigns the two sections to the List and its child respectively; do not duplicate the container text on the unnamed tuple field.

### 3D chart source attributes

Owner: [rust.md#3d-chart-source-attributes](../reference/languages/rust.md#3d-chart-source-attributes)

- `rust.chart3d.attributes`: Three-dimensional charts use a separate set of parser-only Rust attributes so their metadata cannot enter the existing 2D chart path accidentally: `#[libertas_chart3d(...)]`, `#[libertas_chart3d_channel(...)]`, `#[libertas_chart3d_scale(...)]`, `#[libertas_chart3d_guide(...)]`, and `#[libertas_chart3d_view(...)]`. They emit the language-neutral `Chart3D`, `Chart3DChannel`, `Chart3DScale`, `Chart3DGuide`, and `Chart3DView` namespaces ([shared contract](../reference/annotations.md#three-dimensional-chart-annotations)).

### Runtime subscription and signaling

Owner: [rust.md#runtime-subscription-and-signaling](../reference/languages/rust.md#runtime-subscription-and-signaling)

- `rust.endpoint.signaling`: Use the [endpoint signaling orientation](../reference/endpoint-lifecycle.md) and the public `libertas-rs` endpoint rustdoc before implementing callbacks. The SDK owns the complete contract. Its Rust liveness entry point is `libertas_endpoint_peer_alive(server: LibertasEndpoint, peer: Option<LibertasPeer>)`.

### Types-only libraries

Owner: [rust.md#types-only-libraries](../reference/languages/rust.md#types-only-libraries)

- `rust.types-only.directive`: Place the exact no-argument package directive below in the leading `//!` documentation of `src/lib.rs` to publish reusable types, schemas, and localized documents without producing deployable executable code:

  ```rust
  //! Shared service messages
  //! Provides reusable actions and results for compatible services.
  //! #[libertas_types_only]
  ```

### Compatible external representations

Owner: [rust.md#compatible-external-representations](../reference/languages/rust.md#compatible-external-representations)

- `rust.foreign-type.attribute`: Use `#[libertas_foreign_type(developer-package::TypeName)]` ([shared contract](../reference/schema.md#32-foreign-wire-types))

### Testing validation rules

Owner: [rust.md#testing-validation-rules](../reference/languages/rust.md#testing-validation-rules)

- `rust.testing.test-rules` (judgment): Keep Rust language checks and tests alongside schema validation. To test client rules in CI, run Studio's installed schema compiler launcher with `test-rules` and your JSON fixtures. The Rust SDK's `examples/validation-rules` contains a types-only sample with passing, failing, and unanswered rule expectations. See [Debugging rules](../reference/validation-rules.md#debugging-rules) for the fixture format, exact expectations, launcher command, report and exit status; the runner uses the same compile path and evaluator as `evaluate_validation_rules`.

### Build and deployment metadata

Owner: [rust.md#build-and-deployment-metadata](../reference/languages/rust.md#build-and-deployment-metadata)

- `rust.build.build-packages`: **Build packages** provide compilers, generators, headers, and native metadata needed before Cargo can link the App. The developer declares these in `Cargo.toml` because Cargo metadata cannot reliably infer distribution package names from a Rust dependency graph.
- `rust.build.logical-dependencies`: Logical Libertas dependencies are separate. The [deployment dependency contract](../reference/deployment.md) owns their discovery, preflight, exact pinning, and dependency-first deployment. Cargo metadata must not predict or maintain generated Hub App versions.

### Local development and public release sources

Owner: [rust.md#local-development-and-public-release-sources](../reference/languages/rust.md#local-development-and-public-release-sources)

- `rust.build.source-identity`: Use local Cargo path dependencies during internal development and pinned public Git dependencies when those packages are released. Keep one SDK source identity throughout the native dependency graph; a local SDK and a Git checkout are separate Cargo crates even when their package names and versions match.
- `rust.build.upload-limits`: Source size is limited only by the Hub's free disk space, which the Hub checks before accepting the upload; an upload may contain at most 4096 files. Public Git dependencies retain their authored repository and revision selectors; Studio does not replace them with private workspace paths. When moving to a public release, align SDK Git pins throughout the graph.
- `rust.build.upload-files`: Each uploaded crate contributes exactly the files `cargo package --list` names for it: its `include` and `exclude` settings and Git ignore rules apply, so an ignored private file never leaves the machine, and deploying a Rust App needs a local Cargo. Studio leaves out a crate's `translations` directory and never follows a symbolic link; when Cargo would package one, deployment stops and names it. Keep every build input in Cargo's package list.
- `rust.build.libraries-not-apps` (judgment): Ordinary libraries have no separate deployment queue entry or generated App wrapper. Build the executable App and let Cargo build its dependencies naturally. A TypesOnly App installs schema and documents without a native executable build. A Libertas-style Cargo package name alone does not make a library an App.

### Declaring native build packages

Owner: [rust.md#declaring-native-build-packages](../reference/languages/rust.md#declaring-native-build-packages)

- `rust.build.declare-packages`: Declare packages required on every supported Hub under `package.metadata.libertas.build.packages`. Add target rules when a package is specific to a Libertas distribution, base distribution, base version, or architecture:

  ```toml
  [package.metadata.libertas.build]
  packages = ["pkg-config", "libssl-dev"]

  [[package.metadata.libertas.build.targets]]
  distribution = "resolute"
  architecture = "arm64"
  packages = ["protobuf-compiler"]

  [[package.metadata.libertas.build.targets]]
  base-distribution = "ubuntu"
  base-version = "26.04"
  architecture = "x86_64"
  packages = ["libclang-dev"]
  ```

- `rust.build.target-rules`: Every selector in a target rule must match the compiling Hub. All matching rule package arrays are unioned with the common package array, then validated, deduplicated, and sorted. A target rule must contain at least one selector and a non-empty `packages` array. Package names use the Hub's restricted Debian-name grammar; protected operating-system packages are rejected. A `TypesOnly` package is not compiled, so its build-package metadata is not evaluated.
- `rust.build.no-package-guessing`: Build-package installation requires Hub super-user permission. The Hub never guesses a package from compiler error text and never installs an undeclared package automatically. APT resolves declared names to available versions, virtual-package providers, and transitive dependencies; it does not determine which Ubuntu package a Rust project needs. If native code needs an undeclared header or tool, Cargo fails normally and the developer must correct `Cargo.toml`.

### Sending workflow edits

Owner: [rust.md#sending-workflow-edits](../reference/languages/rust.md#sending-workflow-edits)

- `rust.workflow.edit-request-helper`: Use `libertas_endpoint_edit_request(endpoint, &request, trans_id, peer)` to reply with a complete Request value for editing. Pass the transaction and opaque peer from the incoming operation unchanged. The client installs the payload directly and waits for the user to submit. Cookie responses instead use the ordinary `libertas_endpoint_response` helper and `#[libertas_workflow_request]`. See the [workflow contract](../reference/protocols.md#workflow-interactions).

### Folding and inline choices

Owner: [rust.md#folding-and-inline-choices](../reference/languages/rust.md#folding-and-inline-choices)

- `rust.presentation.foldable`: `#[libertas_foldable]` permits manual folding on a struct, union, or list type or field; `#[libertas_foldable(initially_open = false)]` starts it collapsed.

### Sending workflow edits

Owner: [rust.md#sending-workflow-edits](../reference/languages/rust.md#sending-workflow-edits)

- `rust.presentation.inline-choices`: `#[libertas_inline_choices]`

### Backend-constrained choices

Owner: [rust.md#backend-constrained-choices](../reference/languages/rust.md#backend-constrained-choices)

- `rust.choices.constrained-by`: `#[libertas_constrained_by("$.allowed_choices")]`
- `rust.protocol.default-request`: Default endpoint entry uses `#[libertas_default_request]`
- `rust.protocol.action-control`: For additional response controls, put `#[libertas_action_control("sections")]` and `#[libertas_hidden]`

### Union member identity and payload constraints

Owner: [rust.md#union-member-identity-and-payload-constraints](../reference/languages/rust.md#union-member-identity-and-payload-constraints)

- `rust.union.discriminators`: Rust payload enums emit an explicit numeric `Discriminator` on every Union member from the native encoded variant index, consecutively from zero and in that same Children order. A single unnamed payload remains one schema member named after the variant: `VariantA(u32)` needs no fabricated child field. `#[libertas_fixed(7)]` constrains that payload independently of its wire discriminator. No new discriminator macro is required. Rust never emits `UnionTagField`; that marker belongs exclusively to ScriptLanguage native tag fields. See [the Union schema contract](../reference/schema.md#54-unions).
- `rust.workflow.cache-identity`: For large workflow request payloads, `#[libertas_cache_identity]`

### AI Search guidance

Owner: [rust.md#ai-search-guidance](../reference/languages/rust.md#ai-search-guidance)

- `rust.ai.navigation-prompt`: Use `#[libertas_ai_navigation_prompt("NAVIGATION_PROMPT")]` on a protocol Union or page variant to reference a package StringResource.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Project contract and library requirements

Owner: [rust.md#project-contract-and-library-requirements](../reference/languages/rust.md#project-contract-and-library-requirements)

- `rust.project.library-target`: A Libertas application is a Cargo library rooted at `src/lib.rs`; `src/main.rs` is not accepted

### Schema discovery and shape rules

Owner: [rust.md#schema-discovery-and-shape-rules](../reference/languages/rust.md#schema-discovery-and-shape-rules)

- `rust.discovery.types-only-roots`: An included graph cannot reach an ignored Rust type
- `rust.shape.public-and-derives`: A schema struct or enum is public and derives `LibertasAvroDecode`, normally with `LibertasExport`
- `rust.shape.derive-paths`: Derives use imported names or canonical `libertas::`/`libertas_macros::` paths; an unrelated crate's derive does not count
- `rust.shape.named-fields`: Structs have named fields; a tuple struct is rejected
- `rust.shape.public-aliases`: Published type aliases are public
- `rust.exports.exact-name`: Every published named type is defined or re-exported from the root `lib.rs` under its exact declaration name

### Type reference resolution

Owner: [rust.md#type-reference-resolution](../reference/languages/rust.md#type-reference-resolution)

- `rust.scope.native-resolution`: Type references resolve in the scope where they are written; a dependency is explored only through a native path
- `rust.scope.root-exports-not-imports`: Root exports do not import names into child modules; an alias may be local, but the root exports the original name

### Scope rules and errors

Owner: [rust.md#scope-rules-and-errors](../reference/languages/rust.md#scope-rules-and-errors)

- `rust.scope.unresolved-errors`: Nested modules, `self::`/`super::`, `#[path]` and re-export chains follow the same scope rules; missing, ambiguous or cyclic bindings are errors

### Built in type names

Owner: [rust.md#built-in-type-names](../reference/languages/rust.md#built-in-type-names)

- `rust.types.built-in-names`: Built-in names are unqualified after import or qualified exactly as `libertas::LibertasDevice`; another module prefix is rejected

### Enum and variant documentation

Owner: [rust.md#enum-and-variant-documentation](../reference/languages/rust.md#enum-and-variant-documentation)

- `rust.union.native-variants`: Rust enums identify their active variant: no discriminator field, custom tag or untagged layout

### Schema attributes and field presentation

Owner: [rust.md#schema-attributes-and-field-presentation](../reference/languages/rust.md#schema-attributes-and-field-presentation)

- `rust.attr.identifier-arguments`: `libertas_permissions` takes exactly one positional identifier; `libertas_string_resources` takes one or more

### General field attributes

Owner: [rust.md#general-field-attributes](../reference/languages/rust.md#general-field-attributes)

- `rust.attr.default-null`: `#[libertas_default($null)]` is the null default; unquoted `None` on `Option<T>` is a parser error

### Numeric modifiers

Owner: [rust.md#numeric-modifiers](../reference/languages/rust.md#numeric-modifiers)

- `rust.number.attribute`: Rust attribute source spelling
- `rust.number.money`: Rust money source spelling
- `rust.number.format`: Rust format source spelling

### Bitflag catalogs and integer fields

Owner: [rust.md#bitflag-catalogs-and-integer-fields](../reference/languages/rust.md#bitflag-catalogs-and-integer-fields)

- `rust.bitflags.external-catalog`: An external bitflag catalog may stay private but is declared under its exact name at the owning App's root `lib.rs`

### String patterns

Owner: [rust.md#string-patterns](../reference/languages/rust.md#string-patterns)

- `rust.pattern.attribute`: Rust attribute source spelling

### Constant references

Owner: [rust.md#constant-references](../reference/languages/rust.md#constant-references)

- `rust.const.where`: A value-attribute constant reference is the whole value
- `rust.const.which`: A bare `$NAME` finds one crate `const`; ambiguity or an unknown name is an error; `$module::NAME` qualifies
- `rust.const.what`: A constant is an integer, float or string literal, a negation, `a + b`, `a - b`, `a * b`, `a / b` or parentheses; other forms are errors
- `rust.const.reserved-dollar`: Only a whole unquoted `$NAME` is a constant reference in `libertas_pattern`; elsewhere a stray `$` other than `$null` is an error

### Persistent data schemas

Owner: [rust.md#persistent-data-schemas](../reference/languages/rust.md#persistent-data-schemas)

- `rust.data.codec-derives`: Key roots derive `LibertasKeyCodec`, key components `LibertasKeyFieldCodec`; values derive `LibertasAvroEncode` and `LibertasAvroDecode`
- `rust.data.key-wrappers`: Keys reject `Box<T>` and `Option<Option<T>>`; a single `Option<T>` is Nullable

### Permissions

Owner: [rust.md#permissions](../reference/languages/rust.md#permissions)

- `rust.permissions.attribute`: `#[libertas_permissions(CONST)]` on a function names a `const` string array

### String resources

Owner: [rust.md#string-resources](../reference/languages/rust.md#string-resources)

- `rust.string-resources.types-only`: A `TypesOnly` package names one non-empty literal `const` or `static` tuple slice with unique non-empty keys

### Protocol roles and context

Owner: [rust.md#protocol-roles-and-context](../reference/languages/rust.md#protocol-roles-and-context)

- `rust.link.attribute`: Rust attribute source spelling

### Published type ownership

Owner: [rust.md#published-type-ownership](../reference/languages/rust.md#published-type-ownership)

- `rust.exports.external-ownership`: An external type is emitted as its owner's `developer-package::TypeName`; it must be bound in Rust scope, never guessed

<!-- /generated -->
