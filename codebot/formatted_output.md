# Formatting values and text

Load this card when a task declares a typed or formatted value: a date, time
or duration type, `Money`, `PhysicalUnit`, `Bitflags`, a `Format` display
pattern, the package `StringResources` catalog, a `FormattedText` value, or a
translation of localized strings. Its rules are generated from the
[presentation contract](../reference/presentation.md), which owns them; the
[messages card](messages_lmf1.md) carries the LMF1 template and argument rules
of the [Libertas Message Format](../reference/message-format.md), the
[translation card](translation.md) the locale-document workflow, and the
[annotation inventory](schema_annotations.md) gives each attribute's Rust and
TypeScript spelling. Presentation metadata never
changes the underlying typed wire value.

## Order of work

1. Give each number its meaning: date and time types (`datetime.*`), exact
   amounts (`money.*`), units (`unit.*`), flag sets (`bitflags.*`).
2. Add a display pattern only where the locale default is wrong (`fmt.*`); the
   pattern compiles into the template's `format` hint, which the
   [Display Template card](display_template.md) may vary per device.
3. Declare the catalog and encode dynamic wording as `FormattedText`
   (`text.*`), with its static `ReadOnly` placement and request values.
4. Translate strings only, preserving every LMF1 structure (`translate.*`).
5. Compile with the Studio MCP `compile_schema` tool and fix every reported
   rule.

<!-- generated:codebot-rules task=formatted-output -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Date and time

Owner: [presentation.md#date-and-time](../reference/presentation.md#date-and-time)

- `datetime.types`:

  | Type | Raw meaning | Presentation |
  | --- | --- | --- |
  | `LibertasDateTime` | Epoch seconds | Viewer-local date/time; optional timezone in `Format`. |

- `datetime.date-only-calendar`: Never convert `DateOnly` through an epoch or timezone. Defaults, fixed values, requests, and runtime data must all be valid calendar dates.

### Money

Owner: [presentation.md#money](../reference/presentation.md#money)

- `money.no-conversion`: Currency identity does not introduce conversion: applications own exchange rates and must keep unrelated currencies separate. Money follows the [exclusions of numeric semantic attributes](../reference/schema.md#numeric-semantic-exclusions).

### Physical units

Owner: [presentation.md#physical-units](../reference/presentation.md#physical-units)

- `unit.translate-text-not-identifiers` (judgment): Translate field names, descriptions, chart titles, and other authored text through the App's locale documents, not by changing unit identifiers or appending translated unit strings to numeric values.
- `unit.no-conversion`: The identifier states the provided unit and never performs conversion. `percent` raw `50` means fifty percent; it does not rescale `0.5`. `Format` may coexist and formats only the numeric portion. PhysicalUnit follows the [exclusions of numeric semantic attributes](../reference/schema.md#numeric-semantic-exclusions).

### Bitflags

Owner: [presentation.md#bitflags](../reference/presentation.md#bitflags)

- `bitflags.document-members` (judgment): Author the meaning and consequences of each flag in its member comments.

### AI tool-call output

Owner: [presentation.md#ai-tool-call-output](../reference/presentation.md#ai-tool-call-output)

- `text.document-purpose-not-templates` (judgment): Declare `FormattedText` and document the message's domain purpose; do not repeat templates or rendering instructions in field comments for the AI.

### Static read-only placement and request values

Owner: [presentation.md#static-read-only-placement-and-request-values](../reference/presentation.md#static-read-only-placement-and-request-values)

- `text.request-values`: Request and SubscriptionRequest fields may carry these display-only values. Their values come from `CopyFrom` or `CopyTo`, including a copy of an enclosing record or request root; supplied `EditRequest` values; or `Default`/`Fixed` initialization. A leaf does not need its own `CopyFrom` when one of those paths supplies it. Apply the [shared copy rules](../reference/schema.md#shared-copyfrom-and-copyto-rules) to copied values. The client never asks the user to edit a formatted tuple. An unanswered local field may remain undefined; a required undefined field blocks submission. Null is valid only where the destination permits it.

### Persisting formatted record payloads

Owner: [presentation.md#persisting-formatted-record-payloads](../reference/presentation.md#persisting-formatted-record-payloads)

- `text.persist-discouraged` (judgment): `FunctionDataSchema` record payloads may contain statically read-only `FormattedText`. Persistence is supported, but generally discouraged for these payload fields: `FormattedText` supplies runtime rendering, so prefer storing structured domain values and constructing the formatted value at presentation time. This is design guidance, not a schema rejection rule.

### Declaring and encoding formatted values

Owner: [presentation.md#declaring-and-encoding-formatted-values](../reference/presentation.md#declaring-and-encoding-formatted-values)

- `text.enum-arguments-typed` (judgment): For `ROOM_STATUS`, encode argument 0 as the `LibertasDevice` object and argument 1 as a unit-qualified temperature number. Do not pass a translated state as a literal or resource-text grammar fragment. A closed Enumeration instead uses `{1,LibertasEnum,ActualEnumType}` and an integer argument encoded with that Enumeration's semantic type.
- `formatted.encoding.no-extra-base64`: FormattedText always uses the public SDK's canonical Bytes encoding of a resource key plus ordered typed Libertas Message Arguments, including an empty argument list. Do not Base64-wrap it in App code. A String host would only encode the tuple and force the renderer to decode it again, providing no benefit; Bytes also keeps a sharp type distinction from literal String data and prevents authoring mistakes. A generic JSON byte codec may use Base64 as a transport spelling without changing the field's Bytes type. Runtime payloads never carry format templates.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Display formats

Owner: [presentation.md#display-formats](../reference/presentation.md#display-formats)

- `fmt.one-portable-pattern`: `Format` is one portable Unicode LDML pattern that never changes parsing, storage or chart geometry; it compiles into the template `format` hint
- `fmt.number-grammar`: Portable number grammar: `0 # . , ;` subpatterns, `%`, `‰`, `E`, affixes, apostrophe quoting; no `@`, `*`, wider currency tokens or rounding increments
- `fmt.currency-needs-money`: `¤` and `¤¤` require explicit `Money` metadata
- `fmt.date-time-fields`: Date fields `G y M d E`, time fields `H h m s S a`, `z` only for DateTime; DateOnly date fields only, TimeOnly time fields only, never `H` with `h`

### Money

Owner: [presentation.md#money](../reference/presentation.md#money)

- `money.i64-minor-units`: `Money` goes on a signed `i64` Number storing minor units; the currency code and scale are schema metadata
- `money.code-and-scale`: The currency code is three uppercase letters and the scale 0 through 6; choose the currency's minor-unit scale
- `money.currency-pattern`: A currency pattern controls presentation only, rounds half up for display, and cannot use percent, per-mille or scientific notation
- `money.bounds-in-minor-units`: Money `Default`, `Fixed`, `NumberMin`, `NumberMax` and a positive `NumberStep` are integer minor units; bounds and rules operate on minor units

### Bitflags

Owner: [presentation.md#bitflags](../reference/presentation.md#bitflags)

- `bitflags.single-bits`: Each nonzero Bitflags member is one unique bit, one zero label is allowed, and values combine only declared bits

### Static read-only placement and request values

Owner: [presentation.md#static-read-only-placement-and-request-values](../reference/presentation.md#static-read-only-placement-and-request-values)

- `text.static-readonly`: Every FormattedText node is statically ReadOnly: on the field, its type, or an enclosing read-only Table (a ReadOnly List does not count)
- `text.not-in-function-arguments`: FormattedText is forbidden in function and configuration arguments, even when read-only

<!-- /generated -->
