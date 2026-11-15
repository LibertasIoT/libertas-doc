# Presenting a field

Load this card when a task declares or changes how one field or type is
documented, edited and shown: its label and description text, `Hidden`,
`ReadOnly`, `Size`, `RangeMin`/`RangeMax`, an `EnumSource` chooser, a Nullable
or Boolean answer, a `Foldable`, `InlineChoices`, `SuppressLabel` or
`UIIncludeDescription` marker, a media field, or a `Link` on a row. Its rules
are generated from the [presentation contract](../reference/presentation.md),
which owns them; arranging the page in `display.json` is the
[Display Template card](display_template.md), and typed value formats
(`Format`, `Money`, `PhysicalUnit`, `Bitflags`, dates, `FormattedText`) are the
[formatted output card](formatted_output.md). The
[source documentation card](source_documentation.md) owns the complete source
documentation contract, and the [annotation inventory](schema_annotations.md)
gives each attribute's Rust and TypeScript spelling.

## Order of work

1. Document the node: a natural label line, a complete description, and the
   List divider and property placement (`doc.*`).
2. Choose its metadata: visibility and editability (`field.hidden-subtree`,
   `field.readonly-*`), bounds (`field.size-bounds`,
   `field.range-bounds-typed`), data-backed choices (`field.enumsource.*`,
   `field.picker-no-implicit-default`).
3. Decide how an answer is asked: unanswered and null ([ordinary initialization](../reference/schema.md#ordinary-initialization),
   `nullable.*`), Booleans (`boolean.answer-controls`).
4. Add presentation markers where the default drawing is wrong: folding
   (`fold.*`), inline choices (`inline.*`), label suppression (`suppress.*`),
   inline descriptions (`field.include-description`), inline media
   (`media.*`).
5. When a row opens a Request, declare the `Link` and check its rendering
   (`link.*`); the Link contract itself is in
   [protocols/runtime](protocols_runtime.md).
6. Compile with the Studio MCP `compile_schema` tool and fix every reported
   rule.

<!-- generated:codebot-rules task=field-presentation -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Standard properties

Owner: [documentation.md#standard-properties](../reference/documentation.md#standard-properties)

- `doc.boolean.unanswered`: Without a schema Default, a required Boolean starts unanswered. Both choices are unselected; the editor holds an undefined value and refuses submission until one is chosen. It never encodes undefined as false or null.

### Unanswered fields

Owner: [presentation.md#unanswered-fields](../reference/presentation.md#unanswered-fields)

- `field.unanswered`: **Unanswered fields.** A field the person answers that no `Default`, `Fixed` or copy initializes starts [undefined](../reference/schema.md#ordinary-initialization) (unanswered), and a non-Nullable field never holds null. When an editor opens, an undefined field shows only its neutral empty state: its placeholder, such as an unselected choice or "Choose date…", never an error look; an undefined Nullable's text uses the secondary text color. When Next or Submit is blocked, the client marks the first field that blocks it: below that field, an error icon and the missing-answer or validation message, in the platform's error color role and announced politely to assistive technology. It opens any fold or summary page holding the field and scrolls it into view; it does not move focus. The mark clears once that field passes, and it does not move to another field until the next blocked Next or Submit. An undefined field is reported as a missing answer, never as a null value.

### Presentation locks

Owner: [presentation.md#presentation-locks](../reference/presentation.md#presentation-locks)

- `tpl.lock.meaning`: `lock` keeps the person from changing a value the App's business logic already set. Its most common use is with [`AskOnAdd`](../reference/presentation.md#add-time-values): the person gives a member's defining value at Add, and the lock keeps it fixed afterwards. It is presentation only and restricts the person alone: initialization, `CopyFrom`, `EditRequest` values, calculation and link-return `CopyTo` results, AI Import, draft restoration, and the App write a locked value exactly as an unlocked one, and its validation and save-path handling are unchanged.

### Template validation

Owner: [presentation.md#template-validation](../reference/presentation.md#template-validation)

- `inline.lint.over-twelve` (judgment): It warns when an `InlineChoices` field or type declares more than 12 choices, which always use the [selector](../reference/presentation.md#inline-choice-ladder).

### Common field metadata

Owner: [presentation.md#common-field-metadata](../reference/presentation.md#common-field-metadata)

- `field.size-bounds`: `SizeMin`/`SizeMax` follow the [shared size contract](../reference/schema.md#43-value-constraint-and-presentation-attributes): List element count, String UTF-8 byte count, and Bytes byte count. Both TypeScript `@LibertasSize(min=..., max=...)` and Rust `#[libertas_size(min=..., max=...)]` emit these same attributes. The bounds are inclusive; size zero is valid unless excluded by the minimum. Nullable null remains separate from an empty value.
- `field.picker-no-implicit-default`: User and group pickers do not implicitly select the current user or a singleton candidate. Declare an explicit default using the [User and group default contract](../reference/schema.md#user-and-group-defaults).
- `field.hidden-subtree`: `Hidden` suppresses the complete value subtree, including list elements and controls, nested fields, summaries, and promoted values. Hidden data remains part of the typed value for validation and protocol encoding. `flatten` does not expose hidden descendants.
- `field.readonly-user-only`: `ReadOnly` restricts the user only: clients present the value and offer no ordinary editing control for it. It never restricts the platform or the application. Schema initialization (Fixed, Default, localized defaults), `CopyFrom` from a Response or the reference context, `EditRequest` values, calculation and link-return `CopyTo` results, draft restoration and the application's own handling of the submitted value all write a ReadOnly field exactly as they write an editable one. A value that must not change after initialization is `Fixed`, not ReadOnly. Hidden fields are ReadOnly in effect, since no editor shows them, and are written the same way.
- `field.readonly-list-membership`: `ReadOnly` on an Array/List prevents changing its membership or order: no Add, Remove, or reorder action may mutate that list. It does not recursively make member values read-only; members and their fields keep their own editability. To prohibit member edits, mark the applicable member type or fields as well. Context menus, swipe actions, pickers, and deferred actions must enforce the same list-owner rule at both presentation and execution. A promoted row does not change which list owns a removal or reorder operation.

### Add time values

Owner: [presentation.md#add-time-values](../reference/presentation.md#add-time-values)

- `field.ask-on-add.meaning`: **Add-time values.** `AskOnAdd` marks a value the person gives before a List member exists: a field of a List's member Table, reached from the member through struct fields, never through another List; or a List field whose member has a value of its own, which asks the member itself. A field's marker applies wherever its Table is such a member. Any number of a member's fields may carry it, of any type that has a value of its own: a scalar (String, Number, Boolean, DateOnly, DateTime, Bytes), an Enumeration, Bitflags, or a Union. When the person chooses **Add** on the List, the client first asks for every marked value; confirming creates the member with those values and every other field at its normal initialization, then shows the new member as the List's layout shows any new member: a [summary](../reference/presentation.md#summary-array-items) opens its item page, and inline rows, a table, or an accordion show it in place. Without every value, no member is added, and Cancel adds nothing.

### Common field metadata

Owner: [presentation.md#common-field-metadata](../reference/presentation.md#common-field-metadata)

- `field.ask-on-add.person-only`: **The person's Add only:** a member that AI Import, CopyTo, an EditRequest, or the App adds is unaffected, and after Add each value is an ordinary field; whether the person may change it is a template [`lock`](../reference/presentation.md#presentation-locks)'s, a separate concept.
- `field.ask-on-add.with-lock` (judgment): **Commonly paired with [`lock`](../reference/presentation.md#presentation-locks).** The usual purpose of `AskOnAdd` is a value that decides what the member is, such as a document's form: the person gives it once at Add, and a template `lock` (on a Union, `lock: "variant"`) then keeps the person from changing it, so the item editor shows the chosen form and never offers to switch it.
- `field.enumsource.picker-entry-display` (judgment): **A picker entry draws its source member as the source List draws it.** When the source List has a `summary` or `table` rule, each entry shows that rule's paths or columns for its member, with the text above as the entry's first line, and the filter matches that text too. The rule applies to the picker even when the List itself is Hidden and never drawn. Without such a rule the entry is the text alone. A host's chosen value, drawn outside the picker, shows only that text; a hosting item that needs the fuller display carries it as its own ReadOnly, OutputOnly field, normally of the same display-only type the source member uses, so the item and the picker entry read the same.
- `field.enumsource.index-not-identity` (judgment): A saved index remains a position, not a persistent record identity. Apps that need record identity must resolve the selection against the appropriate accepted-data revision.

### MIME-aware Bytes presentation

Owner: [presentation.md#mime-aware-bytes-presentation](../reference/presentation.md#mime-aware-bytes-presentation)

- `media.inline.pdf.authoring`: Choose media and size with the [platform preview limits](../reference/presentation.md#inline-media-size) in mind: PDF previews vary by client and video uses an icon.

### Inline media size

Owner: [presentation.md#inline-media-size](../reference/presentation.md#inline-media-size)

- `media.inline.images` (judgment): **Images.** The raster formats the thumbnail decodes draw inline. SVG and every other vector or markup format are never rendered, at any size.

### SuppressLabel: label-free value rows

Owner: [presentation.md#suppresslabel-label-free-value-rows](../reference/presentation.md#suppresslabel-label-free-value-rows)

- `suppress.source-and-hint`: `@LibertasSuppressLabel` in TypeScript and `#[libertas_suppress_label]` in Rust suppress the visual label contributed by the selected node. The parser result carries `{"Name":"SuppressLabel","Value":"true"}`; Studio compiles it into the [Display Template](../reference/presentation.md#display-template) as the Boolean `suppress_label` hint and removes it from the final schema. A template rule can set `false` to restore the ordinary label under the [cascade](../reference/presentation.md#display-template-cascade). The rule does not propagate to children.
- `suppress.label-only`: The hint suppresses a visual label, never a value or subtree. It does not change folding, editing, links, menus, membership permissions, help, or the semantic name used for accessibility and editor dialogs. A control without a visible label still has its localized accessible name. On a promoted row, only its displayed label is omitted; the promoted value and its declaring link remain available under the [action rule](../reference/presentation.md#actions-on-a-promoted-or-suppressed-row). `Hidden` continues to suppress the entire presentation subtree and wins before every template rule.

### Boolean answer controls

Owner: [presentation.md#boolean-answer-controls](../reference/presentation.md#boolean-answer-controls)

- `boolean.answer-controls`: Required Booleans without a default are explicit, initially unselected choices. A non-nullable Boolean with an explicit Default permits a switch only when no custom labels exist. In an editor, nullable Booleans use one inline radio group for true, false, and explicit null. All three choices remain visible when null is selected; choosing the null option stores actual null. Use localized [TrueText, FalseText, and NullText document properties](../reference/documentation.md#standard-properties) for their captions, with localized Yes, No, and None when omitted. Display labels never change wire values. Every client arranges the choices by rungs 1 to 3 of the [inline-choice ladder](../reference/presentation.md#inline-choice-ladder): beside the title, on one line below it, or one per line. ReadOnly and Fixed keep the current selection visible and disable editing.

### Inline choices

Owner: [presentation.md#inline-choices](../reference/presentation.md#inline-choices)

- `inline.union-one-per-line` (judgment): For a Union, the selected branch's fields render directly below its chosen radio control, before the next choice and indented to the choice's caption, only while the branch has exactly one visible field whose type is Number (a `DateOnly` date included), String, Boolean, `DateTime` or `TimeOnly`, where a Number or String carrying Bitflags or EnumSource counts as an Enumeration; a branch whose payload is itself such a scalar is that one field. Otherwise its fields render after the whole choice group: with no visible field, with two or more, or with one field of any other type, such as an Enumeration (Bitflags and EnumSource included), Union, struct, List, or Bytes (FormattedText included). A field is visible when its row is shown, under the same rules that show or withhold that row (Hidden, and template withholding such as [`when_single_choice`](../reference/presentation.md#single-choice-rows)), and the count is taken again whenever that visibility changes. Promotion of the Union or of its owner never changes this placement. Selecting another choice replaces the applicable fields. A Union with any declared branch that carries data, every branch but a `Nil` one, stands one per line, as the [ladder](../reference/presentation.md#inline-choice-ladder) below states. Authors who want the choices on one line with the selected content below use the [`tabs` layout](../reference/presentation.md#display-template-catalog) instead.
- `inline.false-and-forward-tolerance`: A template rule with `inline: false` still turns inline choices off. The marker changes presentation only, never data, encoding, or paths. A client without inline support keeps the choice selector.

### Link rendering

Owner: [presentation.md#link-rendering](../reference/presentation.md#link-rendering)

- `link.formatted-text-block`: On a FormattedText field the whole block is the tap target; links inside FormattedText markup are not supported.

### Folding structs and unions

Owner: [presentation.md#folding-structs-and-unions](../reference/presentation.md#folding-structs-and-unions)

- `fold.presence-enables`: Presence enables folding; `false` does **not** disable it. Studio compiles it into the [Display Template](../reference/presentation.md#display-template) as the `fold` hint and removes it from the final schema; clients read folding only from the template, where a rule may also change it per device.
- `fold.nondefault`: With `initially_open` `"nondefault"`, the value starts expanded when anything under it differs from its default and collapsed otherwise, so rarely used fields stay out of the way until they hold something. Its default is the value the App's default document gives that node; for a List member, or a node the default document does not reach, it is the default value of its type. Values are compared decoded, at every depth, including a selected Union variant and a List's member count, never by their displayed text, so the result is independent of locale. Values the platform generates rather than the person, such as a virtual device identity, are left out of the comparison, and working out the default registers nothing (no identity, name or icon). Like every initial state, it is decided when the page is displayed and does not close or open the value while the person edits.
- `fold.per-value-permission`: Each nested value owns its folding permission; a marker does not grant permission to its descendants. Lists follow the same explicit-permission rule. Changing a union variant reveals its new fields. Union selectors remain editable independently of folding.
- `fold.promoted-union-row`: For a promoted union, the shared owning row may fold when either the owning Table or the promoted union has Foldable on its type or field. Promotion and [`flatten`](../reference/presentation.md#flatten) remain independent of folding.

### Unanswered fields and null

Owner: [presentation.md#unanswered-fields-and-null](../reference/presentation.md#unanswered-fields-and-null)

- `nullable.empty-string-is-value`: An empty String is a present value, distinct from null; incomplete native input remains invalid.

### Nullable choices

Owner: [presentation.md#nullable-choices](../reference/presentation.md#nullable-choices)

- `nullable.blank-value.unanswered`: The null answer always comes first. While unanswered, no choice is selected and a picker shows “Select…”. A value answer chosen with its value still blank stays unanswered; null is never inferred from a blank editor. Read-only display keeps the caption described above.
- `nullable.initialization`: Initialization follows the Union ([type initialization](../reference/schema.md#shared-copyfrom-and-copyto-rules)): with no Default the question starts with neither answer selected, shows “Select…”, and blocks submission until the person chooses; the reserved Default [`$null`](../reference/schema.md#reserved-default-values) starts it on the null answer; any other Default starts it on the value answer with that value. The stored value, its encoding, CopyFrom, CopyTo, validation rules, AI Import, and calculations still see `null` or the value: the unanswered state exists only in the editor and is never encoded.

### Undefined and its sources

Owner: [schema.md#undefined-and-its-sources](../reference/schema.md#undefined-and-its-sources)

- `init.undefined-relationships` (judgment): **A node is undefined until it is initialized** by `Fixed`, a `Default`, a copy, a supplied value, or [type initialization](../reference/schema.md#ordinary-initialization) (an empty List, a null non-editable Nullable); otherwise it stays undefined until answered, which is how a must-answer question is asked ([EditRequest or WorkflowRequest](../reference/protocols.md#strict-questions-and-data-pages)). Undefined cannot be encoded, so no protocol message carries it; only an editor holds it, showing its neutral empty state ([unanswered fields](../reference/presentation.md#unanswered-fields)) and blocking submission while required.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Link rendering

Owner: [presentation.md#link-rendering](../reference/presentation.md#link-rendering)

- `link.hidden-rows-rejected`: Hidden rows have no link; Studio rejects that placement

### Folding structs and unions

Owner: [presentation.md#folding-structs-and-unions](../reference/presentation.md#folding-structs-and-unions)

- `fold.placement`: `Foldable` (optional `initially_open` true, false or nondefault) is permitted only on an effective Table, Union or List

<!-- /generated -->
