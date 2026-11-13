# Libertas Message Format

<!-- rule: lmf.renderer.libertas-aware class=hard enforced=no tasks=client-runtime -->
Libertas Message Format (LMF) is the platform standard for localized,
structured messages. LMF version 1 (LMF1) is based on ICU MessageFormat 1
(MF1) and adds Libertas semantic types. An LMF1 renderer must therefore be a
Libertas-aware implementation; substituting a platform ICU formatter is not
sufficient.

LMF is used for notifications, Endpoint output marked with `FormattedText`,
human-readable data/database names, and formatted chart labels:

```text
message tuple = resource key + ordered, typed Libertas Message Arguments
invariant blob = canonical_encode(message tuple)
rendered text  = LMF1(localized template, resolved arguments)
```

The canonical tuple is locale-independent and is suitable for transport,
storage, chart identity, or an object key. Rendered text is a locale-specific
view for people and AI systems. <!-- rule: lmf.tuple.identity class=hard enforced=no tasks=messages-lmf1 --> Preserve the tuple; never replace its identity
with pre-rendered text.

## Uses and storage

| Use | Stored or transported value | Additional behavior |
| --- | --- | --- |
| Notification | Resource key and typed arguments inside a delivery envelope | Adds recipients, importance, and source. |
| `FormattedText` `Bytes` | Canonical encoding of one resource key and ordered typed arguments, including an empty argument list | Runtime output presentation; the emitting App package catalog supplies localization. |
| Data name | Invariant blob containing the canonical resource key and typed arguments | The blob is the single-record key or indexed-database/table name. |
| Chart value | The same formatted Bytes tuple | Rendered only as presentation; the raw blob remains category and interaction identity. |

<!-- rule: lmf.formatted-text.bytes-host class=hard enforced=yes tasks=messages-lmf1 title="FormattedText=true requires a Bytes host" -->
`FormattedText=true` requires a `Bytes` host<!-- /rule --> <!-- rule: lmf.formatted-text.wire-unchanged class=hard enforced=no tasks=client-runtime --> and does not change its wire
serialization. <!-- /rule --> <!-- rule: lmf.formatted-text.source-markers class=hard enforced=no tasks=messages-lmf1 --> Use `@LibertasFormattedText` in TypeScript or
`#[libertas_formatted_text]` in Rust, without arguments. See the
[TypeScript marker](languages/typescript.md#formatted-runtime-text) and
[Rust marker](languages/rust.md#formatted-runtime-text). <!-- /rule --> <!-- rule: lmf.formatted-text.canonical-tuple class=hard enforced=no tasks=messages-lmf1 --> The Bytes payload is
always the canonical tuple, even when it contains only a resource key and an
empty argument list. Payloads never contain templates.

A formatted String host would have to Base64-encode the tuple and then decode
it again before every render. That back-and-forth adds no information, storage
benefit, localization capability, or identity guarantee, while making an
encoded tuple look like ordinary literal text. Libertas therefore keeps the
runtime value typed as Bytes end to end. That sharp type distinction makes it
less likely that developers will accidentally mix formatted values with
literal Strings. <!-- rule: lmf.formatted-text.no-base64-string class=hard enforced=no tasks=messages-lmf1 --> A generic JSON boundary may represent those bytes as Base64
as part of its normal Bytes codec, but that transport spelling is not an
App-authored String value and must not be copied into one.

Formatted fields follow the [static read-only placement and request-value
contract](presentation.md#static-read-only-placement-and-request-values).
That owner defines declaration requirements and permitted runtime uses;
the byte encoding here does not grant an exception.

<!-- rule: lmf.resources.one-catalog class=hard enforced=yes tasks=messages-lmf1 title="One App-package StringResources catalog owns stable schema keys and localized templates; functions and types carry no resource catalogs" -->
`StringResources` is owned exactly once by the App package. The schema package
attribute contains its stable, comma-delimited keys, and the package document's
top-level `StringResources` object contains the localized LMF1 templates.
Functions and types do not carry resource catalogs. Notifications, data names,
permission reasons, `FormattedText`, and formatted chart labels emitted by any
function in the package all resolve through this one catalog. The encoded tuple
therefore needs no function or type owner and never carries a resource FQN.

<!-- rule: lmf.resources.catalog-list class=hard enforced=no tasks=messages-lmf1 title="The package StringResources declaration names one or more catalog constants; the catalog is their union" -->
The package declaration names one or more **catalog constants**, such as
`//! #[libertas_string_resources(FEDERAL_STRINGS, CREDIT_STRINGS)]`, so a large
App can keep its templates in several files. The package catalog is their
union: one key space, emitted sorted, so splitting, merging or reordering the
constants never changes the schema, the document or a revision hash. One name
remains the simplest form.

<!-- rule: lmf.resources.extraction-only class=hard enforced=no tasks=messages-lmf1,app-design title="StringResources is for schema and document extraction only: App runtime code emits keys and typed arguments, and the built App contains no message text" -->
**`StringResources` is for schema and document extraction only.** The parser
reads the catalog constants from source whatever the build configuration, and
clients and the Hub render every message from the package document. For a
message, App runtime code therefore emits its resource key and typed arguments
only: it never reads, compares, renders or sends the text, and the built App
contains no message text, neither a catalog nor a copy of one of its
sentences. Keep every catalog out of normal builds: in Rust a module gated
out, such as `#[cfg(test)] mod strings;`, and in TypeScript a module no
runtime module imports. Tests may read the catalog. App code names a resource
by its key, a literal or one built at run time; a key built at run time is
checked by the App's tests, since neither the compiler nor Studio can see it.

<!-- rule: lmf.resources.catalog-per-module class=judgment enforced=no tasks=messages-lmf1 title="In a large App, keep one catalog per source module next to its code and a shared catalog for keys several modules use" -->
**Organizing a large catalog.** Keep one catalog constant per source module or
folder, next to the code that emits its keys, and one shared catalog for keys
several modules use; list them all in the package declaration. A change to one
area then edits only that area's catalog, so parallel changes to different
areas never meet in one file, and each string stays beside its code. Place
each key by the code that references it: one module's key goes in that
module's catalog, a key used by several goes in the shared one. Avoid a single
catalog folder apart from the code, which separates every string from its use
and becomes a shared file again. In Rust:

```rust
//! #[libertas_string_resources(CORE_STRINGS, CREDIT_STRINGS, RETIREMENT_STRINGS)]

// src/credits/mod.rs
#[cfg(test)]
mod strings; // pub const CREDIT_STRINGS: &[(&str, &str)] = &[...];
```

<!-- rule: lmf.resources.key-sets-match class=hard enforced=yes tasks=messages-lmf1 title="Schema and default-document StringResources key sets match exactly; duplicate, missing or undeclared keys are errors" -->
Studio requires the schema's declared key set and the package document's key
set to match exactly. Duplicate keys, within one catalog constant or across
several, duplicate package catalog declarations, missing document values, and
undeclared document keys are validation errors; a duplicate key reports the
source location of every occurrence.

The public SDK encoders `libertasFormattedText(resource, arguments)` in
TypeScript and `libertas_formatted_text(resource, arguments)` in Rust produce
the canonical `Uint8Array`/`Vec<u8>` representation for a Bytes field. <!-- rule: lmf.formatted-text.pass-bytes class=hard enforced=no tasks=messages-lmf1 --> Pass
those bytes directly; do not Base64-wrap them in App code. App code ordinarily
does not decode formatted text: a client resolves and renders it at the final
presentation boundary.

## LMF1 argument syntax

<!-- rule: lmf.syntax.zero-based class=hard enforced=no tasks=messages-lmf1 -->
Every argument identifier is a zero-based position. Named arguments and
one-based printf numbering are not LMF1. <!-- /rule --> The three basic forms are:

```text
{0}
{0,type}
{0,type,style}
```

<!-- rule: lmf.syntax.no-printf class=hard enforced=no tasks=messages-lmf1 -->
For a resource that displays one literal-text argument, use `"{0}"`, not
`"%1$s"` or `"%s"`. Printf placeholders are ordinary text in LMF1 and appear
unchanged on screen. The same syntax applies to every locale's
`StringResources`, including translated resources. A `LiteralText` argument
is inserted as text; braces or percent signs inside it are not parsed again.

The complete grammar summary is:

```text
Untyped:
    {argument}

Simple:
    {argument,number[,style]}
    {argument,date[,style]}
    {argument,time[,style]}
    {argument,spellout[,ruleset]}
    {argument,ordinal[,ruleset]}
    {argument,duration[,ruleset]}

Complex:
    {argument,plural,variants}
    {argument,selectordinal,variants}
    {argument,select,variants}
    {argument,choice,choices}
```

<!-- rule: lmf.syntax.mf1-features class=hard enforced=no tasks=messages-lmf1 -->
LMF1 retains MF1 apostrophe quoting, plural `offset:`, exact plural selectors
such as `=0`, required `other` branches, and `#` substitution inside plural
variants. <!-- /rule --> For example:

```typescript
const StatusStrings = {
    ROOM_TEMPERATURE: "{0,LibertasDevice} reports a room temperature of {1,number,0.0}.",
    ALERTS: "{0,plural,=0 {No alerts} one {# alert} other {# alerts}}",
    OWNER: "{0,select,male {His} female {Her} other {Their}} device"
} as const;
```

<!-- rule: lmf.translation.preserve-structure class=hard enforced=no tasks=messages-lmf1 -->
Translators may reorder positional arguments and translate literal text inside
variants. They must preserve argument positions, types, styles, selector keys,
plural offsets, braces, and apostrophe escaping.

## Libertas extensions

An LMF1 type may be a Libertas type:

```text
{0,LibertasDevice}
{1,LibertasEnum,StatusEnum}
```

<!-- rule: lmf.libertas-enum.style class=hard enforced=no tasks=messages-lmf1 -->
`LibertasEnum` carries an integer value. Its `style` is required and is the
actual Enumeration type name used to resolve the localized variant. Other
Libertas object types, including `LibertasDevice` and `LibertasUser`, resolve
through the client's ordinary permission-aware object resolver.

<!-- rule: lmf.types.encoded-wins class=hard enforced=no tasks=messages-lmf1 -->
The encoded Libertas Message Argument type has priority over a conflicting
template type. This overlap with MF1 is intentional: message data remains
self-describing even when a translation is stale or wrong. For example, an
encoded device ID must still resolve as a device when a template mistakenly
requests `number`; an encoded physical-unit value keeps its unit while the
LMF1 number style controls numeric presentation.

<!-- rule: lmf.types.compatible-template class=hard enforced=no tasks=messages-lmf1 -->
The template type supplies semantics only when compatible with the encoded
argument. `LibertasEnum` may refine an integer-backed enum argument with the
Enumeration name in `style`; it must not reinterpret an argument encoded as a
device, literal text, resource text, or physical unit.

## Number and date/time styles

<!-- rule: lmf.styles.authoritative class=hard enforced=no tasks=messages-lmf1 -->
The MF1 `style` on `number`, `date`, and `time` is authoritative presentation
metadata. There is no parallel Libertas syntax for specifying a message's
number or date format. In particular, schema `Format` attributes describe
ordinary scalar fields and do not override an LMF1 argument style.

Use standard MF1 styles or a portable decimal pattern:

```text
{0,number}          locale-default number
{0,number,integer}  integer presentation
{0,number,percent}  percent presentation
{0,number,0.00}     exactly two fractional digits
{0,date,long}       long localized date
{0,time,short}      short localized time
```

<!-- rule: lmf.styles.unit-arguments class=hard enforced=no tasks=messages-lmf1 -->
Unit-qualified encoded arguments use the same number style while retaining
their encoded physical unit and client-side conversion policy. Do not append a
hard-coded unit in the template.

## Libertas Message Arguments

Each formatted argument carries a one-byte argument-type discriminator for the
currently defined variants, followed by its value. <!-- rule: lmf.arguments.specific-type class=hard enforced=no tasks=messages-lmf1 --> Use the most specific
semantic type available. Encoding a `LibertasDevice` ID takes the same payload
space as encoding a plain integer of the same width; developers must encode it
as the device ID rather than erase its meaning.

Current argument categories include:

- literal text, which is inserted without resource lookup;
- resource text, which resolves an argument-free key in the same resource set;
- FormattedText (tag 13), which carries a complete canonical FormattedText
  Bytes tuple as one argument;
- exact currency (tag 14), which carries an ISO 4217 code, decimal scale, and
  signed integer minor-unit amount;
- DateOnly (tag 15), DateTime (tag 16), and TimeOnly (tag 17), carrying
  calendar dates, instants, and wall-clock times respectively;
- Libertas objects, resolved to an authorized display name;
- booleans, signed/unsigned integers, floats, and doubles;
- unit-qualified signed/unsigned integers, floats, and doubles; and
- plural counts.

<!-- rule: lmf.renderer.no-template-reparse class=hard enforced=no tasks=client-runtime -->
`FormattedText` is distinct from `ResourceText`: its Avro Bytes payload contains
the nested resource key **and its own typed arguments**. The client renders that
tuple with the enclosing message's package catalog, locale, and permission-aware
object resolver, then inserts the result as text. It never reparses the rendered
text as a template. The encoded argument type wins over a conflicting template
type, so a nested value is not treated as a number or selector. Malformed nested
tuples fail the containing render. <!-- rule: lmf.arguments.nesting-limits class=hard enforced=no tasks=messages-lmf1 --> Clients accept at most 16 nested
FormattedText levels and 4,096 encoded bytes per tuple.

Exact currency uses Avro String `currency_code`, Avro Long `scale`, and Avro
Long `minor_units`, in that order after tag 14. <!-- rule: lmf.currency.code-and-scale class=hard enforced=no tasks=messages-lmf1 --> The code is three uppercase
ASCII letters, and the scale is 0 through 6, matching schema `Money`. <!-- /rule --> The
represented amount is
`minor_units × 10^-scale`; for example, `USD`, scale `2`, and `-5` means
negative five cents. The renderer uses the viewer's locale for grouping and
sign placement, defaulting to the explicit ISO currency code and exactly the
encoded number of fractional digits. A compatible `number` placeholder may
choose the currency's symbol, ISO code, or localized name:

| Template | English result for USD, scale 2, minor units 676100 |
| --- | --- |
| `{0}` or `{0,number}` | `USD 6,761.00` |
| `{0,number,currency}` | `$6,761.00` |
| `{0,number,¤#,##0.00}` | `$6,761.00` |
| `{0,number,¤¤#,##0.00}` | `USD 6,761.00` |
| `{0,number,#,##0.00 ¤¤¤}` | `6,761.00 US dollars` |

Currency patterns use the portable decimal-pattern subset with `¤` for a
localized symbol, `¤¤` for the ISO code, and `¤¤¤` for the localized currency
name. The name follows the locale's plural rules with the encoded scale's
visible fractional digits. Quoted currency signs remain literals. Patterns
control grouping, placement, affixes and an optional negative subpattern;
for example, `¤#,##0.00;(¤#,##0.00)` displays negative USD amounts in
parentheses. The encoded scale always overrides the pattern's fractional
digits: `¤#,##0` still shows `$6,761.00` for this argument. Locale data supplies
currency spacing, including a nonbreaking separator between an adjacent
alphabetic code and digits. Symbols may include a regional qualifier, such as
`US$`, depending on the viewer's locale.

<!-- rule: lmf.currency.pattern-restrictions class=hard enforced=no tasks=messages-lmf1 -->
Currency patterns cannot use percent, per-mille, scientific notation, or
currency-sign sequences wider than three. Numeric styles without an unquoted
currency sign, including `integer`, `percent` and `0.00`, retain the default
ISO-code currency presentation. A conflicting template type ignores its style.
Neither a style nor the locale may replace the encoded currency or scale.
The renderer must not round through a floating-point value. The encoded currency semantics
take precedence over a conflicting template type. <!-- rule: lmf.currency.not-selector class=hard enforced=no tasks=messages-lmf1 --> Currency is not a plural or
select operand; use a separate count argument when selection is needed.

### Temporal arguments

Temporal arguments retain the corresponding schema value's meaning:

| Argument | Tag | Avro payload | Meaning and accepted values |
| --- | --- | --- | --- |
| <!-- rule: lmf.temporal.dateonly-range class=hard enforced=no tasks=messages-lmf1 --> `DateOnly` | 15 | Long | Valid Gregorian calendar date in `yyyyMMdd` form, years 1000 through 9999 |
| <!-- rule: lmf.temporal.datetime-range class=hard enforced=no tasks=messages-lmf1 --> `DateTime` | 16 | Long | Unsigned Unix epoch seconds, from 0 through 253402214399 |
| <!-- rule: lmf.temporal.timeonly-range class=hard enforced=no tasks=messages-lmf1 --> `TimeOnly` | 17 | Long | Seconds within the civil day, from 0 through 86400 inclusive |

<!-- rule: lmf.temporal.datetime-seconds class=hard enforced=no tasks=messages-lmf1 -->
`DateTime` uses the SDK's existing unsigned-integer encoding in the Long
payload; its accepted range fits a positive signed Long. The payload is seconds,
not the millisecond convention used by some native formatting libraries.

<!-- rule: lmf.timeonly.end-of-day class=hard enforced=no tasks=client-runtime -->
Render `DateOnly` as a localized Gregorian date without a timezone conversion.
Render `TimeOnly` as a localized clock value without a timezone conversion or a
timezone name. Its upper-bound value 86400 means end of day: display localized
`24:00` for `short`, and `24:00:00` for the other styles; do not wrap it to midnight.
Render `DateTime` in the consumer's rendering timezone, using Gregorian calendar
fields. A client normally uses the viewer's local timezone; a renderer without
an explicit timezone uses its runtime's local timezone.

The defaults are a medium date for DateOnly, a medium time for TimeOnly, and
medium date plus medium time for DateTime. Compatible LMF1 placeholders may
choose `short`, `medium`, `long`, or `full`: `date` for DateOnly, `time` for
TimeOnly, and either component for DateTime. For example,
`{0,date,long}` formats an encoded DateOnly or DateTime date; `{1,time,short}`
formats an encoded TimeOnly or DateTime time. An unrecognized temporal style
uses medium presentation. Custom LDML patterns are not part of this argument
style surface.

<!-- rule: lmf.temporal.combined-instant class=judgment enforced=no tasks=messages-lmf1 -->
For a combined instant, compose its date and time components in the same
template, for example `{0,date,long} at {0,time,short}`. Both refer to the same
encoded instant and rendering timezone; the localized template owns the
connecting words. Long and full DateTime time styles include the timezone;
TimeOnly has no timezone name at any width. Short time omits seconds and the
other widths include them, using the locale's clock convention.

The encoded semantic type takes precedence over conflicting template types.
A DateOnly requested as `number` still renders as a date, and a TimeOnly
requested as `date` still renders as a clock time. Such a conflict ignores the
incompatible type and its style. <!-- rule: lmf.temporal.not-selector class=hard enforced=no tasks=messages-lmf1 --> Temporal arguments cannot serve as numeric
plural, choice, or select operands. Reject invalid dates or out-of-range temporal
payloads even when the template does not display the argument.

<!-- rule: lmf.temporal.not-physical-unit class=hard enforced=no tasks=messages-lmf1 -->
PhysicalUnit arguments represent measured quantities, including durations in
CLDR units. They do not establish a calendar date, an epoch origin, or a civil
clock time; use these temporal argument types for those values.

The Rust SDK exposes the variants directly through the
`LibertasMessageArgument` enum. The core TypeScript SDK exposes a numeric
discriminated `LibertasMessageArgument` union; the optional
`@libertas-ai/consts` package supplies its named
`LibertasMessageArgumentType` constants.

For example:

```rust
let arguments = [
    LibertasMessageArgument::Object(room),
    LibertasMessageArgument::ResourceText("HVAC_HEATING"),
    LibertasMessageArgument::UnitFloat {
        unit_type: "temperature-celsius",
        value: 22.5,
    },
];

let detail = libertas_formatted_text(
    "ITEM_COUNT",
    &[LibertasMessageArgument::Plural(2)],
);
let summary = libertas_formatted_text(
    "SUMMARY",
    &[LibertasMessageArgument::formatted_text(&detail)],
);
let amount = libertas_formatted_text(
    "PAYMENT_AMOUNT",
    &[LibertasMessageArgument::Currency {
        currency_code: "USD",
        scale: 2,
        minor_units: -5,
    }],
);
```

The equivalent TypeScript argument uses the canonical Bytes result directly:

```typescript
const detail = libertasFormattedText("ITEM_COUNT", [{ type: 12, value: 2n }]);
const summary = libertasFormattedText("SUMMARY", [{ type: 13, value: detail }]);
const amount = libertasFormattedText("PAYMENT_AMOUNT", [
    { type: 14, currency_code: "USD", scale: 2n, value: -5n },
]);
```

The same array can render a notification, formatted runtime Bytes field, or
data name. The resource key and typed arguments—not the rendered sentence—are
the value's stable identity.

## Notifications

<!-- rule: lmf.notifications.key-and-values class=hard enforced=no tasks=messages-lmf1 -->
Send a stable resource key and typed values. Recipients, importance, and source
are delivery metadata rather than LMF1 arguments:

```rust
libertas_notification_send(
    &[recipient],
    NotificationImportance::Info,
    Some(room),
    "ROOM_STATUS",
    &arguments,
);
```

<!-- rule: lmf.notifications.literal-diagnostics class=judgment enforced=no tasks=messages-lmf1 -->
Use literal notifications only for genuinely non-localizable diagnostics. Do
not put sensitive identifiers into literal text when a typed object argument
provides permission-aware presentation.

## Runtime fields with `FormattedText=true`

<!-- rule: lmf.formatted-text.structured-authoritative class=judgment enforced=no tasks=messages-lmf1 -->
Structured fields remain authoritative for automation, validation, controls,
and chart geometry. Formatted fields follow the
[static read-only placement and request-value contract](presentation.md#static-read-only-placement-and-request-values):

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

<!-- rule: lmf.formatted.resolve-at-display class=hard enforced=no tasks=client-runtime -->
The client decodes both Bytes tuples before looking up `READY` or
`ROOM_STATUS`. It resolves arguments at display time and never parses a
rendered sentence back into data.

## Charting

Charting uses the same LMF1 renderer for `FormattedText` text marks, axes,
facets, legends, tooltips, and accessibility labels. The renderer uses the
catalog owned by the App package and the client's normal
object, Enumeration, unit, locale, and number/date services.

<!-- rule: lmf.charts.blob-identity class=hard enforced=no tasks=messages-lmf1 -->
LMF1 affects only the label. The formatted Bytes blob remains the raw chart
domain, grouping, ordering, shared-scale, and
interaction identity. Two tuples that render to the same text remain distinct
categories. A locale or template change may alter measurement and therefore
guide layout, but must never alter chart geometry derived from data, domain
membership, or category identity.

<!-- rule: lmf.chart.no-diagnostic-text class=hard enforced=no tasks=client-runtime -->
An unresolved or malformed optional formatted chart value is omitted. If the
value is essential to distinguish visible categories or supply an accessible
name, the complete chart follows the
[chart failure policy](charts/2d.md#10-validation-and-fallback). Raw keys, blobs, and
typed diagnostics may appear in bounded developer diagnostics, never as chart
presentation text.

## Data and database names

The transactional Data API uses [schema-defined binary keys](protocols.md#transactional-key-value-api)
for identity. A human-readable name can use normal `FormattedText` and the
App's package-owned StringResources, independently of that binary key. <!-- rule: lmf.data-names.translation-keeps-keys class=hard enforced=no tasks=messages-lmf1 -->
Translation must not change stored keys or values read by business logic.

`DataName` formatted values carry a package resource key and ordered typed
message arguments encoded together as Avro bytes. Render the label using the
App's catalog. <!-- rule: lmf.data-names.not-storage-keys class=hard enforced=no tasks=messages-lmf1 --> Such names are formatted payloads, not storage keys; the KV API
uses only the declared binary key for record identity.

## Failure handling and review checklist

<!-- rule: lmf.failure.preserve-canonical-value class=hard enforced=no tasks=client-runtime -->
Rendering must never mutate canonical data. If a template, resource, object,
Enumeration, unit, argument, or style cannot be resolved, retain the encoded
value and use the established safe fallback outside chart presentation.

Before publishing an App:

<!-- rule: lmf.review.declare-keys class=hard enforced=no tasks=messages-lmf1 -->
- declare every notification, data-name, permission-reason, and formatted-output
  key in the package `StringResources` catalog; <!-- rule: lmf.review.stable-keys-templates class=judgment enforced=no tasks=messages-lmf1 -->
- use stable resource keys and realistic default-locale LMF1 templates; <!-- rule: lmf.review.verify-locales class=judgment enforced=no tasks=messages-lmf1 -->
- use zero-based positions and verify every locale against the same typed
  argument list;
- preserve all LMF1 structure during translation;
- use Libertas object/enum and physical-unit arguments instead of erasing them
  to plain integers or preformatted text; <!-- rule: lmf.review.trusted-templates class=hard enforced=no tasks=messages-lmf1 -->
- keep templates trusted—never treat user input as an LMF1 template; and <!-- rule: lmf.review.test-cases class=judgment enforced=no tasks=messages-lmf1 -->
- test reordered arguments, apostrophe escaping, number/date styles,
  plural/select/selectordinal/choice branches, missing resources, unresolved
  Libertas values, and malformed formatted-text bytes.
