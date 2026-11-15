# Schema documentation invariants

This document is the normative authoring contract for a codebot that creates or
changes localized Libertas schema documentation. It applies to every authored
schema-visible package, function, named type, parameter, field, Enumeration
member, Union variant, inline value, List container, and List item. It also
applies to custom document-property values and package `StringResources`.

<!-- rule: doc.coverage.authored-public class=hard enforced=no tasks=source-documentation -->
These are codebot authoring and review requirements. Generated document JSON
permits empty strings so parsers can represent compiler-generated nodes and
older or incomplete source. That structural allowance is not permission for a
codebot to omit documentation from an authored public declaration.

<!-- rule: doc.coverage.discriminator-exception class=hard enforced=no tasks=source-documentation -->
The only ordinary exception is a compiler-generated TypeScript discriminator
field whose document text the parser intentionally suppresses. Non-schema
implementation details do not need schema documentation. <!-- rule: doc.coverage.no-hidden-exception class=hard enforced=no tasks=source-documentation --> `Hidden`, `ReadOnly`,
protocol, persistent-data, and chart-related nodes are not exceptions: their
meaning can still be used by validation, accessibility, localization, tools,
management interfaces, or a future presentation.

## Source and output invariants

<!-- rule: doc.source.authoritative class=judgment enforced=no tasks=source-documentation -->
- Source comments are authoritative. Never patch generated schema or document
  JSON to improve wording. <!-- rule: doc.source.label-and-description class=hard enforced=no tasks=source-documentation -->
- Give every authored document node at least two ordinary lines: a concise
  natural label followed by a complete plain-language description. Do not rely
  on a parser fallback derived from a native identifier. <!-- rule: doc.source.rust-comments class=hard enforced=no tasks=source-documentation -->
- Rust package text uses inner rustdoc (`//!`) at the start of `src/lib.rs`.
  Rust declarations use outer rustdoc (`///`). <!-- rule: doc.source.typescript-comments class=hard enforced=no tasks=source-documentation -->
- TypeScript package text uses leading triple-slash comments (`///`) on the
  selected entry module. TypeScript declarations use JSDoc (`/** ... */`). <!-- rule: doc.source.engineering-comments class=judgment enforced=no tasks=source-documentation -->
- Ordinary `//` comments are not localized. Use them, a README, or developer
  documentation for engineering details that must not appear in the user
  document. <!-- rule: doc.source.same-change class=judgment enforced=no tasks=source-documentation -->
- Update source documentation in the same change as the behavior, meaning,
  choices, results, constraints, or presentation it describes. <!-- rule: doc.source.inspect-output class=judgment enforced=no tasks=source-documentation -->
- Inspect the generated default-locale document as an output. Fix the owning
  source comment when a label, description, extension, or List half lands on
  the wrong node.

The final document has one package object, function documents in `List`, named
type documents in `Types`, nested field documents in each node's `List`, and
Enumeration members in `Enum`. <!-- rule: doc.output.type-member class=hard enforced=yes tasks=source-documentation title="A field document Type is a non-negative integer schema-node identifier" --> A field document's numeric `Type` member is its
schema-node identifier; <!-- /rule --> <!-- rule: doc.output.generated-id class=hard enforced=no tasks=source-documentation --> it is not prose and is never authored or translated. <!-- rule: doc.output.use-site-and-type class=judgment enforced=no tasks=source-documentation -->
Documentation traversal stops at a named type reference. Consequently, the
use-site field or parameter and the referenced named type must each make sense
on their own. One never substitutes for the other.

## Universal user-language invariant

<!-- rule: doc.language.audience class=judgment enforced=no tasks=source-documentation -->
Write for the people who use the capability, including nontechnical home,
commercial, research, engineering, and healthcare users. Use specialist domain
terms when that audience genuinely uses them, and explain unfamiliar terms when
the surrounding context does not make them clear.

A label must:

<!-- rule: doc.label.short-natural class=judgment enforced=no tasks=source-documentation -->
- be a short, natural name or action phrase; <!-- rule: doc.label.distinct class=judgment enforced=no tasks=source-documentation -->
- distinguish the node from its siblings; <!-- rule: doc.label.standalone class=judgment enforced=no tasks=source-documentation -->
- work when read aloud or shown without its parent; and <!-- rule: doc.label.not-identifier class=judgment enforced=no tasks=source-documentation -->
- not be a source identifier, version token, full sentence, or keyword list.

<!-- rule: doc.description.adds-meaning class=judgment enforced=no tasks=source-documentation -->
A description must add meaning beyond the label and declared type. It must say
what the person chooses, enters, observes, receives, or can accomplish. <!-- rule: doc.description.behavior class=judgment enforced=no tasks=source-documentation --> When it
describes behavior, use active voice and identify the actor, action, object,
scope, material condition, and modality. Preserve distinctions such as “can,”
“must,” and “automatically.”

All localized prose must:

<!-- rule: doc.prose.concrete-terms class=judgment enforced=no tasks=source-documentation -->
- use concrete domain nouns and verbs rather than vague words such as “data,”
  “value,” “item,” “configuration,” or “information” by themselves; <!-- rule: doc.prose.special-values class=judgment enforced=no tasks=source-documentation -->
- explain an important empty, unavailable, zero, false, `None`, or `Unknown`
  meaning when a person could otherwise misinterpret it; <!-- rule: doc.prose.no-visual-cues class=judgment enforced=no tasks=source-documentation -->
- remain understandable without relying on order, screen position, color,
  icon, shape, animation, or a particular gesture; <!-- rule: doc.prose.client-neutral class=judgment enforced=no tasks=source-documentation -->
- avoid instructions tied to one client, such as “click the button” or “tap the
  row,” unless the behavior itself exists only on that client; <!-- rule: doc.prose.explicit-nouns class=judgment enforced=no tasks=source-documentation -->
- use explicit nouns instead of an ambiguous “it,” “this,” or “they”; <!-- rule: doc.prose.no-stuffing class=judgment enforced=no tasks=source-documentation -->
- avoid unexplained acronyms, keyword stuffing, unsupported claims, and lists of
  capabilities the App does not provide; and <!-- rule: doc.prose.translatable class=judgment enforced=no tasks=source-documentation -->
- be complete enough to translate without guessing an actor, scope, condition,
  or consequence.

<!-- rule: doc.prose.no-internal-vocabulary class=judgment enforced=no tasks=source-documentation -->
Do not expose internal vocabulary in localized prose. In particular, do not
describe something to a user as a function, field, type, struct, table, enum,
Union, variant, schema, payload, protocol, endpoint, request, response, runtime,
discriminator, index, FQN, Avro value, wire value, or implementation version. <!-- rule: doc.prose.no-mechanics class=judgment enforced=no tasks=source-documentation -->
Do not expose native casing, storage encoding, Libertas Schema Paths, integer codes, byte
layouts, or transport mechanics. Preserve required engineering facts in an
ordinary source comment or developer document instead.

<!-- rule: doc.language.technology-neutral class=judgment enforced=no tasks=source-documentation -->
Apply this rule both when authoring source comments and when translating their
localized text. Write natural, idiomatic language for the intended audience;
keep it technology-neutral by describing the capability rather than its
implementation. An App's “weather endpoint,” for example, is a “weather
service” to its users. Prefer the specific purpose or the natural equivalent
of “service” in the target language. This is a wording choice, not a mechanical
word replacement: retain specialist terms when they describe the user's domain
and their precise meaning matters. <!-- rule: doc.language.preserve-meaning class=judgment enforced=no tasks=source-documentation --> Preserve every material meaning, choice,
control, automatic behavior, result, warning, condition, prerequisite, and limit. <!-- rule: doc.language.identifiers-exact class=hard enforced=no tasks=source-documentation -->
This guidance changes prose only; identifiers, structural keys, and message
placeholders retain their exact spelling and syntax.

### Domain references and discoverability

<!-- rule: doc.discovery.domain-references class=judgment enforced=no tasks=source-documentation -->
Document the names people use to identify a supported capability. Where
relevant, connect formal domain references, document or form numbers, standard
abbreviations, and common names in natural descriptive prose. For example, a
tax topic can identify the forms and statutory provisions it covers, while an
equipment topic can identify the industry standard it implements. These are
user-domain references, distinct from internal identifiers and wire codes.
Preserve their exact meaning during translation and explain unfamiliar
abbreviations. Keep labels concise; put this context in descriptions.

<!-- rule: doc.discovery.actual-scope class=judgment enforced=no tasks=source-documentation -->
State the actual purpose, supported scope, and material conditions. A reference
to a subject does not establish that every case within it is supported. <!-- rule: doc.discovery.link-topics class=judgment enforced=no tasks=source-documentation --> For
links that share one request but bind different topics or records, document
the individual topic or selected choice as well as the common operation, so
the destination is understandable from its own documented meaning and data. <!-- rule: doc.discovery.enumeration-destination class=judgment enforced=no tasks=source-documentation -->
When an Enumeration member identifies a navigation destination, its extracted
member description owns the destination’s domain references, abbreviations,
and common names. For example, a qualified business income topic can associate
Section 199A, Form 8995/8995-A, and QBI deduction with that supported topic.
The client supplies the descriptions of the actual bound members; a generic
request description does not replace them.
Maintain this information in its owning source documentation when behavior
changes; generated documents and private code comments are not substitutes.

## Node-specific invariants

### Package

<!-- rule: doc.package.label-description class=judgment enforced=no tasks=source-documentation -->
The package label is the recognizable product, service, or capability name,
not the package identifier. Its description explains the overall outcome,
intended setting or audience, major capabilities, and material prerequisites or
limits. Avoid internal architecture and unsupported marketing claims.

### Function

<!-- rule: doc.function.discovery-description class=judgment enforced=no tasks=source-documentation -->
The function label names the activity a person can configure or use. Its
description is the complete discovery description and must additionally follow
the [developer function-comment guide](function-comments.md),
including the seven completeness categories, explicit actors, and final
developer attribution.

Plaza creates exactly one discovery embedding per public function from that
function's display name and description. Package, argument, type, protocol,
chart, attribute, and generated appendix text is not added. Excellent type and
field documentation therefore never repairs an incomplete function comment. <!-- rule: doc.function.attribution-only-functions class=hard enforced=no tasks=source-documentation -->
Developer attribution belongs only in function descriptions; do not append it
to package, type, field, member, or extension text.

### Named type

<!-- rule: doc.type.concept class=judgment enforced=no tasks=source-documentation -->
Describe the real-world concept represented by one value and why a person sees
or supplies it. Do not say that it is a struct, table, grouping, payload, or a
container of fields. A reusable named type must make sense independently of
every place that references it.

### Parameter and field

<!-- rule: doc.field.input-output class=judgment enforced=no tasks=source-documentation -->
Give every parameter and field a contextual natural label. For an input,
explain what the user chooses or enters and the effect or scope of that choice.
For output or read-only information, explain what the App reports, records, or
calculates and when it is meaningful.

<!-- rule: doc.field.empty-string class=judgment enforced=no tasks=source-documentation -->
Match the wording to the [type's use](app-contract.md#user-facing-and-non-user-facing-types).
For ordinary optional text stored as `""`, say “Leave it empty if absent,” and
explain any consequence of leaving it empty, such as using another name or
leaving an activity ungrouped. Do not say “Leave it undefined” when the answer
is an empty String. <!-- rule: doc.field.nullable-absence class=judgment enforced=no tasks=source-documentation --> For a genuinely nullable answer, explain what absence means
and how it differs from a present empty answer or zero. AI-import instructions
must use the same meaning under the
[blank-value conversion contract](llm-document-transformation.md#blank-values-and-output-types).

<!-- rule: doc.field.output-and-internal class=judgment enforced=no tasks=source-documentation -->
An output description explains the reported result and its availability rather
than telling the person to enter it. A non-user-facing schema field still needs
domain documentation: describe what the App records and the meaning of missing
or empty content. Keep mapping code, storage encodings, and other engineering
details in ordinary `//` comments.

Additional field rules:

<!-- rule: doc.field.boolean class=judgment enforced=no tasks=source-documentation -->
- A Boolean label states the positive choice or state. Its description makes
  the `true` and `false` consequences unambiguous when they matter; “Enabled” by
  itself is not enough.
- For a nullable value, distinguish “not supplied” or “unavailable” from zero,
  false, and an empty collection where that affects meaning. <!-- rule: doc.field.quantities class=judgment enforced=no tasks=source-documentation -->
- For a number, date, time, duration, or physical quantity, describe the domain
  meaning and scope. Do not explain its integer width, epoch, raw date encoding,
  or storage unit as implementation mechanics. Declare machine-readable unit
  and format metadata with the appropriate schema annotation. <!-- rule: doc.field.no-ai-hints class=judgment enforced=no tasks=source-documentation -->
  The platform supplies [AI value-representation hints](llm-document-transformation.md#value-representation-hints)
  from that metadata; do not duplicate those hints in source comments. Once a
  `PhysicalUnit` is declared, the comment never names the unit again (no "in
  gallons" or "(percent)"). Clients already display the declared unit with the
  value under the presentation contract
  ([physical units](charts/2d.md#551-physical-units): `50` with `percent` shows
  as `50%`), so a comment naming it would repeat it on screen. <!-- rule: doc.field.system-object class=judgment enforced=no tasks=source-documentation -->
- For a Libertas system object, name its actual role, such as “Rain gauge,”
  “Patient,” “Delivery vehicle,” or “Sprinkler valve.” Never call it a device
  ID, user ID, endpoint ID, or plain integer. Keep the Libertas system-object
  type intact so clients can resolve its localized identity. <!-- rule: doc.field.free-text class=judgment enforced=no tasks=source-documentation -->
- For free text, describe what content is appropriate and any material use or
  visibility. Never present a secret as ordinary documented input. <!-- rule: doc.field.use-site-retained class=judgment enforced=no tasks=source-documentation -->
- A use-site field retains its own documentation even when its named type is
  thoroughly documented.

### Enumeration and bitflags

<!-- rule: doc.enum.members class=judgment enforced=no tasks=source-documentation -->
The Enumeration type describes the decision, state, or classification as a
whole. Every member has a natural label and a description of its observable
meaning or consequence. This includes `None`, `Unknown`, and zero-valued
members. Never expose the native constant name or numeric code.

<!-- rule: doc.enum.bitflags class=judgment enforced=no tasks=source-documentation -->
A Bitflags catalog follows the same rules, but each nonzero member describes
one independently selectable option. Do not phrase its type as one mutually
exclusive choice.

<!-- rule: doc.enum.no-positional class=hard enforced=no tasks=source-documentation -->
The parsers' public positional `@LibertasEnum(A, B, C)` and
`#[libertas_enum(A, B, C)]` compatibility forms fabricate member nodes without
a source-document location for each member. A codebot therefore must not
generate those forms: they cannot satisfy this all-member documentation
invariant. Use a native numeric enum whose declaration and every member can be
documented. Existing positional catalogs remain parseable, but parser support
does not waive the codebot authoring rule.

### Union and variant

<!-- rule: doc.union.variants class=judgment enforced=no tasks=source-documentation -->
The Union type describes the family of possible actions, events, states, or
results. Every variant has a standalone natural label plus a description of the
observable action or state, its trigger or scope, and its important consequence.
Do not label variants “Request,” “Response,” “V1,” or by their discriminator
tokens. <!-- rule: doc.union.inline-fields class=judgment enforced=no tasks=source-documentation --> Document every inline field inside a variant by the ordinary field
rules.

<!-- rule: doc.union.typescript-source class=hard enforced=no tasks=source-documentation -->
TypeScript may obtain final variant text from a discriminator member, the Union
branch, or the variant's named payload type. Choose one authoritative source
for the intended result. If more than one source contains prose, keep the text
semantically identical; never depend on precedence to hide contradictory
wording. Rust takes the variant's own rustdoc.

### List container and item

<!-- rule: doc.list.two-sections class=hard enforced=no tasks=source-documentation -->
Every List has two complete document sections separated by a line containing at
least four hyphens:

<!-- rule: doc.list.container-half class=hard enforced=no tasks=source-documentation -->
- Above `----`, use a plural label and explain the collection's purpose,
  meaningful ordering, limits, or selection behavior. <!-- rule: doc.list.item-half class=hard enforced=no tasks=source-documentation -->
- Below `----`, use a singular label and explain what one entry represents and
  how a person recognizes or uses it.

<!-- rule: doc.list.both-halves class=judgment enforced=no tasks=source-documentation -->
Author both halves even when the item is a separately documented named type. <!-- rule: doc.list.property-ownership class=hard enforced=no tasks=source-documentation -->
Custom properties above the divider belong to the List container; properties
below it belong to the List item.

### Hidden, interaction-context, and chart data

<!-- rule: doc.hidden.truthful class=judgment enforced=no tasks=source-documentation -->
Document hidden and read-only values truthfully without revealing a secret.
Explain their domain purpose, such as associating a follow-up action with the
same inspection, rather than describing them as a cookie, token, cursor, or
transaction mechanism.

<!-- rule: doc.chart.accessible class=judgment enforced=no tasks=source-documentation -->
Chart documentation describes the domain relationship and the meaning of each
visible quantity or category. It must remain useful to a screen reader and must
not depend on axis position, color, shape, rendering library, or client-computed
geometry. <!-- rule: doc.chart.visible-text class=judgment enforced=no tasks=source-documentation --> Visible human text uses a localized Enumeration, `FormattedText`, or
the appropriate Libertas system-object type when applicable. A plain `String`
is reserved for genuine literal domain or user-authored text, not a fixed
English vocabulary.

## Exact source-document syntax

<!-- rule: doc.syntax.label-line class=hard enforced=no tasks=source-documentation -->
The first ordinary line becomes `DisplayName` for a package, function, or named
type, and `Name` for a field, item, member, or variant. Remaining ordinary lines
become `Description`. For reliable cross-language output, always provide both a
label line and at least one description line.

<!-- rule: doc.syntax.flowing-description class=hard enforced=no tasks=client-runtime -->
Long descriptions use HTML-style flowing prose. A single logical line break in
`Description` is source wrapping and is canonicalized to exactly one space. A
run of two or more logical line breaks is canonicalized to one LF so it remains
one intentional paragraph break. CRLF counts as one logical line break, as do a
lone CR or LF; horizontal whitespace around a break does not change the rule. <!-- rule: doc.syntax.paragraph-breaks class=hard enforced=no tasks=source-documentation -->
Codebots may wrap a long source sentence across adjacent rustdoc/JSDoc lines,
but must leave a blank documentation line only when one paragraph break is
intended. This normalization applies to `Description`, not custom-property or
`StringResources` values.

<!-- rule: doc.syntax.typescript-param class=hard enforced=no tasks=source-documentation -->
TypeScript function parameters may use `@param name` documentation. The parser
merges that text over documentation attached directly to the parameter, so use
one authoritative location or keep both semantically identical. <!-- rule: doc.syntax.rust-param class=hard enforced=no tasks=source-documentation --> Rust parameter
rustdoc and schema attributes are written on the typed parameter inside the
exported function signature.

### TypeScript example

```typescript
/// Greenhouse climate care
/// Helps growers monitor rooms and maintain suitable plant conditions.
/// @LibertasDefaultLocale("en-US")

/**
 * Fan behavior
 * Describes how ventilation responds to the selected temperature.
 */
export enum FanBehavior {
    /**
     * Automatic ventilation
     * The greenhouse starts and stops the fan according to room temperature.
     */
    Automatic,

    /**
     * Fan remains off
     * The greenhouse does not start the fan automatically.
     */
    Off,
}

/**
 * Growing-room settings
 * Describes the conditions the grower chooses for one room.
 */
export interface RoomSettings {
    /**
     * Room thermostat
     * The grower chooses the thermostat that measures this growing room.
     */
    thermostat: LibertasDevice;

    /**
     * Notes
     * The grower records a short note shown with this room.
     * [DefaultText]
     * Check seedlings each morning
     */
    notes: string;

    /**
     * Alert recipients
     * Lists the people who receive temperature warnings for this room.
     * ----
     * Alert recipient
     * Identifies one person who receives this room's warnings.
     */
    recipients: LibertasUser[];
}

/**
 * Care for a growing room
 * The greenhouse monitors one room and warns the selected people when its
 * temperature leaves the grower's preferred range. The grower can change that
 * range and the fan behavior at any time. Developed by Example Developer
 * (example_developer).
 * [DefaultText]
 * Seedling room care
 */
export function careForRoom(
    /**
     * Growing-room choices
     * The grower chooses the room, warning recipients, and starting notes.
     */
    settings: RoomSettings,
): void {
    void settings;
}
```

### Rust example

```rust
//! Greenhouse climate care
//! Helps growers monitor rooms and maintain suitable plant conditions.
//! #[libertas_default_locale("en-US")]

/// Growing-room settings
/// Describes the conditions the grower chooses for one room.
#[derive(LibertasExport, LibertasAvroDecode)]
pub struct RoomSettings {
    /// Room thermostat
    /// The grower chooses the thermostat that measures this growing room.
    pub thermostat: LibertasDevice,

    /// Notes
    /// The grower records a short note shown with this room.
    /// [DefaultText]
    /// Check seedlings each morning
    pub notes: String,

    /// Alert recipients
    /// Lists the people who receive temperature warnings for this room.
    /// ----
    /// Alert recipient
    /// Identifies one person who receives this room's warnings.
    pub recipients: Vec<LibertasUser>,
}

/// Care for a growing room
/// The greenhouse monitors one room and warns the selected people when its
/// temperature leaves the grower's preferred range. The grower can change that
/// range and the fan behavior at any time. Developed by Example Developer
/// (example_developer).
/// [DefaultText]
/// Seedling room care
#[libertas_export]
pub fn care_for_room(
    /// Growing-room choices
    /// The grower chooses the room, warning recipients, and starting notes.
    settings: RoomSettings,
) {
    let _ = settings;
}
```

<!-- rule: doc.syntax.example-profile class=judgment enforced=no tasks=source-documentation -->
Examples use a placeholder developer profile. A codebot must substitute the
current Plaza account and display name under the function-comment contract.

## Custom document properties

A custom property, also called a document extension, uses this exact form:

```text
[PropertyName]
Property value starts on the next line.
It continues until another property marker or the end of this document section.
```

Rules:

<!-- rule: doc.property.marker-line class=hard enforced=no tasks=source-documentation -->
- The case-sensitive `[PropertyName]` marker is the only content on its line. <!-- rule: doc.property.after-description class=hard enforced=no tasks=source-documentation -->
- Put extensions after the ordinary label and description. <!-- rule: doc.property.before-tags class=hard enforced=no tasks=source-documentation -->
- Put all ordinary text and extension blocks before the first
  `@Libertas...` schema tag in TypeScript JSDoc. The TypeScript document parser
  stops collecting document text at that first tag. This ordering is also safe
  in Rust rustdoc. <!-- rule: doc.property.no-repeat class=hard enforced=no tasks=source-documentation -->
- Multiple extensions are allowed. Never repeat a property name in one section. <!-- rule: doc.property.string-value class=hard enforced=yes tasks=source-documentation title="Every document property value is a string" -->
- Every value is a string. <!-- /rule --> <!-- rule: doc.property.value-normalization class=hard enforced=no tasks=source-documentation --> Leading and trailing blank space is removed while
  internal newlines are preserved. <!-- rule: doc.property.flattened class=hard enforced=no tasks=client-runtime -->
- The property is flattened beside the owner's ordinary document members; it
  is not nested inside an `Extensions` object. <!-- rule: doc.property.localized-values class=hard enforced=no tasks=source-documentation -->
- Property names and structural keys are stable and are never translated.
  Property values are localized.
- On a List, the `----` divider controls property ownership as well as ordinary
  text: properties above it belong to the container and properties below it
  belong to the item. <!-- rule: doc.property.unknown class=judgment enforced=no tasks=source-documentation -->
- Unknown valid properties are preserved and translated, but Libertas gives
  them no generic display or behavior. Use one only when a documented consumer
  contract defines its semantics. Never invent an extension from its name.

<!-- rule: doc.property.reserved-names class=hard enforced=yes tasks=source-documentation title="DisplayName, Description, List, StringResources, Strings, Enum and DocumentVersion are reserved document property names" -->
The names `DisplayName`, `Description`, `List`, `StringResources`, `Strings`,
`Enum`, and `DocumentVersion` are globally reserved. <!-- /rule --> <!-- rule: doc.property.structural-names class=hard enforced=no tasks=source-documentation --> A name that collides with
a structural key at its owning level, including `Locale`, `Types`, `Name`, or
`Type`, is also reserved. <!-- /rule --> `Strings` is prohibited; document format `0.0.2` has
one package-owned `StringResources` map and no function- or type-owned string
table. See the [reserved property names](#custom-document-properties) and
[single package catalog contract](message-format.md#uses-and-storage).

### Standard properties

<!-- rule: doc.boolean.labels class=hard enforced=yes tasks=source-documentation title="`[TrueText]` and `[FalseText]` are optional Boolean field labels, each one non-empty line" -->
`[TrueText]` and `[FalseText]` are optional localized Boolean field labels.
Each supplied value is one non-empty line. <!-- rule: doc.boolean.label-effects class=hard enforced=no tasks=source-documentation --> They label `true` and `false`
respectively; omitted labels use localized Yes and No. Either property selects
an [explicit choice control](presentation.md#boolean-answer-controls). Labels do
not supply defaults, change Boolean values, or cause submission. Use ordinary
document localization and fallback. <!-- rule: doc.boolean.semantics class=hard enforced=no tasks=app-design,source-documentation,data-model --> **A Boolean's `true` and `false` can carry any two
meanings the field needs**, such as Accept and Decline: Yes and No are only the
fallback captions, `[TrueText]` and `[FalseText]` state any other meaning, and
the App's rules read the value under that meaning.

```rust
/// Accept this invitation?
/// Choose whether to accept or decline.
/// [TrueText]
/// Accept
/// [FalseText]
/// Decline
pub accept_invitation: bool,
```

<!-- rule: doc.boolean.unanswered class=hard enforced=no tasks=client-runtime,field-presentation,data-model -->
Without a schema Default, a required Boolean starts unanswered. Both choices
are unselected; the editor holds an undefined value and refuses submission until
one is chosen. It never encodes undefined as false or null. <!-- rule: doc.boolean.nullable-third-choice class=hard enforced=no tasks=client-runtime --> A nullable Boolean's
explicit null remains distinct from the true/false choices; editable nullable
Booleans always offer a third clearing choice. Its caption uses `NullText` or
localized None. CopyFrom and EditRequest preserve supplied values.

<!-- rule: doc.boolean.switch class=hard enforced=no tasks=source-documentation -->
A non-nullable Boolean uses an on/off switch only when it has an explicit
Boolean Default and neither custom label is supplied. Otherwise it uses the two-choice control,
including when a default preselects an answer. This decision follows metadata,
not whether the user has already answered. Fixed and ReadOnly retain their
existing restrictions on editing; no new schema attribute or opcode is added.


<!-- rule: doc.defaulttext.placement class=hard enforced=yes tasks=source-documentation title="`[DefaultText]` is one non-empty line, only on an effective String or EndpointServer field or on a function" -->
`[DefaultText]` supplies one non-empty localized line of initial text on an
effective `String` field, an `EndpointServer` field, or a function. On a
function it is the initial task name. <!-- rule: doc.defaulttext.real-content class=judgment enforced=no tasks=source-documentation --> It is real, safe content suitable for
the locale and context, never a placeholder instruction, sentinel, resource
key, source identifier, or schema `Default`. It does not emit or replace the
`Default` schema attribute. <!-- rule: doc.defaulttext.omit class=judgment enforced=no tasks=source-documentation --> Omit it when the ordinary fallback already gives
exactly the intended value, or no truthful general value exists. Never
fabricate user-specific identity, clinical, financial, or security content.

<!-- rule: doc.nullable.texts class=hard enforced=yes tasks=source-documentation title="`[NullText]` and `[SetText]` are one non-empty line each, only on a nullable field" -->
`[NullText]` supplies one non-empty localized line for the explicit null of a
nullable field: the null branch of its
[Nullable choice](presentation.md#nullable-choices) and its displayed null.
`[SetText]` supplies one non-empty localized line for the value branch. <!-- rule: doc.nullable.required-texts class=hard enforced=yes tasks=source-documentation title="An editable Nullable declares `[NullText]`, and `[SetText]` wherever its control draws a value answer, even when the text equals the fallback" --> An
editable Nullable must declare `[NullText]`, and `[SetText]` wherever the
control draws a value answer (a Number, String, Bytes, date or time, Table or List), even when the intended text
equals the fallback; the general rule to omit a property matching its fallback
does not apply to them. <!-- rule: doc.nullable.null-text-rationale class=judgment enforced=no tasks=app-design,source-documentation,data-model --> This is deliberate: AI CodeBots
especially assume null means None, which has caused subtle design flaws;
writing `[NullText]` on every question a person answers forces the author to
decide what null means there. <!-- rule: doc.nullable.null-text-meaning class=hard enforced=no tasks=app-design,source-documentation,data-model --> **Null can carry any meaning the field needs**: it
means None only until `[NullText]` states another meaning, which every
Nullable, editable or not, must then declare; a non-editable Nullable shows it
wherever its null is displayed. <!-- rule: doc.nullable.no-settext-boolean-picker class=hard enforced=no tasks=source-documentation --> A nullable Boolean (TrueText, FalseText, NullText) and a
picker (NullText, then its members) never show `[SetText]`. <!-- rule: doc.nullable.editable class=hard enforced=no tasks=source-documentation --> A client falls back
to localized None and Specify only when a label is missing, and a non-editable
Nullable never needs `[SetText]`: a Nullable is editable when a Request variant or a
function's configuration reaches it without passing through a ReadOnly, Hidden
or Fixed node ([Nullable choices](presentation.md#nullable-choices), under the
[required answer-label rule](#standard-properties)). <!-- rule: doc.nullable.labels-only class=hard enforced=no tasks=source-documentation --> Both change only labels:
they neither supply a value nor label false, zero, a present empty string or
collection, an unanswered required field, or a selected Nil Union branch. <!-- rule: doc.nullable.two-answers class=judgment enforced=no tasks=source-documentation -->
Write them as the two answers the person chooses between; `SetText` is the
lead-in to the value entered right after it, such as “Died in 2026 on” before a
date.

```rust
/// Signature on delivery?
/// Choose Yes or No, or keep the carrier's own rule.
/// [NullText]
/// Carrier's rule
pub signature_required: Option<bool>,

/// Real estate professional status (Pub 925)
/// Choose whether you qualify under section 469(c)(7); each rental still records participation.
/// [NullText]
/// Not a real estate professional
/// [SetText]
/// Real estate professional
pub real_estate_professional: Option<ProfessionalRentals>,
```

<!-- rule: doc.initial-names.concise class=judgment enforced=no tasks=source-documentation -->
Choose concise, useful, editable initial names for tasks and Endpoint servers.
Their ordinary localized labels already provide naming fallbacks; declare
`DefaultText` only when the intended initial name differs from that fallback.

<!-- rule: doc.property.no-substitute class=hard enforced=no tasks=source-documentation -->
No standard property substitutes for the owner's ordinary label and
description.

<!-- rule: doc.initial-text.persisted-priority class=hard enforced=no tasks=client-runtime -->
Persisted or user-assigned text has absolute priority and is immutable to
programmatic defaulting. Clients must not replace it during locale, document,
or schema refresh, migration, upgrade, or any later default application. This
includes a deliberately empty saved value or task name. <!-- rule: doc.initial-text.priority-chain class=hard enforced=no tasks=source-documentation --> The remaining priority
chain applies only when no persisted value exists and the client is
constructing a new value. For a new String or Endpoint-server presentation
value left to ordinary initialization by the
[shared copy rules](schema.md#shared-copyfrom-and-copyto-rules),
an applicable schema `Fixed`/`Default` value has first initialization
priority, localized `DefaultText` is next, and the ordinary empty or localized
field-label fallback is last. For a new blank task name, localized
`DefaultText` has first initialization priority, then the localized
function `DisplayName`, then the native function name.

#### Answer labels in AI descriptions

The answer labels a field document declares reach the model with the values
they name. AI Import result schemas and Hub tool parameter and result
descriptions append one line after the node's
[value-representation hints](llm-document-transformation.md#value-representation-hints):

```text
Answer labels: true: <TrueText>; false: <FalseText>; null: <NullText>; a value: <SetText>
```

<!-- rule: doc.answer-labels.declared-only.authoring class=hard enforced=no tasks=source-documentation -->
Declare answer labels explicitly when AI needs their meanings; [fallback UI captions are not sent](#answer-labels-in-ai-descriptions).

<!-- /rule -->

<!-- rule: doc.answer-labels.declared-only class=hard enforced=no tasks=client-runtime -->
Each part appears only when its property is declared, in this order; with none
declared the line is omitted. Only declared labels are sent, never a fallback
such as Yes, No, None, or Specify. The labels are the document locale's text;
the line's own words are fixed English, like the hints. A label changes no
value: the model still reads and returns JSON `true`, `false`, `null`, or the
value itself, and conversion is unchanged. In
[AI Search](protocols.md#navigation-context-supplied-to-ai), a Boolean or null
row value whose field declares the matching label is sent as that label.

## StringResources prose

`StringResources` is one package-owned table shared by every function and type.
Stable keys are identifiers and are never translated. See the
[single package catalog contract](message-format.md#uses-and-storage). They are normally not
substitutes for localized prose. The task-access approval UI is the explicit
exception: it always shows each raw function `Permissions` key or `AccessHost`
key as the request summary, then separately shows that App package's localized
same-key value as the explanation. Two Apps may request the same permission for
different reasons, so each App owns and supplies its own explanation. See
[function permission reasons](protocols.md#function-and-endpoint-permissions) and
[network access reasons](network-access.md#declaration). <!-- rule: doc.resources.complete-messages class=judgment enforced=no tasks=source-documentation --> Every
default-locale value and every translated value must be natural user-facing
text and, when formatted, a complete LMF1 message rather than a fragment meant
to be concatenated.

<!-- rule: doc.resources.access-reason class=judgment enforced=no tasks=source-documentation -->
- A permission or network-access reason identifies the App action or service
  involved, what access is needed, and the user benefit. “Network access
  required” is not sufficient. <!-- rule: doc.resources.broad-network-reason class=judgment enforced=no tasks=source-documentation -->
- A concrete `AccessHost` reason explains in ordinary terms why the named
  service is contacted. Reasons for `{network_access_local}` and
  `{network_access_external}` explain why broad local or external connectivity
  is necessary; the reserved key does not excuse vague prose. <!-- rule: doc.resources.error-message class=judgment enforced=no tasks=source-documentation -->
- An error message says what happened and, when possible, what the person can
  do next. Do not expose stack traces, codes, or diagnostics as the explanation. <!-- rule: doc.resources.notification class=judgment enforced=no tasks=source-documentation -->
- A notification or status message names its actor, object, scope, and material
  condition. Do not assemble grammar from separately translated fragments. <!-- rule: doc.resources.persistent-keys class=hard enforced=no tasks=source-documentation -->
- Persistent record identity uses the declared fixed-width binary key codec.
  Human-readable names may use the package's localized formatted resources;
  never replace a key with rendered text or make its identity depend on locale.
- Translation preserves every LMF1 argument and its structure under the
  [translation document contract](translation.md).

<!-- rule: doc.resources.stable-keys class=hard enforced=no tasks=source-documentation -->
Localized templates stay in the package StringResources catalog. Changing
wording does not rename stored keys. Retained legacy data-name tuples follow
the [message-format compatibility rule](message-format.md#data-and-database-names).

## Translation invariant

Translate all user text, and nothing structural, under the
[translation document contract](translation.md), which owns what a translation
changes and preserves. See the
[schema document translation guide](../guides/translation.md)
for the complete translation workflow.

## Codebot workflow

<!-- rule: doc.workflow.inventory class=judgment enforced=no tasks=source-documentation -->
1. Inventory every schema-visible declaration and each document node it emits,
   including both List halves and every Enumeration or Union member.
2. Separate user meaning from engineering notes. Put user meaning in extracted
   rustdoc/JSDoc and preserve engineering facts in ordinary comments or
   developer documentation.
3. Write a standalone natural label and complete description for every authored
   node. Apply the node-specific rules above.
4. Apply the separate function-comment discovery and attribution checklist to
   every public function. <!-- rule: doc.workflow.properties class=hard enforced=no tasks=source-documentation -->
5. Add only documented custom properties, on the correct owner and List side.
   Add `DefaultText` only when a useful initial value differs from the ordinary
   fallback. Give every editable Nullable `NullText`, and `SetText` where its
   control draws a value answer, even when the text equals the fallback
   ([standard properties](#standard-properties)). <!-- rule: doc.workflow.resources class=hard enforced=no tasks=source-documentation -->
6. Write complete, specific default-locale `StringResources` values and verify
   every referenced permission or host reason exists.
7. Generate and inspect the default document. Confirm ownership, flattening,
   labels, descriptions, extensions, and intentionally empty generated nodes. <!-- rule: doc.workflow.checks class=judgment enforced=no tasks=source-documentation -->
8. Run the language checker and Studio schema validation. Treat failures as
   source defects; never patch generated JSON.

## Review checklist

- Every authored package, function, type, parameter, field, List half,
  Enumeration member, Union variant, and inline value has a non-empty natural
  label and description.
- Each label is concise, sibling-distinguishable, meaningful aloud, and not a
  source name, version, sentence, or keyword list.
- Each description adds purpose or meaning and names actor, action, object,
  scope, condition, and modality wherever behavior is involved.
- Input text explains what the user chooses and its effect; result text explains
  what is shown, recorded, calculated, or delivered.
- Empty, unavailable, zero, false, `None`, and `Unknown` meanings are clear where
  material.
- Every Enumeration member and Union variant stands alone. Every List has a
  plural container and singular item on the correct sides of `----`.
- No localized prose leaks source names, protocol roles, transport/storage
  mechanics, or visual-only cues. Domain terminology is audience-appropriate.
- Hidden and interaction-context values have truthful, non-secret domain
  descriptions. <!-- rule: doc.review.properties class=hard enforced=no tasks=source-documentation -->
- Every custom property is defined by a consuming contract, placed on the right
  owner, and follows the same prose rules. `DefaultText`, `TrueText`,
  `FalseText`, `NullText`, and `SetText` values are non-empty single lines. Omit properties
  whose intended text exactly matches the ordinary localized fallback, except
  the `NullText` and `SetText` an editable Nullable must declare; never
  fabricate personal or sensitive content to fill a default.
- Initial task and Endpoint server names are useful and editable; provide
  `DefaultText` only when the ordinary localized naming fallback differs.
- `StringResources` messages and reasons are truthful, specific, complete, and
  preserve LMF1 semantics.
- Every function separately passes the function-comment completeness,
  discovery, and attribution checklist.
- The generated default document was inspected and no generated schema or
  document JSON was edited by hand.

## Related contracts

- [Developer function-comment guide](function-comments.md)
- [Schema source annotation reference](annotations.md)
- [Presentation and localization](presentation.md)
- [App authoring contract](app-contract.md)
- [Schema reference](schema.md)
- [Schema document translation guide](../guides/translation.md)
