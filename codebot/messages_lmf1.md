# Writing messages (LMF1)

Load this card when a task adds or changes a `StringResources` template, a
notification, a `FormattedText` value, a data name or a formatted chart label,
or translates resources. Studio checks the catalog's keys and the
`FormattedText` host, but it does not parse LMF1 templates or their arguments:
this card is the only place a CodeBot meets those rules. They are generated
from the [Libertas Message Format](../reference/message-format.md) and the
[App contract](../reference/app-contract.md#documentation-and-localization),
which own them. The [formatted output card](formatted_output.md) carries the
[presentation rules](../reference/presentation.md#static-read-only-placement-and-request-values)
for where a `FormattedText` node may appear.

## Order of work

1. Declare every key in the one package catalog (`lmf.resources.*`).
2. Write each template with zero-based positional arguments and MF1 syntax
   (`lmf.syntax.*`, `lmf.styles.*`, `lmf.libertas-enum.*`, `lmf.types.*`).
3. Encode each argument with its most specific type and pass the canonical
   bytes (`lmf.arguments.*`, `lmf.currency.*`, `lmf.temporal.*`,
   `lmf.tuple.*`, `lmf.formatted-text.*`).
4. Apply the rules of each use (`lmf.notifications.*`, `lmf.charts.*`,
   `lmf.data-names.*`) and preserve the structure in translation
   (`lmf.translation.*`).
5. Review every template and locale (`lmf.review.*`).

<!-- generated:codebot-rules task=messages-lmf1 -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Documentation and localization

Owner: [app-contract.md#documentation-and-localization](../reference/app-contract.md#documentation-and-localization)

- `lmf.resources.stable-keys`: String-resource keys are stable identifiers and are never translated. Values are translated as LMF1, preserving the structure the [message-format contract](../reference/message-format.md#lmf1-argument-syntax) defines.

### Libertas Message Format

Owner: [message-format.md#libertas-message-format](../reference/message-format.md#libertas-message-format)

- `lmf.tuple.identity`: Preserve the tuple; never replace its identity with pre-rendered text.

### Uses and storage

Owner: [message-format.md#uses-and-storage](../reference/message-format.md#uses-and-storage)

- `lmf.formatted-text.source-markers`: Use `@LibertasFormattedText` in TypeScript or `#[libertas_formatted_text]` in Rust, without arguments. See the [TypeScript marker](../reference/languages/typescript.md#formatted-runtime-text) and [Rust marker](../reference/languages/rust.md#formatted-runtime-text).
- `lmf.formatted-text.canonical-tuple`: The Bytes payload is always the canonical tuple, even when it contains only a resource key and an empty argument list. Payloads never contain templates.
- `lmf.formatted-text.no-base64-string`: A generic JSON boundary may represent those bytes as Base64 as part of its normal Bytes codec, but that transport spelling is not an App-authored String value and must not be copied into one.
- `lmf.resources.catalog-list`: The package declaration names one or more **catalog constants**, such as `//! #[libertas_string_resources(FEDERAL_STRINGS, CREDIT_STRINGS)]`, so a large App can keep its templates in several files. The package catalog is their union: one key space, emitted sorted, so splitting, merging or reordering the constants never changes the schema, the document or a revision hash. One name remains the simplest form.
- `lmf.resources.extraction-only`: **`StringResources` is for schema and document extraction only.** The parser reads the catalog constants from source whatever the build configuration, and clients and the Hub render every message from the package document. For a message, App runtime code therefore emits its resource key and typed arguments only: it never reads, compares, renders or sends the text, and the built App contains no message text, neither a catalog nor a copy of one of its sentences. Keep every catalog out of normal builds: in Rust a module gated out, such as `#[cfg(test)] mod strings;`, and in TypeScript a module no runtime module imports. Tests may read the catalog. App code names a resource by its key, a literal or one built at run time; a key built at run time is checked by the App's tests, since neither the compiler nor Studio can see it.
- `lmf.resources.catalog-per-module` (judgment): **Organizing a large catalog.** Keep one catalog constant per source module or folder, next to the code that emits its keys, and one shared catalog for keys several modules use; list them all in the package declaration. A change to one area then edits only that area's catalog, so parallel changes to different areas never meet in one file, and each string stays beside its code. Place each key by the code that references it: one module's key goes in that module's catalog, a key used by several goes in the shared one. Avoid a single catalog folder apart from the code, which separates every string from its use and becomes a shared file again. In Rust:

  ```rust
  //! #[libertas_string_resources(CORE_STRINGS, CREDIT_STRINGS, RETIREMENT_STRINGS)]

  // src/credits/mod.rs
  #[cfg(test)]
  mod strings; // pub const CREDIT_STRINGS: &[(&str, &str)] = &[...];
  ```

- `lmf.formatted-text.pass-bytes`: Pass those bytes directly; do not Base64-wrap them in App code. App code ordinarily does not decode formatted text: a client resolves and renders it at the final presentation boundary.

### LMF1 argument syntax

Owner: [message-format.md#lmf1-argument-syntax](../reference/message-format.md#lmf1-argument-syntax)

- `lmf.syntax.zero-based`: Every argument identifier is a zero-based position. Named arguments and one-based printf numbering are not LMF1.
- `lmf.syntax.no-printf`: For a resource that displays one literal-text argument, use `"{0}"`, not `"%1$s"` or `"%s"`. Printf placeholders are ordinary text in LMF1 and appear unchanged on screen. The same syntax applies to every locale's `StringResources`, including translated resources. A `LiteralText` argument is inserted as text; braces or percent signs inside it are not parsed again.
- `lmf.syntax.mf1-features`: LMF1 retains MF1 apostrophe quoting, plural `offset:`, exact plural selectors such as `=0`, required `other` branches, and `#` substitution inside plural variants.
- `lmf.translation.preserve-structure`: Translators may reorder positional arguments and translate literal text inside variants. They must preserve argument positions, types, styles, selector keys, plural offsets, braces, and apostrophe escaping.

### Libertas extensions

Owner: [message-format.md#libertas-extensions](../reference/message-format.md#libertas-extensions)

- `lmf.libertas-enum.style`: `LibertasEnum` carries an integer value. Its `style` is required and is the actual Enumeration type name used to resolve the localized variant. Other Libertas object types, including `LibertasDevice` and `LibertasUser`, resolve through the client's ordinary permission-aware object resolver.
- `lmf.types.encoded-wins`: The encoded Libertas Message Argument type has priority over a conflicting template type. This overlap with MF1 is intentional: message data remains self-describing even when a translation is stale or wrong. For example, an encoded device ID must still resolve as a device when a template mistakenly requests `number`; an encoded physical-unit value keeps its unit while the LMF1 number style controls numeric presentation.
- `lmf.types.compatible-template`: The template type supplies semantics only when compatible with the encoded argument. `LibertasEnum` may refine an integer-backed enum argument with the Enumeration name in `style`; it must not reinterpret an argument encoded as a device, literal text, resource text, or physical unit.

### Number and date/time styles

Owner: [message-format.md#number-and-datetime-styles](../reference/message-format.md#number-and-datetime-styles)

- `lmf.styles.authoritative`: The MF1 `style` on `number`, `date`, and `time` is authoritative presentation metadata. There is no parallel Libertas syntax for specifying a message's number or date format. In particular, schema `Format` attributes describe ordinary scalar fields and do not override an LMF1 argument style.
- `lmf.styles.unit-arguments`: Unit-qualified encoded arguments use the same number style while retaining their encoded physical unit and client-side conversion policy. Do not append a hard-coded unit in the template.

### Libertas Message Arguments

Owner: [message-format.md#libertas-message-arguments](../reference/message-format.md#libertas-message-arguments)

- `lmf.arguments.specific-type`: Use the most specific semantic type available. Encoding a `LibertasDevice` ID takes the same payload space as encoding a plain integer of the same width; developers must encode it as the device ID rather than erase its meaning.
- `lmf.arguments.nesting-limits`: Clients accept at most 16 nested FormattedText levels and 4,096 encoded bytes per tuple.
- `lmf.currency.code-and-scale`: The code is three uppercase ASCII letters, and the scale is 0 through 6, matching schema `Money`.
- `lmf.currency.pattern-restrictions`: Currency patterns cannot use percent, per-mille, scientific notation, or currency-sign sequences wider than three. Numeric styles without an unquoted currency sign, including `integer`, `percent` and `0.00`, retain the default ISO-code currency presentation. A conflicting template type ignores its style. Neither a style nor the locale may replace the encoded currency or scale. The renderer must not round through a floating-point value. The encoded currency semantics take precedence over a conflicting template type.
- `lmf.currency.not-selector`: Currency is not a plural or select operand; use a separate count argument when selection is needed.

### Temporal arguments

Owner: [message-format.md#temporal-arguments](../reference/message-format.md#temporal-arguments)

- `lmf.temporal.dateonly-range`:

  | Argument | Tag | Avro payload | Meaning and accepted values |
  | --- | --- | --- | --- |
  | `DateOnly` | 15 | Long | Valid Gregorian calendar date in `yyyyMMdd` form, years 1000 through 9999 |

- `lmf.temporal.datetime-range`:

  | Argument | Tag | Avro payload | Meaning and accepted values |
  | --- | --- | --- | --- |
  | `DateTime` | 16 | Long | Unsigned Unix epoch seconds, from 0 through 253402214399 |

- `lmf.temporal.timeonly-range`:

  | Argument | Tag | Avro payload | Meaning and accepted values |
  | --- | --- | --- | --- |
  | `TimeOnly` | 17 | Long | Seconds within the civil day, from 0 through 86400 inclusive |

- `lmf.temporal.datetime-seconds`: `DateTime` uses the SDK's existing unsigned-integer encoding in the Long payload; its accepted range fits a positive signed Long. The payload is seconds, not the millisecond convention used by some native formatting libraries.
- `lmf.temporal.combined-instant` (judgment): For a combined instant, compose its date and time components in the same template, for example `{0,date,long} at {0,time,short}`. Both refer to the same encoded instant and rendering timezone; the localized template owns the connecting words. Long and full DateTime time styles include the timezone; TimeOnly has no timezone name at any width. Short time omits seconds and the other widths include them, using the locale's clock convention.
- `lmf.temporal.not-selector`: Temporal arguments cannot serve as numeric plural, choice, or select operands. Reject invalid dates or out-of-range temporal payloads even when the template does not display the argument.
- `lmf.temporal.not-physical-unit`: PhysicalUnit arguments represent measured quantities, including durations in CLDR units. They do not establish a calendar date, an epoch origin, or a civil clock time; use these temporal argument types for those values.

### Notifications

Owner: [message-format.md#notifications](../reference/message-format.md#notifications)

- `lmf.notifications.key-and-values`: Send a stable resource key and typed values. Recipients, importance, and source are delivery metadata rather than LMF1 arguments:

  ```rust
  libertas_notification_send(
      &[recipient],
      NotificationImportance::Info,
      Some(room),
      "ROOM_STATUS",
      &arguments,
  );
  ```

- `lmf.notifications.literal-diagnostics` (judgment): Use literal notifications only for genuinely non-localizable diagnostics. Do not put sensitive identifiers into literal text when a typed object argument provides permission-aware presentation.

### Runtime fields with `FormattedText=true`

Owner: [message-format.md#runtime-fields-with-formattedtexttrue](../reference/message-format.md#runtime-fields-with-formattedtexttrue)

- `lmf.formatted-text.structured-authoritative` (judgment): Structured fields remain authoritative for automation, validation, controls, and chart geometry. Formatted fields follow the [static read-only placement and request-value contract](../reference/presentation.md#static-read-only-placement-and-request-values):

  ```rust
  //! Room condition messages
  //! Gives occupants localized summaries of current room conditions.
  //! #[libertas_string_resources(PROTOCOL_STRINGS)]

  pub const PROTOCOL_STRINGS: &[(&str, &str)] = &[
      ("READY", "Room condition reporting is ready."),
      ("ROOM_STATUS", "{0,LibertasDevice} reports a room temperature of {1,number,0.0}."),
  ];

  /// Room condition
  /// Describes the latest reported condition in one room.
  #[derive(LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
  pub struct RoomState {
      /// Availability summary
      /// Tells occupants whether the room condition report is ready.
      #[libertas_read_only]
      #[libertas_formatted_text]
      pub summary: Vec<u8>,

      /// Current room condition
      /// Tells occupants what is happening in the room and includes its relevant measurements.
      #[libertas_read_only]
      #[libertas_formatted_text]
      pub status: Vec<u8>,
  }

  fn encode_room_state(room: LibertasDevice, temperature_celsius: f64) -> RoomState {
      let status_arguments = [
          LibertasMessageArgument::Object(room),
          LibertasMessageArgument::UnitDouble {
              unit_type: "temperature-celsius",
              value: temperature_celsius,
          },
      ];
      RoomState {
          summary: libertas_formatted_text("READY", &[]),
          status: libertas_formatted_text("ROOM_STATUS", &status_arguments),
      }
  }
  ```

### Charting

Owner: [message-format.md#charting](../reference/message-format.md#charting)

- `lmf.charts.blob-identity`: LMF1 affects only the label. The formatted Bytes blob remains the raw chart domain, grouping, ordering, shared-scale, and interaction identity. Two tuples that render to the same text remain distinct categories. A locale or template change may alter measurement and therefore guide layout, but must never alter chart geometry derived from data, domain membership, or category identity.

### Data and database names

Owner: [message-format.md#data-and-database-names](../reference/message-format.md#data-and-database-names)

- `lmf.data-names.translation-keeps-keys`: Translation must not change stored keys or values read by business logic.
- `lmf.data-names.not-storage-keys`: Such names are formatted payloads, not storage keys; the KV API uses only the declared binary key for record identity.

### Failure handling and review checklist

Owner: [message-format.md#failure-handling-and-review-checklist](../reference/message-format.md#failure-handling-and-review-checklist)

- `lmf.review.declare-keys`: declare every notification, data-name, permission-reason, and formatted-output key in the package `StringResources` catalog;
- `lmf.review.stable-keys-templates` (judgment): use stable resource keys and realistic default-locale LMF1 templates;
- `lmf.review.verify-locales` (judgment): use zero-based positions and verify every locale against the same typed argument list;
- `lmf.review.trusted-templates`: keep templates trusted—never treat user input as an LMF1 template; and
- `lmf.review.test-cases` (judgment): test reordered arguments, apostrophe escaping, number/date styles, plural/select/selectordinal/choice branches, missing resources, unresolved Libertas values, and malformed formatted-text bytes.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Uses and storage

Owner: [message-format.md#uses-and-storage](../reference/message-format.md#uses-and-storage)

- `lmf.formatted-text.bytes-host`: FormattedText=true requires a Bytes host
- `lmf.resources.one-catalog`: One App-package StringResources catalog owns stable schema keys and localized templates; functions and types carry no resource catalogs
- `lmf.resources.key-sets-match`: Schema and default-document StringResources key sets match exactly; duplicate, missing or undeclared keys are errors

<!-- /generated -->
