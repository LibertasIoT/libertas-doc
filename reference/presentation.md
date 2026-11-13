# Presentation and localization

Presentation attributes preserve the underlying typed value and wire encoding.
They tell Studio and clients how to validate, edit, localize, summarize, or
chart the value. How a value is arranged and styled on a device belongs to the
package's [Display Template](#display-template); the attributes that only
change that are compiled into it.

## Source documentation

Extraction syntax is only part of the contract. Follow the
[schema documentation invariants](documentation.md) for every
package, function, type, parameter, field, member, variant, and List half. A
public function must additionally follow the
[developer function-comment invariants](function-comments.md) so its
discovery description is complete, written in active voice, and explicit about
who performs every action.

TypeScript uses leading `///` package comments and JSDoc on declarations:

```typescript
/// Weather monitor
/// Helps people review recent conditions measured at one weather station.

/**
 * Weather measurements
 * Lists recent readings from the selected weather station.
 * ----
 * Weather measurement
 * Records one observation made by the selected weather station.
 */
samples: Sample[];
```

Rust uses inner rustdoc for the package and outer rustdoc for declarations:

```rust
//! Weather monitor
//! Helps people review recent conditions measured at one weather station.

/// Weather measurements
/// Lists recent readings from the selected weather station.
/// ----
/// Weather measurement
/// Records one observation made by the selected weather station.
pub samples: Vec<Sample>,
```

The first line is the display name. Remaining ordinary text is the description.
For a List, at least four hyphens separate container documentation above from
item documentation below. Put the divider above custom document properties that
belong to the item. <!-- rule: doc.plain-comments-ignored class=hard enforced=no tasks=source-documentation --> Plain code comments are ignored by document extraction.

Codebots always author at least one natural label line and one complete
description line for every authored node. Put custom `[PropertyName]` blocks
after that ordinary text and before the first TypeScript `@Libertas...` tag.
Use `[DefaultText]` when a useful initial String value, Endpoint server name,
or function task name differs from its ordinary localized fallback. Use
`[NullText]` and `[SetText]` when a [Nullable choice](#nullable-choices) needs its
own branch labels. The full
standard and custom property contract, including reserved names, List
ownership, localization and initialization, is in the
[schema documentation invariants](documentation.md#custom-document-properties).

## AI tool-call descriptions

<!-- rule: docs.comments.no-assistant-guidance class=hard enforced=no tasks=source-documentation -->
The Hub appends the tool-call [value-representation hints](llm-document-transformation.md#value-representation-hints)
to AI tool parameter and response descriptions, from the effective schema
attributes, including inherited attributes, and built-in date/time types, and
then the field's declared [answer labels](documentation.md#answer-labels-in-ai-descriptions).
AI Import uses the same list. Authored descriptions remain intact. App authors
and CodeBots do not repeat this guidance in source comments. Keep comments focused on
domain meaning, scope, consequences, and any App-specific limits. This does not
remove the requirement for a natural label and complete description on every
authored node.

<!-- rule: ai.hints.value-representation-only class=hard enforced=no tasks=client-runtime -->
These hints describe the tool JSON representation. They do not change App
storage or protocol encoding, supply missing values or dynamic choices, or
execute copying or calculations. <!-- rule: tpl.ai-descriptions-untouched class=hard enforced=no tasks=display-template --> Display Template presentation never reaches
AI descriptions: no template rule, `format` pattern, `label`, or `title`
changes what a tool or AI Import description says.

Which requests become tools, and how their parameters and values are formed,
including the `null` rule, is owned by [AI tool calls](protocols.md#ai-tool-calls).

## Display Template

Every App package has exactly one **Display Template**, version 0.1: the
presentation layer that decides how the values its schema describes are drawn
on a given device. The schema owns what the App accepts, sends, computes,
protects, or means; the template owns only how the same value is arranged and
styled. A template can make a correct App look worse, never make it incorrect.
This section owns the template's file, compiled form, cascade, conditions,
catalog, and behavior hints. The [Libertas Schema Path](schema.md#display-template-selectors)
owns its selector grammar and [publication](publication.md#display-template-identity)
owns its identity and delivery.

- <!-- rule: tpl.mandatory class=hard enforced=no tasks=client-runtime --> **Mandatory.** Every package has a template, including a TypesOnly package.
  A package with no presentation attributes and no `display.json` compiles to
  the empty template. The empty template is valid, has a digest and a
  documents-manifest entry, and renders exactly the platform default.
- **Sole runtime owner of presentation.** Clients read promotion and
  flattening, titles, folding, inline choices, SuppressLabel,
  UIIncludeDescription, the `Format` display pattern, inline media size, and
  chart appearance only from the template. <!-- rule: tpl.template-only-members class=hard enforced=yes tasks=display-template title="Promotion, flattening, labels and titles are authored only as template rules" --> Promotion, flattening, labels, and titles are authored
  only as template rules. <!-- rule: tpl.source-attributes-compiled class=hard enforced=yes tasks=display-template title="Source presentation attributes are compiled into the template at the lowest specificity and removed from the schema" -->
  The other source attributes remain the authoring convenience: Studio
  validates them, compiles every occurrence into a template rule at the lowest
  specificity, and removes them from the final schema. Each attribute's section
  below keeps its authoring text and meaning; the template is where that
  meaning reaches a client.
- <!-- rule: tpl.not-schema class=hard enforced=no tasks=display-template --> **Not part of the schema.** The template is absent from the schema JSON, its
  revision digest, and the App version. A template change is a document-only
  revision: it never rebuilds the App or allocates an App version.
- <!-- rule: tpl.rows-default class=hard enforced=no tasks=display-template --> **Opt-in arrangement.** `rows`, one row per child, is the default layout.
  Every other layout applies only where a rule asks for it. A `table` or
  table-arranged `summary` whose columns do not fit, and a `master-detail`
  below the pane threshold, fall back under the
  [width-based layout](#width-based-layout).
- <!-- rule: tpl.presentation-only class=hard enforced=no tasks=display-template title="No template rule, condition or device state changes what a field stores, validates, copies, calculates, encodes or sends" --> **Presentation only, a layer apart from business logic.** Template rules are
  not on the same level as the App's business logic and never reach it. No
  rule, condition, or device state changes what a field stores, its validation
  rules, `CopyFrom` or `CopyTo`, calculation requests, encoding, or the
  [AI Import](llm-document-transformation.md) and
  [AI Search](protocols.md#ai-navigation-and-ai-search) contracts; none makes a
  field Hidden or read-only or changes a user's submitted payload. No rule
  reaches an [AI description](#ai-tool-call-descriptions). <!-- rule: tpl.withheld-row-unchanged class=hard enforced=no tasks=display-template --> A rule that
  withholds a row, such as [`when_single_choice`](#single-choice-rows), leaves
  the value and every rule that reads it exactly as if the row were shown. The
  [pane arrangement](#master-detail-panes) may load or refresh a leading page
  through a declared pure navigation Request. A condition may
  change style, layout, fold state, or existing page-control captions only.
  Hidden, ReadOnly, and every other schema rule apply before and regardless of
  the template.
- <!-- rule: tpl.forward-tolerant class=hard enforced=no tasks=client-runtime --> **Forward tolerant.** A client that meets a token value, layout, hint, or
  member it does not know uses the platform default for that item and applies
  the rest of the rule, so a newer template never breaks an older client.

### Template boundary

<!-- rule: tpl.boundary.tests class=judgment enforced=no tasks=display-template -->
An attribute belongs in the schema when changing it would change what the App
accepts, sends, computes, protects, or means. It belongs in the template when
changing it would only change how the same value is drawn on one device. Apply
three tests in order:

1. <!-- rule: tpl.boundary.wire class=judgment enforced=no tasks=display-template --> **Does the server or the wire depend on it?** Encoding, defaults,
   nullability, ranges, uniqueness, validation, copies, calculations, action
   sets, access, and identity are schema.
2. <!-- rule: tpl.boundary.meaning class=judgment enforced=no tasks=display-template --> **Does it state what the value is or means?** Money and currency, units,
   dates, formatted text, labels, help text, and which fields a chart encodes
   are schema; meaning is content, localized with the strings.
3. <!-- rule: tpl.boundary.device class=judgment enforced=no tasks=display-template --> **Would a different choice be right on another screen size?** Promotion,
   flattening, folding, inline choices versus a chooser, an inline
   description, density,
   arrangement, and chart appearance are template, with a source attribute,
   where one exists, as the default rule.

| Schema only | Template (a source attribute, where one exists, is the default rule) |
| --- | --- |
| Hidden, ReadOnly, Fixed, Default, Nullable, Size, Number ranges, Unique, Unordered | `promote` with `use_parent_name`, and `flatten` with `name_row` |
| ValidationRules, RangeMin, RangeMax, EnumSource, ConstrainedBy, ActionControl | Foldable and `initially_open` |
| CopyFrom, CopyTo, CalculationRequest, Link, LinkOnly, RequestConfirmation | InlineChoices |
| NextRequest, NextResponse, PrevRequest, WorkflowRequest, DefaultRequest, ExcludeUI | SuppressLabel, UIIncludeDescription |
| <!-- rule: template.boundary.source-attribute-roles class=hard enforced=no tasks=display-template --> AccessPrivilege, CacheIdentity | `Format` display pattern and precision, never the value or currency |
| Money, PhysicalUnit, DateOnly, FormattedText, display names, descriptions | Chart scale type, guide position, palette, and preview `chart` sizing |
| MimeTypes | Inline `media` size |
| Chart channel bindings: which field is x, y, color, or tooltip | Template-only concepts: layout, density, emphasis, `label`, `title`, `button_labels`, `action_controls`, `when_single_choice`, device conditions |

<!-- rule: tpl.boundary.unordered-excludeui class=hard enforced=no tasks=display-template title="Unordered and ExcludeUI stay in the schema; `Format`, `label` and `title` are template" -->
Unordered and ExcludeUI look presentational, but the wire and the protocol
depend on them, so they stay in the schema. `Format` looks semantic, but
`Money` already carries the currency and the precision that matter to the
value, so the display pattern moves. [`label` and `title`](#labels-and-titles) look like
schema labels, but the node's display name stays its meaning in the schema;
they change only what a person sees, so they are template. <!-- rule: tpl.boundary.catalog-only class=hard enforced=yes tasks=display-template title="A rule member outside the catalog, including every schema-only attribute, is rejected" --> Studio enforces the table: a rule
member that is not in the [catalog](#display-template-catalog), including every
schema-only attribute, is rejected.

### Display Template file

<!-- rule: tpl.file.source-of-truth class=hard enforced=no tasks=display-template -->
The source of truth is `display.json` at the project root, beside
`Cargo.toml` or `package.json`, under version control. There is one file per
project; device variance is expressed inside it as conditions, never as a
second template for a locale or platform. <!-- rule: tpl.file.missing-valid class=hard enforced=no tasks=display-template --> A missing file is valid and compiles
to a template holding only the rules derived from source attributes. Studio's
editor writes this file in place through the same validated writer as a hand
edit, never a second copy. <!-- rule: tpl.file.hub-compiled-only class=hard enforced=no tasks=client-runtime --> The Hub stores only the compiled, published copy
and never accepts a template Studio did not compile from source. <!-- /rule --> For the
conversational loop of hand-editing `display.json` and watching it live in
Studio, see [the Studio MCP live-preview guide](../guides/studio.md#live-preview-while-editing-displayjson).

```json
{
  "DisplayVersion": 1,
  "rules": [
    { "select": "^<Hub.topics>[]", "layout": "rows", "style": { "emphasis": "primary" } },
    { "select": "^<Hub>",
      "right_pane": ["EditWages", "EditInterest"],
      "detail": ["^<Hub.told>", "^<Hub.topics>[]"], "refresh": "OpenHub" },
    { "select": "^<Review.return_lines>", "layout": "table",
      "columns": [{ "value": "label", "header": "RETURN_LINE" }, "amount"] },
    { "select": "^<IncomeDocumentItem>", "layout": "card", "when": { "form": "phone" } },
    { "select": "^<Answer>", "inline": true },
    { "select": "^<Business>.expenses", "fold": { "initially_open": false } },
    { "select": "$.estimate:pending", "style": { "emphasis": "muted" } },
    { "select": "^<TopicRow>:condition([status] is NeedsAttention)", "style": { "color": "error", "icon": "warning" } },
    { "select": "^<Employee>", "layout": "split", "direction": "row",
      "regions": [
        { "fields": ["photo"], "ratio": 1, "style": { "align": "center" } },
        { "fields": ["name", "date_of_birth", "employee_id"], "ratio": 2, "layout": "form-grid", "columns": 2 }
      ] }
  ]
}
```

The example is schematic: its names stand for an App's own variants, types,
and fields. <!-- rule: tpl.file.display-version class=hard enforced=yes tasks=display-template title="`DisplayVersion` is 1 and `rules` is an ordered array" --> `DisplayVersion` is the JSON format version, like a document's
`DocumentVersion`; version 0.1 of the template is `DisplayVersion` 1, and the
number never chooses a winner in convergence. `rules` is an ordered array.

A rule is one JSON object with these members:

| Member | Required | Meaning |
| --- | --- | --- |
| `select` | Yes | One [selector](schema.md#display-template-selectors), optionally ending in a pseudo-state or a `:condition(...)` [data condition](#display-template-conditions). |
| `when` | No | A [presentation condition](#display-template-conditions) on the pane's size class and orientation, the device's form, pointer, or theme, or edit/view mode. Without it the rule applies everywhere. |
| `layout` | No | One [catalog layout](#display-template-catalog), with the layout-specific members its row names. |
| `right_pane`, `refresh`, `detail` | No | Leading Response relationships, its pure loader, and optional linked-node highlighting under [master-detail panes](#master-detail-panes). |
| `style` | No | An object of [style tokens](#style-tokens). |
| `label` | No | Replaces the node's displayed name and keeps its value, on a scalar field too ([labels and titles](#labels-and-titles)): `{ "field": "path" }` takes the text from a String or FormattedText field below the node, or below the struct that holds a scalar field, and a string names a fixed `StringResources` key. |
| `title` | No | Replaces the node's whole row, label and value, with its text, in the same two forms; never on a scalar, whose row is its value ([labels and titles](#labels-and-titles)). |
| `button_labels` | No | Localized captions for existing protocol-page controls, by slot: `go_back`, `next`, `close`, `actions` ([button labels](#button-labels)). |
| `action_controls` | No | Labels, placement, and applicable style for App-declared response controls and built-in Actions ([named action controls](#named-action-controls)). |
| `flatten` | No | Draws a struct's, List's, or Union's children at that node's own indentation ([flattening](#flatten)): `true`; `false`, which undoes a flattening that a lower-ranked rule set; or `{ "name_row": false }` on a struct to remove its name-only row as well. |
| `inline`, `fold`, `promote`, `when_single_choice`, `when_empty`, `when_embedded`, `suppress_label`, `include_description`, `format`, `scale`, `guide`, `palette`, `media`, `chart` | No | [Behavior hints](#behavior-hints). |
| <!-- rule: tpl.file.lock class=hard enforced=no tasks=display-template --> `lock` | No | `true` or, on a Union, `"variant"`: the person cannot change the value, while every other writer still can ([presentation locks](#presentation-locks)). |
| <!-- rule: tpl.file.switch-exclusive class=hard enforced=yes tasks=display-template title="A rule with `switch` has no other presentation member" --> `switch` | No | A [conditional switch](#display-template-switch) whose chosen case supplies the presentation members. A rule with `switch` has no other presentation member. |

<!-- rule: tpl.file.rule-shape class=hard enforced=yes tasks=display-template title="A rule has exactly one `select` and at least one presentation member; any other member is rejected" -->
A rule has exactly one `select` and at least one presentation member, directly
or through its switch. Members it does not name keep their inherited or
default values. Studio rejects any other member.

### Compiled template

Studio compiles one template document per package, in this order:

1. **Defaults:** one rule per occurrence of a source presentation attribute,
   at the lowest specificity. A field attribute becomes a rule selecting that
   field: `^<Type>.field` for a field of a named type, the strict path of each
   occurrence for a field of an inline type, or `^<function>.parameter` for a
   function parameter. A type-declaration attribute becomes a relaxed-type rule,
   `^<Type>`. <!-- rule: tpl.compiled.field-over-type class=hard enforced=no tasks=display-template --> Type-derived defaults precede field-derived ones, so a field's
   own attribute wins where both set the same hint.
2. **Rules:** the rules of `display.json`, validated, in file order.

```json
{ "DisplayVersion": 1, "defaults": [], "rules": [] }
```

That is the empty template. <!-- rule: tpl.compiled.direct-lookup class=hard enforced=no tasks=display-template --> Each derived rule reproduces its source
attribute's meaning exactly, including direct declaration lookup: a Foldable
on a field yields a field `fold` rule and one on a type yields a type `fold`
rule, and neither is inherited through effective-attribute resolution. <!-- rule: tpl.compiled.never-edited class=hard enforced=no tasks=display-template --> The compiled template is derived and never edited by hand. Its
identity is the SHA-256 digest of its canonical JSON, computed with the same
canonicalization as the [schema revision digest](schema.md#113-revision-digest).
`compile_schema` with `includeArtifacts` returns it beside each package's
default document.

### Display Template cascade

A rule applies to a node when its selector matches the node, its `when` holds,
its data condition or pseudo-state holds, and, for a switch, a case or `else`
is chosen. <!-- rule: tpl.cascade.member-resolution class=hard enforced=no tasks=display-template --> Every presentation member resolves independently: the
highest-ranked applicable rule that sets that member wins, and a layout
resolves together with its layout-specific members. <!-- rule: tpl.cascade.button-slot-independent class=hard enforced=no tasks=display-template --> Each `button_labels` slot
resolves independently; setting one slot leaves the others' cascade intact. <!-- /rule -->
Ranks, lowest first:

1. the platform default;
2. a rule of another package's compiled template, on a node that package
   declares; among these rules, that template's own ranks and order decide;
3. a rule derived from a source attribute (`defaults`);
4. a relaxed-type rule: a selector whose first form is `^<Type>`, or any
   selector using `...`;
5. the same with a `when` condition;
6. a strict-path rule: `^<Variant>`, `^<Variant.field...>`, `^<function...>`,
   or `$.field`, with or without `[]`;
7. the same with a `when` condition.

<!-- rule: tpl.cascade.later-wins class=hard enforced=no tasks=display-template -->
Among rules of equal rank, the later rule in compiled order wins. <!-- rule: tpl.cascade.conditions-gate class=hard enforced=no tasks=display-template --> Data
conditions, pseudo-states, and switches gate a rule; they never change its
rank. <!-- rule: tpl.cascade.inheritance class=hard enforced=no tasks=display-template --> The style tokens `density` and `emphasis` inherit down the value tree
unless a descendant resolves its own; no other member inherits, and a layout
never does.

<!-- rule: tpl.cascade.dependency-templates class=hard enforced=no tasks=display-template -->
A screen uses the full template of the package that owns its displayed root:
the Endpoint protocol Union's package for a protocol screen, and the function's
package for a configuration editor or stored data. Nodes declared by another
package additionally receive that package's compiled template, its
`display.json` rules included, at rank 2, below every rule of the screen's own
package. A dependency's rule applies where its selector, resolved against that
package's schema, matches a node that package declares. A type therefore keeps
its authored presentation wherever it is shown, the using package can still
change it, and a TypesOnly package may ship a `display.json` for its types.
Every package template a screen needs arrives with its schema.

### Display Template conditions

<!-- rule: tpl.when.keys class=hard enforced=yes tasks=display-template title="`when` accepts only the closed keys and values; a list means any, several keys must all hold" -->
A `when` object accepts only these keys, each with a closed value set. A value
may be a list, meaning any of the listed values (`"form": ["tablet",
"desktop"]`); several keys in one `when` must all hold.

| Key | Values | Meaning |
| --- | --- | --- |
| `width` | `compact` (under 600 dp), `medium` (600 dp up to 840 dp), `expanded` (840 dp up to 1200 dp), `large` (1200 dp up to 1600 dp), `extra-large` (1600 dp and up) | How much fits across the pane now: the platform's window size class. |
| `height` | `compact` (under 600 dp), `medium` (600 dp up to 840 dp), `expanded` (840 dp and up) | How much fits down the pane now. |
| `orientation` | `portrait`, `landscape` | The pane's orientation. |
| `pointer` | `coarse`, `fine` | Touch, or a mouse or trackpad. |
| `theme` | `light`, `dark` | The active color theme. |
| `form` | `phone`, `tablet`, `desktop`, `tv` | What the device is, not the window: `phone` when its smallest width in portrait is under 600 dp; `tablet` at 600 dp and up with a coarse pointer; `desktop` with a fine pointer and a resizable window; `tv` for a lean-back display. |
| `mode` | `edit`, `view` | Whether the person is entering data in the node. See [Editing and viewing](#editing-and-viewing). |

<!-- rule: tpl.when.size-classes class=hard enforced=no tasks=display-template title="Conditions use size classes and a form factor, never pixels" -->
Conditions use size classes and a form factor, never pixels. A 2160 px phone
in landscape is about 800 dp wide (`width: medium`, `height: compact`,
`form: phone`); a 32-inch monitor is `width: extra-large`, `height: expanded`,
`form: desktop`. A rule for 840 dp and wider lists
`["expanded", "large", "extra-large"]`. <!-- rule: tpl.when.widen-by-form-or-height class=judgment enforced=no tasks=display-template --> A rule that widens a layout reads
`form`, or `width` together with `height`, never `width` alone. <!-- rule: tpl.when.no-condition-for-fit class=judgment enforced=no tasks=display-template --> Pane count,
split lines, `form-grid` columns, and table fit need no condition: they follow
the [width-based layout](#width-based-layout) rules at every size.

<!-- rule: tpl.when.container-semantics class=hard enforced=no tasks=display-template -->
Conditions have container semantics: `width`, `height`, and `orientation` are
evaluated against the nearest enclosing pane, not the window, while `form`,
`pointer`, and `theme` describe the device. A list rendered
in the left pane of a `master-detail` layout on a tablet sees that pane's width
class.

#### Editing and viewing

<!-- rule: tpl.mode.client-decides class=hard enforced=no tasks=client-runtime -->
The client decides `mode` for each node from what it already renders; the App
supplies no flag and the protocol carries none. A node is in `edit` mode when it
lies in a Request the person is composing (a new Request, an EditRequest reply,
a WorkflowRequest editor, or a `master-detail` editing pane) and it, or a
descendant, holds a field the person can change: not ReadOnly, Hidden, or
Fixed. Every other node is in `view` mode: Responses, SubscriptionData, a
submitted Request shown read-only, and a Request subtree with nothing the
person can change. <!-- rule: tpl.mode.first-and-reopen-both-edit class=hard enforced=no tasks=display-template --> Composing a Request for the first time and reopening it to
review or change it are both `edit`; a rule that must tell them apart reads the
[interaction reference context](protocols.md#interaction-reference-context)
with `:condition`.

<!-- rule: tpl.mode.obligations class=judgment enforced=no tasks=client-runtime -->
The two modes have different obligations. In `view` the person browses, so
grouping that hides content until chosen, such as `tabs`, costs nothing, and
Studio warns about dense view screens. In `edit` the person must find and
answer every required field and see every error, so an editor is a scrolling
form that keeps its fields reachable in entry order: no density warning
applies, and titled `sections` under `rows` may group the fields under
headings. <!-- rule: tpl.mode.no-tabs-in-edit class=hard enforced=yes tasks=display-template title="An editable Table is never laid out as `tabs` while `edit` may apply" --> An editable item is never hidden: a Table that holds a field the
person can change is never laid out as `tabs` while `edit` may apply, and
Studio rejects such a rule. A Union's branch strip is not a grouping: choosing
the branch is the answer, and the chosen branch's fields are shown. <!-- rule: tpl.mode.unset-applies-both class=hard enforced=no tasks=display-template --> A rule
without `mode` applies in both modes.

<a id="unanswered-fields"></a>

<!-- rule: field.unanswered class=hard enforced=no tasks=client-runtime,field-presentation,data-model -->
**Unanswered fields.** A field the person answers that no `Default`, `Fixed` or copy initializes starts
[undefined](schema.md#ordinary-initialization) (unanswered), and a non-Nullable field never holds
null. When an editor opens, an undefined field shows only its neutral empty
state: its placeholder, such as an unselected choice or "Choose date…", never
an error look; an undefined Nullable's text uses the secondary text color.
When Next or Submit is blocked, the client marks the first field that blocks
it: below that field, an error icon and the missing-answer or validation
message, in the platform's error color role and announced politely to
assistive technology. It opens any fold or summary page holding the field and
scrolls it into view; it does not move focus. The mark clears once that field
passes, and it does not move to another field until the next blocked Next or
Submit. An undefined field is reported as a missing answer, never as a null
value.

<!-- rule: tpl.condition.pseudo-states-style-only class=hard enforced=yes tasks=display-template title="`:pending`, `:error` and `:selected` change style only; `:condition(expression)` style, layout, fold state or existing page-control captions" -->
A selector may end in one runtime state or one data condition:

| Condition | Holds when | May change |
| --- | --- | --- |
| `:pending` | The node owns an operation still awaiting its reply and shows the existing pending indicator: a submit, a link, or a calculation. | Style only |
| `:error` | The node currently shows validation or submission feedback. | Style only |
| `:selected` | The node is the selected member: the member shown in a `master-detail` detail pane, or the current `tabs` section. | Style only |
| `:condition(expression)` | The expression, evaluated on the displayed value, is true. | Style, layout, fold state, or existing page-control captions |

<!-- rule: tpl.condition.expression class=hard enforced=yes tasks=display-template title="A data condition is a Boolean ValidationRules expression reading only the selected node and the reference context" -->
A data condition uses the [ValidationRules expression language](validation-rules.md)
rooted at the selected node: `[]` is the node, `[status]` a child, curly
references read the [interaction reference context](protocols.md#interaction-reference-context),
and the same `|`, `is`, and comparison operators apply. The expression must be
Boolean and reads only the selected node and the reference context. Absent
data follows the expression language's null rules, and a condition that does
not yield true does not apply.

<!-- rule: tpl.condition.locale-independent class=hard enforced=yes tasks=display-template title="A data condition or switch case never inspects text; compare a resource key or branch on typed data" -->
**Conditions are locale independent.** Display text is localized, so a data
condition or switch case never inspects text: it may not compare a String or
FormattedText value with a string literal or another text value, nor apply a
string operation to one. It branches on locale-independent data instead: an
Enumeration or Union (`is`), a Boolean, a Number, a date, a null check, or a
FormattedText's resource key from
[`resource(...)`](validation-rules.md#131-string-resource-references), for
example `resource([status]) == "STATUS_NOT_STARTED"`, which compares the key
rather than its localized text. Studio rejects a condition that reads text any
other way. <!-- rule: tpl.condition.status-badge class=judgment enforced=no tasks=display-template -->
`^<TopicRow>:condition([status] is NeedsAttention)` with `color` and `icon`
declares a status badge; no schema attribute is needed.

<!-- rule: tpl.condition.style-layout-fold-only class=hard enforced=yes tasks=display-template title="A data condition may change style, layout, fold state or existing page-control captions only, never hide, lock or change a value" -->
A condition is re-evaluated whenever its inputs change. It may change style,
layout, fold state, or the captions of existing page controls only. It can
never hide a field or control, make anything read-only, change a value, or
change what is sent; those are schema.

<!-- rule: tpl.condition.recheck-combined class=judgment enforced=no tasks=display-template,data-model title="Re-check a condition that combines several fields whenever it, or the data it reads or should read, changes" -->
**Re-check a combined condition when it or its data changes.** The client
evaluates a condition on the value being displayed or edited, including rows
added or removed before any save, so nothing on the server keeps it correct.
A condition that combines several fields names each one: a page-is-empty test,
for example, counts every List of the page. When you change such a condition,
or add, remove, rename, or retype a field or List that it reads or should
read, read the whole condition again against the current types, update it, and
view each outcome in the Display Template preview. Studio checks that every
path resolves and that the expression is Boolean; it cannot tell that a newly
added field belongs in the condition. <!-- /rule -->

### Display Template switch

A rule may carry a `switch` instead of a `:condition(...)` on its selector: an
ordered `cases` list, each with a `condition` (a Boolean ValidationRules
expression rooted at the selected node, with the same reads as a data
condition) and its presentation members, and an optional `else`. <!-- rule: tpl.switch.first-true-case class=hard enforced=no tasks=display-template --> Cases are
evaluated in file order; the first true case applies and evaluation stops, so a
later case never repeats an earlier test. When no case is true, `else`
applies, and without `else` the rule does not apply. `when` stays at the rule
level and gates the whole switch. `:condition(...)` on the selector is the
one-case shorthand.

```json
{ "select": "^<TopicRow>", "switch": {
    "cases": [
      { "condition": "[status] is NeedsAttention", "style": { "color": "error", "icon": "warning" } },
      { "condition": "[status] is Done",           "style": { "color": "primary", "icon": "check" } }
    ],
    "else": { "style": { "emphasis": "muted" } } } }
```

<!-- rule: tpl.switch.shape class=hard enforced=yes tasks=display-template title="A switch has at least one case, at most one `else`, Boolean conditions and no empty case; not beside a `:condition` selector" -->
A switch has at least one case and at most one `else`; every case has a
Boolean condition, and no case or `else` lacks a presentation member. A rule
cannot have both a `:condition(...)` selector and a `switch`. Cases and `else`
carry the same members as a rule except `select`, `when`, and `switch`.

### Display Template catalog

<!-- rule: tpl.catalog.closed class=hard enforced=yes tasks=display-template title="The catalog is closed: no free-form CSS, pixel value, literal color or font" -->
The catalog is closed: every renderer implements each layout, token, and hint
below. There is no free-form CSS, pixel value, literal color, or font. <!-- rule: tpl.catalog.layout-node-kind class=hard enforced=yes tasks=display-template title="A layout applies only to the node kinds its catalog row allows" --> Studio
rejects a layout on a node kind its row does not allow.

<!-- rule: tpl.catalog.sequential-screens-are-workflow class=judgment enforced=no tasks=display-template -->
Sequential screens are a workflow, not a layout. A step-by-step layout would
add a second Previous and Next inside screens that already move through a
workflow's Previous and Next, which confuses users; an App that wants
sequential screens splits the request into a
[workflow](protocols.md#workflow-interactions), where the service validates,
branches, and saves each step. `tabs` is how one request is split into
sections. <!-- rule: tpl.catalog.no-stepper class=hard enforced=yes tasks=display-template title="`stepper` is not a layout: Studio rejects it and clients draw `rows`" --> `stepper` is not a Display Template layout: Studio rejects it, and a
client that meets it draws `rows` under the forward-tolerant rule.

| Layout | Applies to | Behavior |
| --- | --- | --- |
| `rows` | Any node | The default: one row per child. On a Table, optional `sections` stack each section under its title on the same page, in order, each drawn by its own [group](#layout-groups) layout. |
| <!-- rule: layout.form-grid class=hard enforced=yes tasks=display-template title="`form-grid` on a Table flows fields into at most `columns` (1 to 4) per row; `span` takes more columns" --> `form-grid` | Table | Fields flow in declaration order into at most `columns` (1 to 4) per row, each as label and value; the [width-based layout](#form-grid-columns) decides how many fit. A field's `span` token takes more columns. `columns: 1` is the label-and-value form. |
| `card` | Table, Union, or List member | A bordered block titled by the node's header text. On a Union it holds the Union's variant choice and selected payload as the Union draws them. |
| <!-- rule: layout.table class=hard enforced=yes tasks=display-template title="`table` on a List of Tables: scalar `columns` in order and an optional display-only `sort` by one column" --> `table` | List of Tables | One row per member; `columns` lists the columns shown, in order (see **Table columns** below), each a scalar value below the member with an optional header; an optional `sort` names one column's `value` and orders the rows ascending by it for display only, never changing the List. Shows as a table only while its [columns fit](#table-column-fit); otherwise `rows`. |
| <!-- rule: layout.summary class=hard enforced=yes tasks=display-template title="`summary` on a List of Tables or Unions: previews from `paths`, `arrangement` `list` (default), `table` or `grid`" --> `summary` | List of Tables or Unions | Read-only item previews from ordered member-relative `paths`; opening a preview shows the complete item. `arrangement` is `list` by default, `table`, or `grid`, which flows the previews as the [`grid`](#summary-grid) layout does; a table whose [columns do not fit](#table-column-fit) becomes the same projected list. See **Summary array items** below. |
| `tabs` | Union, or Table with `sections` | One tab per Union branch or section. |
| `master-detail` | Response | `right_pane` lists the editing Request variants that belong beside this leading Response; declaring it implies this layout. Required `refresh` loads and refreshes the leading page. Optional `detail` selectors name linked nodes eligible for selection highlighting; a selector ending in `[]` names List members. Membership depends on the current editing Request, including an editor opened directly through Go to or AI Search. Optional `panes` sizes the two panes. Shows two panes only at or above the [pane threshold](#pane-threshold); otherwise `rows`. See [master-detail panes](#master-detail-panes). |
| `accordion` | Table or List | Folding sections with the [folding](#folding-structs-and-unions) behavior and `initially_open`. On a Table with `sections`, each section folds as one [group](#layout-groups) under its title; otherwise each child folds. |
| `categorized` | Classifier Table | Only the item Lists that hold a member, plus one control that adds any kind; see [categorized layout](#categorized-layout). |
| `split` | Table | Two or three `regions` ([groups](#layout-groups)) in a `direction`, `row` or `column`; a `row` region may declare a `min_width`. See below. |
| <!-- rule: layout.grid class=hard enforced=yes tasks=display-template title="`grid` on a List of Tables or Unions flows whole members in List order into cells using the member's layout and opening the item; optional `columns` maximum and `min_width`" --> `grid` | List of Tables or Unions | Members flow into cells, in List order, as many per line as fit; each cell draws the whole member with the member's own layout and opens it. Optional `columns` (1 to 8) is a maximum and `min_width` a cell-width token. See [grid](#summary-grid). |

<a id="categorized-layout"></a>

<!-- rule: layout.categorized class=hard enforced=yes tasks=display-template title="`categorized` applies to a classifier Table, draws populated category Lists, and places one horizontal AI Import / Manually add document row before classifier headings and content, sticky below the page header while content scrolls, including when flattened" -->
**Categorized layout.** `categorized` applies to a
[classifier](llm-document-transformation.md#categorized-import) Table,
including one whose categories are classifiers themselves, and suits a
classifier with many categories of which a person usually has few. It draws:

- each item List under the classifier that holds at least one member, in
  declaration order and under its category's title when categories nest, as
  that List draws by its own rules. A List with no member is not drawn, unless
  it carries a validation message or is the target of a reveal or an issue
  link, which shows it;
- each single-document position (a selected field that is not a List) and
  the classifier's fields that are not import positions, as ordinary rows;
- one horizontal action row containing **AI Import** and **Manually add
  document**, before the classifier's heading and all classification content,
  owned by the outermost categorized classifier. The row remains available
  when flattening removes that classifier's name row, and stays below the page
  header while the form content scrolls.

  <!-- rule: layout.categorized.no-descendant-actions class=hard enforced=no tasks=client-runtime -->
  Descendant classifiers,
  forms and category Lists do not repeat import or membership Add controls.
  The manual chooser always shows a search box and the classification hierarchy,
  with nested category headings in declaration order and localized kind names
  beneath them. Searching keeps the ancestors of matching kinds visible. It
  offers every kind that a List under the classifier holds. Choosing a kind
  appends a member to the List whose import position
  holds that kind, with that kind selected (its form, or that variant of the
  item's marked Union) and the member's other fields at their normal
  initialization; the new member opens, as every value the person creates
  does. Cancelling adds nothing.

  <!-- rule: chooser.category-vs-leaf class=hard enforced=no tasks=client-runtime title="In a hierarchical chooser, category rows are section headers styled apart from selectable leaf rows" -->
  In this chooser, and in every chooser that shows a hierarchy, a **category
  row** is a section header, not an option: a smaller title-style label in a
  distinct, secondary or accent color and stronger weight, extra space above
  it, no press or selection state, never focusable as a choice, and announced
  to assistive technology as a heading. A **leaf row** is a selectable option:
  body-style text, a full-width target of at least the platform minimum (48 dp
  on Android, 44 pt on iOS), the ordinary press and focus states, and its
  indentation under its category drawn by layout, never by spaces in the text.
  A nested category is indented like its parent's leaves and keeps the header
  style. Long lists may pin the current category header while scrolling.

Hidden Lists and kinds are neither drawn nor offered, and a read-only
classifier or List offers no manual Add. The outer classifier's
[categorized import](llm-document-transformation.md#categorized-import) covers
all its kinds. Existing documents keep ordinary editing and removal controls;
Lists inside a document keep their ordinary membership controls. The layout is
presentation only:
data, encoding, paths, validation, the walk and AI Import are the same as
with `rows`, and a client without it draws `rows`.

<!-- rule: layout.row.label-value-fit class=judgment enforced=no tasks=client-runtime -->
In ordinary label-and-value rows, every client measures the rendered label
and value, including icons and controls, using the current font and localized
text. It keeps both beside one another when their natural widths and the
platform's label-value gap fit the available row width; otherwise the value
goes below the label. An editable row's trailing link button counts as part of
the value; when value and button do not fit beneath the label either, the
button moves below the value at its trailing edge. The available width belongs
to the actual pane, region, or `form-grid` cell. This client-owned arrangement
needs no additional template rule.

**Table columns.** Each `columns` entry is `{ "value": path, "header": text }`,
with `header` optional; a bare path string is shorthand for
`{ "value": path }`:

```json
"columns": [
  { "value": "label", "header": "RETURN_LINE" },
  "amount",
  { "value": "explanation", "header": { "field": "explanation_heading" } }
]
```

- <!-- rule: layout.table.column-value class=hard enforced=yes tasks=display-template title="A column `value` is a struct-field path, typed path or array of paths ending at a scalar; no two columns share one" --> `value` is a [Libertas Schema Path](schema.md#libertas-schema-path) of field
  native names relative to the member, through struct fields only, never
  through a List or a Union, as a [`label` source](#labels-and-titles) is; or a
  [typed path](#typed-paths); or an array of such paths, whose first path that
  resolves for the member supplies the cell. It ends at a scalar field (not a
  Table, List, or Union); a FormattedText field is a scalar. No two columns
  share a `value`.
- <!-- rule: layout.table.hidden-value-source class=hard enforced=no tasks=display-template --> A Hidden field remains a valid `value` source, as it is for `label` and
  `title`: the column shows its value read-only, while the field still has no
  row of its own. Hidden stays a schema attribute that the template never sets
  or overrides.
- <!-- rule: layout.table.column-header class=hard enforced=yes tasks=display-template title="A column `header` is a `StringResources` key or a field below the struct that holds the List" --> `header` is a package [`StringResources`](#string-resources-and-formatted-output)
  key, or `{ "field": "path" }` read, like a `label`, through struct fields
  only below the struct that holds the List, ending at a String or
  FormattedText field that may be Hidden. The field form is invalid where the
  List has no holding struct or is reached only through a Union. Without a
  header, or when its text is null, unanswered, or empty, the column is headed
  by the value field's localized display name.
- <!-- rule: layout.list.add-caption class=hard enforced=no tasks=client-runtime title="A List's Add control is captioned Add plus the member's display name, in every layout" --> A List's Add control reads "Add" followed by the member's localized display
  name (for example "Add Person"), in every layout, table mode included, on
  every client. A member type with no display name falls back to "Add to"
  plus the List's label.
- <!-- rule: layout.table.empty-no-header class=hard enforced=no tasks=client-runtime title="A table with no members draws no column header row" --> A table draws its column header row only while its List has at least one
  member. With no members it shows the List's count and its Add control (when
  [offered](schema.md#52-lists)) and no header row; the header returns with the
  first member. This changes no value, validation or encoding.

<a id="summary-array-items"></a>

<!-- rule: layout.summary.when class=judgment enforced=no tasks=display-template -->
**Summary array items.** Use `summary` when a List needs brief item previews
and a separate complete item editor or viewer:

```json
{
  "select": "^<SaveIncomes>.items",
  "layout": "summary",
  "paths": [{"path": "document", "show": "variant"}, "owner", "document.W2.wages", "details"],
  "arrangement": "list"
}
```

<!-- rule: layout.summary.paths class=hard enforced=yes tasks=display-template title="Summary `paths` are member-relative struct and variant paths that never cross a List or use an index" -->
`paths` is a non-empty ordered array of native-name dotted paths or explicit
variant projections. A string path is relative to each member and traverses
struct fields and explicitly named Union
variants, such as `document.W2.wages`; for a Union member it begins with its
variant name. It may end at a scalar, struct, Union, or List, but never crosses
a List, uses an index or wildcard, or names a root or reference object. Studio
validates every path against the member schema, including inactive variants.

<!-- rule: layout.summary.variant-projection class=hard enforced=yes tasks=display-template title="A `show: variant` projection selects a Union and contributes its variant name; only `path` and `show` are accepted" -->
`{"path": "document", "show": "variant"}` selects the Union at `document`
and contributes one passive field: its ordinary field label and the localized
display name of its selected variant, such as **Document — W-2**. It does not
expand that variant's payload. For a Union-valued List member,
`{"show": "variant"}` selects the member itself. Only `path` and `show` are
accepted in these objects; an explicit `path` must be a non-empty valid path,
and its target must be a Union. An unanswered Union has an empty variant value;
a selected Nil branch still displays its variant name. This presentation
operation does not add a Schema Path operator or reserve native identifiers:
`document.$variant` continues to mean a real field or variant named `$variant`.

<a id="typed-paths"></a>

**Multi-morphism table display (typed paths).** Members of one List can hold
different shapes: an item Table
may hold one marked Union of forms, or one marked form, beside fields such as
an owner. A **typed path** reads a field by its type, wherever that type sits in
the member, so one column or summary path serves every shape:

```json
"columns": [
  { "value": ["^<W2>.employer.name", "^<Form8919>.firm.name", "^<Patr1099>.payer.name"],
    "header": "PAYER" },
  "owner"
]
```

<!-- rule: layout.typed-path class=hard enforced=yes tasks=display-template title="A typed path `^<Type>.rest` reads the nearest descendant of that type through struct fields and selected variants, never a List" -->
`^<Type>.rest` starts at the nearest descendant of the member whose effective
type is `Type`, reached through struct fields and the **currently selected**
variant of each Union, never through a List, and then follows `rest` through
struct fields. When the member holds no such descendant, for example because
another variant is selected, the path does not resolve: a column cell is empty
and a summary path contributes no field. A table column `value`, and each entry
of a summary `paths` array, may be a typed path or an array of plain and typed
paths whose first resolving path is used. Studio validates that each `Type` is
reachable from the member through fields and Union variants without crossing a
List, that each `rest` exists on `Type`, and that a column's paths end at a
scalar. An array holding one path is the same value as that path, and an array
may not repeat a path. <!-- rule: layout.typed-path.header-and-presentation-only class=hard enforced=no tasks=display-template --> Without a `header`, an array-valued column is headed by the
localized display name of its first path's field, whatever the rows hold; the
same holds for a summary `arrangement: "table"` column built from an array entry. The
`path` of a `{"path": …, "show": "variant"}` projection stays a plain path; a
typed or array path there is refused. A table's `sort` names a column by its whole `value`, so for a column
whose `value` is an array, `sort` is that same array; each row sorts by the value
its first resolving path supplies, and rows whose cell is empty sort last. A
typed path is presentation only: it reads a value and never changes what is
stored, validated, copied, calculated, encoded or imported.

A selected struct or active Union expands into its visible leaf descendants
in declaration order. A nested List stays one count boundary rather than
expanding its members. Paths contribute in authored order; when selections
overlap, each leaf appears only at its first occurrence. <!-- rule: layout.summary.hidden-and-inactive class=hard enforced=no tasks=display-template --> Hidden nodes and their
subtrees remain absent regardless of a path. Withholding an editor row through
`when_single_choice` does not withhold its selected summary value. A path into an inactive Union
variant contributes no list field and an empty table cell. <!-- /rule --> Ordinary localized
value formatting and the template cascade apply to the selected original
nodes. `arrangement: "list"` shows one ordinary label/value row per projected
field, with a clear boundary between array items. It does not join several
fields into a single compact line. `arrangement: "table"` uses aligned columns,
one per projected leaf or explicit variant projection in path and declaration
order, with item rows in List order, while its [columns fit](#table-column-fit).
Otherwise the same projection appears as the field list, including on narrow phones.
`arrangement: "grid"` draws [cards](#summary-grid) instead.
This differs from the scalar-only `columns`
of the `table` layout above.

<a id="summary-grid"></a>

**Grid.** The `grid` layout, and `summary` with `arrangement: "grid"`, flow a
List's members into cells in List order, for store-style listings. The grid
owns only the cells; each cell is a generic host that draws its content with
the layout the template gives the member. A plain `grid` cell holds the whole
member; a summary grid cell holds the member's `paths` projection:

```json
{ "select": "^<Catalog>.products", "layout": "summary", "arrangement": "grid",
  "paths": ["photo", "name", "price", "stock"], "min_width": "tile", "columns": 4 },
{ "select": "^<Product>", "layout": "split", "direction": "column", "regions": [
    { "fields": ["photo"], "ratio": 1 },
    { "fields": ["name", "price", "stock"], "ratio": 1, "layout": "rows" } ] },
{ "select": "^<Product>.photo", "media": { "max_height": "medium", "aspect": "square" } }
```

<!-- rule: layout.summary.grid-keys class=hard enforced=yes tasks=display-template title="Optional grid `columns` (integer 1 to 8, a maximum) and `min_width` (width token, default `tile`) apply only to `grid` or summary `arrangement: grid`" -->
`columns` and `min_width` are accepted on `grid` and, under `summary`, only
with `arrangement: "grid"`; both are optional. `columns` is an integer from 1
to 8, the most cells on one line. `min_width` is a
[width token](#width-based-layout), `tile` by default. Studio rejects any
other value.

<!-- rule: layout.summary.grid-render class=hard enforced=no tasks=client-runtime title="A grid flows cells into the columns its actual container width holds, recomputed on every width change" -->
**Columns.** The column count is as many cells at `min_width` plus a gutter
between each pair as fit the grid's actual container width (its pane, region
or cell, not the screen), at least 1 and at most `columns`; the window cap on
split lines and `form-grid` columns does not apply. The client recomputes it on
every width change, including rotation, window or pane resize, split screen and
text-size changes, stretches cells evenly so each line fills the width with
equal gutters, aligns the cells of a line to its tallest, and keeps the scroll
position on the same item across a reflow.

<!-- rule: layout.summary.grid-cell class=hard enforced=no tasks=client-runtime,display-template title="A grid cell is a bordered host drawing its member content with the member's own template layout" -->
**Cells.** Each cell is one bordered block drawn by the layout the template
cascade selects for the member (`rows` by default, or any layout the catalog
allows for the member's kind) with its ordinary labels, `suppress_label`,
`format` and [`media`](#inline-media-size) rules; Hidden nodes stay absent.
The grid adds no title, line limit or image slot of its own: a title comes
from a `card` layout's header, an image from a media field.

<!-- rule: layout.summary.grid-behavior class=hard enforced=no tasks=client-runtime title="A grid cell is a passive preview that opens the complete item, in editable and read-only Lists; Add follows the grid; an unsupported grid draws as rows or list" -->
**Behavior.** A cell is a passive preview whatever layout it hosts, in an
editable Request as in a read-only Response: the whole cell is the one target
that opens the complete item under the passive-preview and item-editing rules
below, nothing in it is an inline editor or a separate link, and it is
announced as one button named by its values and their labels. Keyboard focus
moves cell by cell in List order. Media in cells load as the cell scrolls into
view, from the value the client already holds. The List's own Add control
follows the grid; an added member opens in the item editor, and Commit returns
to the grid at that member. An empty List draws as any empty List does,
including under [`when_empty`](#behavior-hints). A client that does not
support `grid` draws a plain `grid` as `rows` and a summary grid as
`arrangement: "list"`.

<!-- rule: layout.summary.passive-previews class=hard enforced=no tasks=client-runtime -->
The previews are passive even in an editable Request: their values, labels,
and cells open the **whole item**, never an inline field editor or a link on
a selected descendant. The client opens a full-page item editor within its
working surface, including on a desktop, using the original complete parent
draft and the same interaction reference context. Fields outside `paths`
remain available there, with ordinary promotion, flattening, structural
indentation, automatic editing, validation, copying,
calculations, and links. A read-only Response instead opens a complete item
viewer. The projection neither marks schema fields ReadOnly nor changes the
stored or submitted value.

<!-- rule: layout.summary.item-editing class=hard enforced=no tasks=client-runtime -->
**Commit** validates the complete item through the ordinary validation owners,
including dynamic rules when their context is available, and waits for available
pending calculations. Errors keep the item editor open. Successful Commit
returns to the summary, preserving committed fields and the list position,
without implicitly submitting the parent Request or making a backend workflow
transition. Cancel, including local Back, restores an existing item to its
value before that item editor opened, while preserving the parent draft outside
the item. Invalid answers and changed constraints do not block cancellation.
The client tracks the original item
through list changes by stable identity, and nested summary Lists use the same
navigation. An item with validation errors shows an error indicator on its
preview even when the offending field is outside `paths`. Add initializes a
provisional item through the ordinary List lifecycle and opens its complete
editor. The normal automatic editing flow still applies: an unanswered promoted
Union opens its variant chooser, while defaults and automatically resolved single
choices retain their ordinary behavior. Opening an existing unanswered item uses
the same automatic chooser; cancelling it retains the existing item. Opening an
answered or defaulted item does not ask again. Choosing a variant does not commit the
item. Cancelling this initial required chooser, or Cancel including local Back
from the new-item editor, removes that exact provisional item; only a
successful Commit retains it and returns to the summary. The schema's membership,
add, delete, and reorder rules still apply. Editable Lists expose a visible
Delete button for each complete item, including compact items; the button
removes the item through ordinary List mutation and never opens its editor.
ReadOnly List membership suppresses Add and Delete without recursively making
members read-only. While any new item is provisional,
the client pauses ordinary durable draft saves; Commit or Cancel resumes them
through the normal draft owner, so recovery never restores an uncommitted new
item. The parent Request cannot be submitted while a provisional item remains.
Clients that do not
recognize `summary` use ordinary `rows` under the forward-tolerant rule.

<!-- rule: pane.title.minimum-content-width class=hard enforced=no tasks=client-runtime -->
**Content sizing in Studio and the web interface.** Every full-screen protocol
page, including Request and Response pages, uses an automatic preferred width.
Simple forms use their labels and ordinary input space; tables reuse their
measured column minimums, while multi-column forms and split layouts include
their column and pane requirements. The page always fits its available space,
including a narrow window or a resized master-detail pane. Each pane computes
its own content and control requirements. The parent
combines those widths with the divider gap and fits the arrangement within
the window; a changed pane page or layout recalculates the combined width.
Without a saved divider position, the two panes use their content-based widths
rather than allocating unused width from the `panes` ratios. A manual divider
position takes precedence; the editor fills the pane the viewer allocates.
Typing ordinary field values does not widen a form. Summary item editors and
read-only item viewers use the same automatic content sizing on their own full
screen, including the header and action row; they retain the complete parent
draft and their ordinary item navigation.
Short forms place their actions immediately below the content. Long forms
scroll the data while keeping the action row visible. Actions align with the
form's left edge and stay together: eligible Go back controls followed by
primary Next, or Close followed by Next in an embedded editor. The existing
[pane divider](#master-detail-panes) adjusts the master-detail allocation.
The top bar shares the content layout's width, with its buttons right-aligned
to the content inside the page's padding. The chosen arrangement of its
title and buttons supplies a minimum content width, capped by the available
window or pane space; controls follow the ordinary wrapping and placement rules
when that minimum cannot fit. A master-detail working surface retains its
shared top bar across the whole arrangement.

<a id="master-detail-panes"></a>

<!-- rule: master-detail.declaration class=hard enforced=yes tasks=display-template title="A leading Response rule declares `right_pane` (editing Request variants of the same protocol) and `refresh` (a pure navigation Request)" -->
**Master-detail panes.** A rule on a leading Response declares `right_pane`,
a non-empty array of editing Request variant native names from the same
protocol, and `refresh`, a pure navigation Request that loads that Response. <!-- rule: master-detail.implies-layout class=hard enforced=yes tasks=display-template title="`right_pane` implies `layout: master-detail`; an explicit layout must agree" -->
`right_pane` implies `layout: "master-detail"`; an explicit layout must agree.
For example:

```json
{
  "select": "^<Income>",
  "right_pane": ["Wages", "Interest", "InvestmentSales"],
  "refresh": "OpenIncome",
  "panes": { "trailing": { "min_width": "medium", "ratio": 3 } }
}
```

<!-- rule: master-detail.panes class=hard enforced=yes tasks=display-template title="`panes` takes only `leading` and `trailing` with a `min_width` token and a positive `ratio`, and never changes membership" -->
`panes` is optional. Its `leading` and `trailing` members are each optional
and accept only a `min_width` [size-class token](#width-based-layout) and a
positive `ratio`; an omitted member keeps its default,
`{ "min_width": "compact", "ratio": 2 }` for the leading pane and
`{ "min_width": "compact", "ratio": 3 }` for the trailing pane. `panes` sizes
the arrangement and never changes membership. A device condition is not needed:
the [pane threshold](#pane-threshold) decides when two panes show.

<!-- rule: master-detail.membership class=hard enforced=no tasks=client-runtime -->
The **current editing Request** determines membership. Resolve a
[WorkflowRequest](protocols.md#workflow-interactions) to its Request editor,
or adopt the complete Request supplied by EditRequest, before looking up
`right_pane`. The carrier Response, originating link, Go to, AI Search, and
previously displayed section do not determine placement. <!-- rule: master-detail.one-owner class=hard enforced=yes tasks=display-template title="At most one leading Response may own an editing Request under overlapping device conditions" --> At most one leading
Response may own that Request under overlapping device conditions. <!-- rule: master-detail.no-data-condition class=hard enforced=yes tasks=display-template title="A relationship cannot depend on a leading-page data condition, pseudo-state or switch" --> A rule's
relationship cannot depend on a leading-page data condition, pseudo-state,
or conditional switch. Ordinary presentation precedence resolves device rules.
An editor outside an applicable relationship uses the whole working surface. <!-- rule: master-detail.editing-requests-only class=hard enforced=yes tasks=display-template title="A relationship names editing Requests, not Response variants or navigation Requests" -->
A displayed Response also uses the whole working surface; a relationship names
editing Requests, not Response variants or navigation Requests that lead to them.

- <!-- rule: pane.editor.requires-correlated-leading-response class=hard enforced=no tasks=client-runtime --> **Initial leading-page load.** On installing an associated editor at or
  above the [pane threshold](#pane-threshold), construct `refresh` from the
  current complete editing Request by the shared
  [construction-context rules](schema.md#context-and-syntax), without a bound
  member, and send it as an ordinary correlated Request. Keep the editor's
  server-supplied interaction reference context unchanged. The reply must supply
  the declared leading Response before the editor is embedded; a same-named
  Response already on screen is not evidence that it is the right section or
  record. Resolve placement before rendering the editor: during this load keep
  an existing leading page in its current layout, or show the ordinary loading
  surface when no page is present. Render the editor directly in its resolved
  pane, without first showing it on the whole working surface. The same order
  applies when navigation chooses another editor while the leading pane is
  already visible. This supports entry into any section, including direct entry
  with no previous leading page. It never submits the editing Request automatically.
  On failure, report the ordinary error and keep the editor usable as a full
  page. A stale or duplicate reply cannot replace another editor.
- **Navigation ownership.** Both panes belong to the endpoint's one working
  surface. The leading pane owns complete navigation and the shared toolbar.
  A [stacked link](protocols.md#links) covers only its sending pane; its pending
  operation disables both panes under the shared rule below, navigate-away uses
  this placement contract, and crossing the pane threshold preserves the stack
  above the same live full-page editor.
  Both panes fill the content height and scroll independently; the editor keeps
  its heading and content actions without another screen toolbar. <!-- rule: master-detail.embedded-editor-controls class=hard enforced=no tasks=client-runtime -->
  An embedded workflow editor shows **Close** and primary **Next** in one
  compact action row. **Go back disappears**, including every `PrevRequest`
  chooser and shortcut, and the editor has no Back control of its own.
  This is unconditional while embedded; no comparison with Close or another
  action determines suppression. Next retains ordinary validation and explicit
  submission. Close clears the right pane, sends nothing, and keeps the leading
  page; accepted Close follows the ordinary
  [draft lifetime](annotations.md#draft-scope-and-lifetime). <!-- /rule -->
  Outside the pane the editor's eligible Go back and Cancel controls return.
  Platform Back acts on the whole working surface under the
  [landing](protocols.md#control-endpoint-landing-and-native-back) and
  [leaving](protocols.md#exit-while-pending) rules.
- <!-- rule: master-detail.app-owns-navigation class=judgment enforced=no tasks=display-template --> **Complete navigation is the App's responsibility.** The leading page must
  expose the navigation needed to revisit editable sections and leave the
  arrangement. Its links and actions construct from its own context, never
  the right editor's current values or private workflow context. Selecting
  another destination closes an idle editor before opening it; selection does
  not submit or save the edit. <!-- rule: master-detail.context-dependent-previous-stays-full-page class=judgment enforced=no tasks=display-template --> A flow requiring a previous action that depends
  on the editing context must remain a full-page flow.
- **One pending operation for both panes.** While an operation awaits its
  reply, the covered surface is fully disabled, including edits, request-sending
  controls and the trailing pane's Close, because the pending reply may place a
  page in that pane. The [waiting presentation](protocols.md#waiting-presentation)
  owns screen versus link-panel progress; merely editing a linked request
  shows no waiting spinner. Auxiliary calculations retain their ordinary
  ownership. During a leading-page load
  or refresh the working surface accepts no edit, link, import, Submit, or
  Close; it shows loading until the reply. Native Back follows the leaving
  rule. All replies remain owned by their ordinary pending operations.
- <!-- rule: master-detail.refresh.after-commit class=hard enforced=no tasks=client-runtime --> **Every successful right-panel commit refreshes.** After explicit Submit
  receives any reply outside the failure class (neither a failure status nor
  an Error), construct the declared `refresh` from the current leading page
  with the ordinary construction rules and no bound member, and send it.
  This includes successful commits that install no next page. If the commit
  reply itself supplies the updated leading Response, it replaces that page
  and clears the editor; no redundant refresh Request is sent. <!-- rule: master-detail.refresh.commit-only class=hard enforced=no tasks=client-runtime --> Validation
  failures, rejected commits, Close, calculations, and a link return that keeps
  the editor open do not refresh. Never replay the Request that originally
  produced the leading page: it may have been a save.
- **Following a commit.** Apply the refresh before presenting a new page from
  the commit. A newly resolved editor is placed by its own `right_pane`
  relationship; if it belongs to another leading page, load that page from
  that editor before embedding it. No row match is required. A terminal
  Response or an unassociated editor becomes the whole working surface.
  A refresh failure leaves the previous leading page in place, reports the
  ordinary error, and does not discard the valid commit reply.
- **Refresh replies.** The declared leading Response replaces the leading
  page even without NextRequest or WorkflowRequest; the
  [non-transition acknowledgement](protocols.md#protocol-roles) rule does not
  apply to this page load. An Error leaves the previous leading page in place
  and follows ordinary error handling. Any other admitted refresh reply becomes
  the whole working surface, as in a single-pane arrangement.
- <!-- rule: master-detail.detail class=hard enforced=yes tasks=display-template title="`detail` names visible linked nodes reached through struct fields, each carrying a Link with NextResponse, none repeated" --> **Optional row highlighting.** `detail`, when present, is a non-empty array
  of selectors naming linked nodes within the leading Response. An entry
  ending in `[]` selects List members; another entry selects one node. Paths
  traverse struct fields only, never another List or Union. Selected nodes
  and their ancestors must be visible, and each node must carry a Link with
  NextResponse. Highlighting uses the schema's explicit
  [enum/protocol association](schema.md#enumeration-protocol-associations),
  matching marked enum values in the current editing Request and linked row,
  including shared Request variants and direct entry. <!-- rule: master-detail.detail.selected-style class=judgment enforced=no tasks=display-template --> The matching node exposes
  the [`:selected` pseudo-state](#display-template-conditions); the template
  supplies its visible style, for example `"style": { "emphasis": "primary",
  "color": "primary" }`. Restrict `detail` to the intended navigation rows:
  including a summary link that repeats a topic creates an ambiguous match.
  Missing or ambiguous matches clear highlighting and never
  determine pane membership or require a full-page fallback. Clients do not
  compare link-request fields or choose the first row to infer selection.
- **Crossing the pane threshold.** Narrowing shows the current editor as a full
  page above its leading page, with ordinary Back, Go back, and Cancel controls.
  Native Back follows the [endpoint-exit rule](protocols.md#control-endpoint-landing-and-native-back)
  after resizing too. This change sends no editing Request. Widening evaluates the current editing
  Request's relationship again and embeds it, restoring Close and suppressing
  Go back. If its leading page has not yet been loaded, perform the initial
  load described above. A pending operation retains ownership through resizing;
  apply its reply once, then place it by the current pane threshold.
- <!-- rule: pane.collapse.preserves-editor-navigation class=hard enforced=no tasks=client-runtime --> **Divider and collapse.** Two panes show one platform-owned divider in the
  gutter. Dragging it moves the leading pane's edge between the leading
  minimum and the width that leaves the trailing minimum; releasing more than
  half the leading minimum below it **collapses** the leading pane, and
  releasing between those points snaps to the leading minimum. A double click
  or double tap restores automatic content sizing in Studio and the web
  interface, and the default share on Android and iOS. The divider is focusable
  between the panes and is a splitter for assistive technology, valued in percent and
  named for the leading page ("Resize Income pane"): Left and Right (mirrored
  for right-to-left) move 5% of the width shared by the panes, Home and End go
  to the two limits, Enter toggles collapse, and Android and iOS add Collapse
  and Expand screen-reader actions. A collapsed leading pane leaves one button,
  titled with the leading page's title and a sidebar glyph, at the leading
  edge of the trailing pane's heading; it restores the leading pane, which
  keeps its scroll position and loaded value. Collapse is still the embedded
  arrangement: Close and Next stay, Go back stays suppressed, and collapse
  and expand send nothing and never refresh. Only the leading pane collapses.
  With no editor in the trailing pane the leading page shows expanded. Collapse
  moves focus to the expand button when focus was in the leading pane, and
  expand moves it to the leading pane's first control; both announce the new
  state.
- <!-- rule: master-detail.viewer-preference class=hard enforced=no tasks=client-runtime --> **Viewer preference.** The divider position and collapse state are a
  viewing preference kept on the viewer's device for that Hub, package, and
  leading Response, separately for a `medium` window and for an `expanded` or
  wider one. They are never App data, never sent to the Hub, never part of a
  draft, and never written to `display.json` or the compiled template. The
  stored share is applied within the current limits and kept as stored, so
  narrowing and widening again restores it; an unreadable value is ignored.
  A stacked arrangement ignores the preference without resetting it.

<!-- rule: master-detail.refresh.loader class=hard enforced=yes tasks=display-template title="The `refresh` loader is constructible from the leading Response and every listed editor, has no editable input, confirmation or subscription, and admits the leading Response" -->
The loader must be constructible from both the leading Response and every
listed editing Request, using Fixed, Default, or supplied CopyFrom values.
It has no editable input, RequestConfirmation, or subscription behavior, changes
nothing in the service, and admits the leading Response in NextResponse.
When refresh carries earlier values such as a revision, the service admits
pure navigation by identity rather than rejecting a stale editing revision.
Below the pane threshold no pane-only load or refresh is sent. <!-- rule: master-detail.right-pane-required class=hard enforced=yes tasks=display-template title="New relationships declare `right_pane`; Studio rejects a `master-detail` rule without it" -->
Retained compiled templates without `right_pane` keep their earlier detail-link
arrangement; new relationships use the explicit editing-Request declaration,
and Studio rejects a `master-detail` rule without `right_pane`.

<!-- rule: layout.sections class=hard enforced=yes tasks=display-template title="`sections` lists direct children by native name with an optional `StringResources` title; a child appears in at most one section" -->
`sections` is an ordered list of [groups](#layout-groups),
`{ "fields": [...], "title": "KEY" }` plus an optional layout. `fields` names direct children by native name; `title` optionally
names a package [`StringResources`](#string-resources-and-formatted-output)
key whose localized text titles the section, and without it the section is
titled by its first field's display name. A child appears in at most one
section, and unlisted children go to the last section. Each section shows its
listed fields in the listed order, followed by any unlisted children in
declaration order. <!-- rule: layout.sections.tabs-and-rows-pair class=judgment enforced=no tasks=display-template --> The same `sections` serve `tabs` and `rows`, so a pair of
rules can show one Table as stacked sections in `edit` mode and as tabs in
`view` mode with the same titles. Under `rows`, each section title is a
heading above its fields; nothing is hidden, and moving to the next field
crosses section boundaries in order.

Because no tab hides an editable field in `edit` mode, a Submit blocked by a
field never has to reveal another tab.

On Android and iOS, each tab's direct children start at normal page or pane
padding, with indentation measured relative to that tab's content. This applies
to both Table sections and Union branches, including nested tabs. Nested
structs, Lists, and Unions within a tab retain their ordinary indentation and
`flatten` rules. The tab owner's row keeps its surrounding placement; entering
a tab changes only the content's visual indentation, not its value ancestry.

<a id="layout-groups"></a>

<!-- rule: layout.groups class=hard enforced=yes tasks=display-template title="Sections (`rows`/`tabs`/`accordion`) and split regions use `{fields,title?,layout?,style?}` plus layout members (`ratio`/`min_width` also on regions); only `rows` (default), `form-grid`, `card`, `split`, `tabs`, `accordion`, nested without a depth cap; titles belong to sections, fields stay in the parent pool without repeats, unlisted fields go to the last nested group; width fit stacks groups, never changing values, validation, calculation or encoding" -->
**Layout groups.** A Table's `sections` (under `rows`, `tabs`, or
`accordion`) and a `split`'s `regions` are **groups**: anonymous sets of the
Table's direct children with one shape, `{ "fields", "title"?, "layout"?,
"style"? }` plus that layout's own members, and under a `split` also `ratio`
and `min_width`. `title` applies where the parent shows group titles
(`rows`, `tabs`, `accordion`). A group's `layout` is any layout the catalog
allows on a Table except `categorized`, which belongs to a classifier itself:
`rows` (the default), `form-grid`, `card`, `split`, `tabs` or `accordion`,
with its own `sections` or `regions` over the group's fields, so groups nest
to any depth. Nested groups name only fields of their enclosing group; a field
appears in at most one group at each level, and a group's unlisted fields go
to its last nested group. Width fit stacks whatever does not fit, at every
level. Groups change only arrangement, never a value, validation, calculation,
or encoding.

<!-- rule: layout.split class=hard enforced=yes tasks=display-template title="`split` has two or three regions, each a group with a positive `ratio` and direct-child `fields`" -->
Each `split` region is a [group](#layout-groups) with a positive `ratio`, the
direct children it holds as `fields`, optional `style`, and its own layout,
`rows` by default. A region of a `row`
split may also declare `min_width`, a [size-class token](#width-based-layout)
defaulting to `compact`. A child
appears in at most one region, and unlisted children go to the last region. A
region shows its listed fields in the listed order, followed by any unlisted
children in declaration order. A split is how a photo on the left with name,
date of birth, and employee ID as rows on the right is expressed. <!-- rule: layout.split.row-needs-no-condition class=judgment enforced=no tasks=display-template --> A `row` split places its regions in
lines under the [width-based layout](#split-lines) rules, so it needs no device
condition: on a phone its regions stack, and where they fit they share a line.

```json
{ "select": "^<Employee>", "layout": "split", "direction": "row",
  "regions": [
    { "fields": ["photo"], "ratio": 1, "style": { "align": "center" } },
    { "fields": ["name", "date_of_birth", "employee_id"], "ratio": 2,
      "layout": "form-grid", "columns": 2 }
  ] }
```

<!-- rule: layout.columns.ratios-never-wrap class=hard enforced=no tasks=client-runtime -->
Adjacent split regions have a platform-owned gutter in either direction and
at every nesting level: 16 dp on Android, 16 pt on iOS, and 16 CSS px in Studio.
Stacked lines of a `row` split are separated by the same gutter. The gutter is
automatic, requires no `display.json` member, and is independent of the row
`spacing` style token. A `column` split divides its height by the ratios after
the gutters are deducted and never wraps. Nested device conditions use the
resulting region size.

<!-- rule: layout.tabs.scroll-full-labels class=hard enforced=no tasks=client-runtime -->
`tabs` shows every tab label in full. When the labels do not fit across the
pane, the tab strip scrolls horizontally; labels are never truncated and tabs
never fall back to stacked sections.

<a id="width-based-layout"></a>

#### Width-based layout

Every client adapts pane count, split lines, `form-grid` columns, and tables to
the space it has, at every window size, in two layers: the **window** caps the
structure, and **content fit**, checked against the containing pane or region,
fills within that cap. None of this changes a value, validation, `CopyFrom` or
`CopyTo`, calculation, encoding, or AI Import or Search.

<!-- rule: layout.width.size-classes class=hard enforced=yes tasks=display-template title="Size classes are one vocabulary: `width` condition values and `min_width` tokens (compact 360, medium 600, expanded 840, large 1200, extra-large 1600 dp)" -->
The size classes are one vocabulary. They are the values of the `width`
[condition](#display-template-conditions), and the same names are the
`min_width` tokens of split regions and `panes`, where each means its class's
lower edge:

| Class | `width` condition | `min_width` |
| --- | --- | --- |
| `compact` | under 600 dp | 360 dp, the narrowest supported phone in portrait (default) |
| `medium` | 600 dp up to 840 dp | 600 dp |
| `expanded` | 840 dp up to 1200 dp | 840 dp |
| `large` | 1200 dp up to 1600 dp | 1200 dp |
| `extra-large` | 1600 dp and up | 1600 dp |

<!-- rule: layout.width.tile-tokens class=hard enforced=yes tasks=display-template title="`tile` (160 dp) and `tile-wide` (240 dp) are `min_width` tokens only for `form-grid` and `grid`, never width conditions, split regions or panes" -->
Two more `min_width` tokens size cells narrower than a window class:
`tile` (160 dp) and `tile-wide` (240 dp). They are accepted only on
`form-grid` and `grid`, never as a `width` condition or on a region or pane.

<!-- rule: layout.width.text-scaling class=hard enforced=no tasks=client-runtime -->
dp is the Android density-independent pixel, an iOS point, and a CSS pixel in
Studio. A `min_width` scales with the platform's text-size setting, by its
factor clamped between 1 and 2, so larger text reflows sooner. A region that
holds a nested split needs at least the largest minimum among the nested
regions, because the nested split can itself stack.

<!-- rule: layout.width.window-cap class=hard enforced=no tasks=client-runtime -->
The window's width class caps the structure. The window is the app window on
Android and iOS, Studio's view, or the simulated device of a preview; a pane
never raises the cap:

| Window width | Panes at most | Split line or `form-grid` columns at most |
| --- | --- | --- |
| `compact` | 1 | 1 |
| `medium` | 2 | 2 |
| `expanded` | 2 | 3 |
| `large`, `extra-large` | 2 | 4 |

<!-- rule: layout.width.compact-height-one-pane class=hard enforced=no tasks=client-runtime -->
A window whose `height` class is `compact` shows one pane, so a phone in
landscape keeps room for the editor above the keyboard; its columns keep the
width cap.

<a id="split-lines"></a>

<!-- rule: layout.wrap.preserve-field-order class=hard enforced=no tasks=client-runtime -->
**Split lines.** A `row` split places its regions greedily in declaration
order. A region joins the current line while the line holds fewer regions than
the column cap and the line's minimums plus one gutter per adjacent pair fit
the split's width; otherwise it starts the next line. A region alone on a line
takes the full width, and its content wraps; nothing scrolls horizontally.
Within a line the width after gutters is divided by `ratio`; a region that
would fall below its minimum gets its minimum, and the rest is divided again
by ratio among the others until no region is below its minimum. Lines stack in
order, and reading, focus, and field order stay region order then field order;
wrapping never reorders. Each nested split uses its region's resulting width
and the same window cap. Right-to-left text mirrors each line.

<a id="pane-threshold"></a>

<!-- rule: master-detail.pane-threshold class=hard enforced=no tasks=client-runtime -->
**Pane threshold.** A [master-detail](#master-detail-panes) arrangement shows
two panes when the window allows two and the working surface is at least the
leading minimum plus the gutter plus the trailing minimum wide; otherwise it
shows one pane, stacked. With the default minimums the threshold is 736 dp at
normal text size. With two panes, the width after the gutter is shared by the
leading share, the leading `ratio` over both ratios by default (40% and 60%) or
the viewer's [preference](#master-detail-panes), held between the leading
minimum and the width that leaves the trailing minimum; the trailing pane takes
the rest.

<a id="form-grid-columns"></a>

<!-- rule: layout.form-grid.columns-maximum class=hard enforced=no tasks=display-template -->
**`form-grid` columns.** `columns` is a maximum. The grid uses as many columns
as fit its width at its `min_width` each (`compact` by default), with the gutter between them,
at least one and at most the smaller of `columns` and the window's column cap.
Fields flow in declaration order as before, and a `span` larger than the
columns in use is reduced to them. For example, `columns: 3` in an `expanded`
window shows one column below 736 dp, two from 736 dp, and three from 1112 dp.

<a id="table-column-fit"></a>

<!-- rule: layout.table.column-fit class=hard enforced=no tasks=client-runtime -->
**Table column fit.** A `table`, or a `summary` with `arrangement: "table"`,
shows as a table when its column minimums plus a gutter between each pair fit
its width; otherwise it shows the rows or list it falls back to. The fit
measures text in the current font, text size, and locale, never a size-class
token. A column's minimum is the largest of its header's longest word and
each current cell's minimum: a number, money, date, unit, enumeration, or
Boolean cell counts its full width, and a wrapping text cell counts its full
width up to the larger of its longest word and twelve times its font size.
Width left over goes to the columns holding wrapping text, in proportion to
their full widths, or, when no column wraps, to every column in proportion to
its minimum. Nothing scrolls horizontally. The choice depends on the
displayed data and is the same for the same value; when a data change switches
between table and rows, the reading position stays on the same member.

Pane count, lines, columns, and table fit are evaluated again when the window
or a pane changes size, when the text size changes, and when a divider drag
ends. During a drag the panes resize and their content reflows, but template
rules are resolved again only once the drag ends.

#### Labels and titles

Two members change the text a person sees on the node their rule selects:

- <!-- rule: label.label-replaces-name class=hard enforced=no tasks=display-template --> `label` replaces only the node's displayed name. The row keeps its value:
  the node's own value, or the child a
  [`promote`](#promotion-flattening-and-union-selectors) rule puts on
  the row.
- <!-- rule: label.title-replaces-row class=hard enforced=no tasks=display-template --> `title` replaces the whole row with its text. The row's label and value are
  both removed, and nothing is promoted into it, because it has no value; a
  child that a `promote` rule names keeps its own row, and Studio warns that
  the `promote` has no effect.

<!-- rule: label.text-not-typography class=judgment enforced=no tasks=display-template -->
Both change only the text, never its typography. The node keeps the
typography of its position: a page's title is drawn as a page title, a group
header inside a page as a group header, and a List member's row as a member's
row. A [`role`](#style-tokens) style token changes that look where an App
wants another one, for example `role: label` for a quieter group header; no
client reads `label` or `title` as a typography instruction.

Both take the same two value forms:

- <!-- rule: label.field-source class=hard enforced=yes tasks=display-template title="A `field` source is a struct-field path below the node (or below the struct holding a scalar field) ending at a String or FormattedText, Hidden or ReadOnly allowed" --> `{ "field": "path" }` takes the text from the data. `path` is a dot path of
  field native names below the selected node, through struct fields only,
  never through a List or a Union, ending at a String or FormattedText field;
  that field may be Hidden or ReadOnly. Its value, a FormattedText as its
  localized text, is the text. A scalar has no fields below it, so on a
  scalar field the path reads below the struct that holds the field, as a
  [`table`](#display-template-catalog) column header's reads below the struct
  that holds its List: a sibling field or one below a sibling. A scalar that
  no struct holds, such as a List's member, a function parameter, or a scalar
  selected by its type, takes only the `"KEY"` form.
- <!-- rule: label.key-source class=hard enforced=yes tasks=display-template title="`KEY` gives fixed text from a package `StringResources` key" --> `"KEY"` gives fixed text: a package
  [`StringResources`](#string-resources-and-formatted-output) key, as a
  section's `title` is.

<!-- rule: label.union-key-only class=hard enforced=yes tasks=display-template title="On a Union node or a node reached only through a Union, only the `KEY` form is valid" -->
A field path never passes through a Union, because the selected branch is
known only at run time. On a Union node, and on a node reached only through a
Union, such as a Union branch, a node below one, or a named type used only
there, only the `"KEY"` form is valid; Studio rejects the field form there
with a message that names the key form.

A `label` or `title` rule may select:

- a page, a displayed Request or Response variant (`^<Hub>`): the text is the page title and
  replaces the page's top row as the member says;
- a struct, by its type (`^<Type>`) or by a field of it: the text replaces the
  struct's row as the member says;
- a Union field, in the `"KEY"` form only: with `label` the selected
  variant stays its value;
- a List's members (`^<Resp.list>[]`): each member row takes its text from its
  own field, which replaces both the member name and its position number;
- with `label` only, a scalar field, such as a Number, String, Boolean,
  DateOnly, DateTime, Money, or Enumeration field, by a field path or by its
  type: the text replaces the row's name, and the row keeps its value and
  editor. A scalar List member is labelled through the List's members.

<!-- rule: label.placement class=hard enforced=yes tasks=display-template title="`label` selects a page, struct, Union field (`KEY`), List members or a scalar field; `title` never a scalar; neither a List itself or a scalar Union branch" -->
`title` never selects a scalar: a scalar's row is its value, which a title
would remove. Neither member selects a List itself (select its members with
`[]`) or a scalar Union branch, whose payload is drawn under the Union's row
with no row name of its own (select the Union field).

<!-- rule: label.typed-ancestor-occurrence class=hard enforced=no tasks=display-template -->
A rule selecting a field below a typed ancestor, such as
`^<FarmRental>.expenses.chemicals`, labels only the scalar reached that way.
The same struct reused elsewhere, such as `^<Farm>.expenses`, keeps the
schema's display name:

```json
{ "select": "^<FarmRental>.expenses.chemicals", "label": "DISPLAY_4835_LINE9_CHEMICALS" }
```

<!-- rule: label.empty-source-fallback class=hard enforced=no tasks=display-template -->
When the field is null, unanswered, or empty, the node draws as it would
without the member, with its position for a List member, so a row is never
blank. The display name stays the schema's meaning: AI tool descriptions and
every declaration keep it, and `label` and `title` change only what a person
sees. <!-- rule: label.source-hidden class=judgment enforced=no tasks=display-template --> The source field is normally declared Hidden in the schema, so its
value is not drawn twice; a Hidden field remains a valid source. Hidden stays
a schema attribute that the template never sets: Studio neither hides a
visible source field nor rejects it, but reports a warning at the `label` or
`title` line of `display.json` that names the visible field and suggests
Hidden. A visible source field keeps its own row.

<!-- rule: label.exclusive class=hard enforced=yes tasks=display-template title="`label` and `title` exclude each other, and neither combines with `suppress_label` on one node" -->
`label` and `title` exclude each other on one node, and neither combines with
`suppress_label` on that node. A `label` with a `promote`d child is how a row
is [data-titled](#data-titled-rows). <!-- rule: label.on-promoted-field class=hard enforced=no tasks=display-template --> A `label` on a promoted field labels the
row only with `use_parent_name` false; with `true` the row takes its struct's
displayed name, the field's `label` shows nowhere, and Studio warns. A Hidden
field has no row, so its `label` shows nowhere. <!-- /rule -->

<!-- rule: label.condition-limits class=hard enforced=no tasks=display-template -->
Both resolve through the
[cascade](#display-template-cascade) and `when` like every other member; a
[data condition](#display-template-conditions), switch case, or pseudo-state never sets either; see the
[condition limits](#display-template-conditions).

On a struct, the two members combine with [`promote`](#field-promotion) on a
child and with [`flatten`](#flatten) on the struct as follows:

| On the struct | With | Result |
| --- | --- | --- |
| `label` | `promote` on a child, `use_parent_name` true | The row's label is the struct's displayed name, that is, the `label` text; the child is the value. |
| <!-- rule: label.combine.label-promote-child-name class=hard enforced=no tasks=display-template --> `label` | `promote` on a child, `use_parent_name` false | The row's label is the child's own name. Studio warns that the struct's `label` shows nowhere. |
| <!-- rule: label.combine.title-promote class=hard enforced=no tasks=display-template --> `title` | `promote` on a child | The child is not promoted and keeps its own row. Studio warns that the `promote` has no effect. |
| <!-- rule: label.combine.title-flatten-name-row class=hard enforced=yes tasks=display-template title="`title` and `flatten` `{ name_row: false }` cannot combine on one struct" --> `title` | `flatten` with `{ "name_row": false }` | Rejected: one removes the name row and the other replaces it. |

<!-- rule: label.links-stay class=hard enforced=no tasks=display-template -->
With `label`, the link of a promoted value stays on the row; with `title`, a
link on the struct stays on the title row
([actions](#actions-on-a-promoted-or-suppressed-row)).

```json
{ "select": "^<TopicRow>", "label": { "field": "title" } }
{ "select": "^<TopicRow>.status", "promote": true }
{ "select": "^<Hub>", "title": { "field": "topics.name" } }
```

The first two rules draw each topic row as "W-2 wages   Entered": its `title`
field as the label and its promoted `status` as the value. The third draws the
hub page's top row as just "Income", the name of the section whose topics the
page lists.

#### Button labels

`button_labels` changes the captions of existing controls on one protocol
page. It is a non-empty object with one or more of these slots:

| Slot | Existing control | Behavior retained |
| --- | --- | --- |
| `go_back` | Workflow Go Back | Constructs and sends the selected `PrevRequest`; native Back is unaffected. |
| `next` | Workflow Next/Submit | Validates and submits the current Request; it does not choose a `NextRequest`. |
| <!-- rule: template.close.recapture-only class=hard enforced=no tasks=display-template --> `close` | Page, detail-pane, or stacked-link page Close | Performs the existing local dismissal without submitting; a stacked page's system Close may only be recaptioned, never removed. |
| `actions` | Page Actions | No eligible choices hides the control; one selects directly; several open the ordinary chooser. |

<!-- rule: button.slot-values class=hard enforced=yes tasks=display-template title="`button_labels` slots take a `StringResources` key or `null`; literal text, field sources, empty strings and unknown slots are invalid" -->
Each slot accepts a package `StringResources` key, or `null` to restore the
localized platform caption. Literal display text, field sources, empty strings,
and unknown slots are invalid. <!-- rule: button.localization-and-no-inheritance class=hard enforced=no tasks=display-template --> Localization uses the package that authored the
winning rule and the ordinary resource fallback. If a client cannot resolve a
non-empty caption, it uses the platform caption, never a blank button or raw
resource key. Omission preserves a lower-ranked override or the platform default;
`null` overrides lower-ranked captions with that default. Slots do not inherit
into children or another page.

<!-- rule: button.single-action-source class=hard enforced=no tasks=display-template title="`actions` also accepts `{ source: single_action }`: the sole offered action's localized variant name, else the next caption source" -->
The `actions` slot also accepts exactly `{ "source": "single_action" }`.
It uses the sole offered action's localized schema variant `Name`. Count the
final offered set after `NextRequest`, `ConstrainedBy`, access privileges, `ExcludeUI`, and
`LinkOnly` filtering, and an explicit `Actions` array when the page supplies
one ([named action controls](protocols.md#named-action-controls)). With several choices, or a missing or blank name, the
source supplies no caption, and the next source in the
[Actions caption precedence](#named-action-controls) applies: an explicit
Actions field name, then the localized platform Actions caption, never a raw
native name or payload type name. With no choices, retain the existing hidden
control. This source
uses the action's owning document, not the
template author's resources or a display title. No other slot accepts it.
An explicit `null` resets this source just like a resource override.

```json
{ "select": "^<Hub>", "button_labels": { "actions": "GO_TO" } }
{ "select": "^<SectionHub>", "button_labels": {
    "actions": { "source": "single_action" } } }
{ "select": "^<SaveIncome>", "button_labels": {
    "next": "SAVE_AND_CONTINUE", "close": "CLOSE_EDITOR" } }
{ "select": "^<Finish>", "button_labels": { "next": "FINISH_REVIEW" } }
{ "select": "^<SaveIncome>", "when": { "form": "phone" },
  "button_labels": { "next": null } }
```

<!-- rule: button.whole-page-variants class=hard enforced=yes tasks=display-template title="`button_labels` selects whole protocol page variants; `go_back` and `next` need a Request variant" -->
Rules select whole Request, Response, or SubscriptionData variants of an
Endpoint protocol. `go_back` and `next` require a Request variant. A rule
cannot select a field, reusable payload type, List member, configuration, or
persistence-only variant to rename page controls. <!-- /rule -->

<!-- rule: button.conditional.allowed class=hard enforced=yes tasks=display-template title="Data conditions and switch branches may set button_labels; pseudo-states may not" -->
Device `when` conditions are allowed.
[Data conditions](#display-template-conditions), switch cases, and `else` may
set `button_labels`; pseudo-states (`:pending`, `:error`, `:selected`) may not.
A valid caption remains dormant when its control is absent.

<!-- rule: button.conditional.caption-only class=hard enforced=no tasks=client-runtime -->
A conditional caption changes only the text of an existing control. Next
validates and submits the same Request; Go Back, Close, and Actions keep their
existing behavior. No caption condition shows, hides, enables, disables, or
retargets a control, selects a `NextRequest`, or sends anything.

<!-- rule: button.conditional.root-and-timing class=hard enforced=no tasks=client-runtime -->
The condition is rooted at the active whole protocol-page value: the Request
being edited, Response, or SubscriptionData. It reads only that value and the
interaction reference context, remains
[locale independent](#display-template-conditions), and is re-evaluated on the
displayed value whenever its inputs change. A condition that does not yield
true does not apply.

<!-- rule: button.conditional.slot-cascade class=hard enforced=no tasks=display-template -->
Inside a condition or switch branch, slots resolve independently under the
[ordinary cascade](#display-template-cascade). An omitted slot preserves the
lower-ranked caption; `null` restores the platform caption. The `actions` slot
still accepts `{ "source": "single_action" }`. A switch applies its first true
case, otherwise `else`, otherwise nothing.

<!-- rule: button.conditional.stable-placement class=hard enforced=no tasks=client-runtime -->
Decide the page-button group's [top-bar-or-row placement](#protocol-page-heading-and-top-bar)
using the widest localized caption each visible slot can take across matching
unconditional rules, data conditions, and all switch branches, including the
platform fallback. Evaluate device conditions normally; ignore data-condition
outcomes only for this measurement. A caption change while editing must not
move the group between the top bar and the secondary row. Width, locale,
font, text size, device conditions, template, and control availability still
trigger normal placement measurement.

<!-- rule: button.conditional.unresolvable class=hard enforced=no tasks=client-runtime -->
An unresolvable or blank conditional caption uses the localized platform
caption, never a blank button or raw resource key.

For a Request whose `documents` field is a List:

```json
{ "select": "^<SaveDocuments>", "switch": {
    "cases": [{ "condition": "count([documents]) == 0",
      "button_labels": { "next": "CONTINUE_WITHOUT_DOCUMENTS" } }],
    "else": { "button_labels": { "next": "SAVE_AND_CONTINUE" } } } }
```

<!-- rule: button.active-variant-owns class=hard enforced=no tasks=client-runtime -->
The active displayed variant owns its captions. A workflow editor uses its
Request, not its originating Response. In master-detail, the leading page
owns its toolbar and the trailing page owns its own controls. An embedded
editor's Go Back remains suppressed, even when a caption is configured.
Captions never create controls, change eligible requests or their order,
change input presentation, submit automatically, or alter validation,
confirmation, pending-operation ownership, or draft lifetime. Request names
inside choosers, native navigation, ordinary request-editor Send, and
confirmation/picker Send and Cancel labels are unchanged.

<!-- rule: button.truthful-wording class=judgment enforced=no tasks=display-template -->
Choose wording that remains truthful for every state in which the page uses
it. For example, "Go to…" can describe a sole destination-chooser request;
"Return to review" is appropriate only when the offered operation actually
returns to review. Renaming Actions does not change its target. "Save changes"
does not promise a particular next page when the service can ask follow-up
questions; Close must not imply submission or saving.

Clients measure the resolved caption with the actual locale and accessibility
text size under the existing responsive control and heading rules, allowing
content action rows to wrap as needed while retaining readable text and touch
targets. The visible caption is also the accessible name. Re-resolve it when
the displayed variant, offered action set, template, locale, or device condition changes. Older
clients ignore this optional member and retain their platform captions.

#### Named action controls

<!-- rule: control.declared-by-attribute class=hard enforced=no tasks=display-template -->
The [ActionControl schema attribute](protocols.md#named-action-controls) declares
the controls and their choices. No authored template is required. A named
control uses its field's ordinary localized display name; this is a document
name. The built-in Actions control
keeps its platform caption when implicit, and an explicit Actions field can
supply its default name. The selected Requests keep their own localized names
inside a chooser.

<!-- rule: actions.controls.normal-touch-targets class=hard enforced=no tasks=client-runtime -->
Actions is the prominent page-heading control. Its default treatment uses a
filled accent background, contrasting text and a semibold caption. Optional
named controls use secondary emphasis: a transparent background, subtle outline
and normal-weight caption. Controls share normal platform sizing, padding and
corner treatment; emphasis never reduces their touch targets. Authored emphasis
and density tokens retain their ordinary overrides.

<!-- rule: control.group-placement class=judgment enforced=no tasks=client-runtime -->
Keep all visible page controls, the optional named controls and Actions when
eligible, together as one group. Named controls use field declaration order;
Actions is always last, at the logical trailing end (right in a left-to-right
layout). Put the complete group in the top bar whenever it fits there under
the [buttons-before-title rule](#protocol-page-heading-and-top-bar). Never
promote a smaller subset. Actions alone, or the named controls alone, are
placed by the same rule.

<!-- rule: actions.controls.secondary-row class=hard enforced=no tasks=client-runtime -->
When the group cannot fit, put every button on a new secondary row below the
heading and above content, leading aligned with full captions. No button
shares the content title row. The row stays under the top bar when the heading
scrolls out, under the [pinned-buttons rule](#protocol-page-heading-and-top-bar).
The secondary row wraps as space requires, preserving
order; an individually oversized caption wraps inside its button and grows
its height. Actions retains its prominent styling.

A custom Actions caption does not make it an optional named control. An empty
eligible set hides its control and releases its space; an
empty secondary region takes no space. In master-detail, the leading page owns
global controls; trailing Response controls stay together on the row below
their local heading. An embedded Request editor keeps
Close and Next, with no response controls.

<!-- rule: control.action-controls class=hard enforced=yes tasks=display-template title="`action_controls` on a whole Response or SubscriptionData page is keyed by a declared control name or `Actions` and sets `label`, `placement`, `emphasis` and `density`" -->
On a whole Response or SubscriptionData protocol-page rule, `action_controls`
is an object keyed by an exact declared control name or `Actions`. A rule
cannot invent a control. Each entry can set `label`, `placement`, and the
`emphasis` and `density` style tokens. <!-- /rule --> Members cascade independently under device `when`
conditions; [data conditions](#display-template-conditions), switches, and pseudo-states cannot set them.

```json
{
  "select": "^<Hub>",
  "action_controls": {
    "sections": { "label": { "field": "section_caption" }, "placement": "secondary" },
    "Actions": { "label": "PAGE_ACTIONS", "placement": "secondary" }
  }
}
```

<!-- rule: control.placement class=hard enforced=yes tasks=display-template title="`placement` is `secondary` or `header`; `secondary` keeps the complete visible group below the heading" -->
`placement` is `secondary` or `header`. Omission and `header` request the top
bar whenever the complete group fits there. A `secondary` placement on any
visible control keeps the complete group on the row below the heading.
A multiline caption or a button too tall for the native top bar likewise keeps
the complete group below. A trailing
pane does not get another screen toolbar. Styling never changes eligibility
or invocation.

<!-- rule: control.label class=hard enforced=yes tasks=display-template title="A control `label` is a `StringResources` key, a struct-field path to a String or ReadOnly FormattedText below the page, or `null`" -->
`label` accepts a package StringResources key, `{ "field": "path" }`, or
`null` to restore the default caption. The field form uses the existing
[label path](#labels-and-titles) below this displayed payload through struct
fields only, ending at a String or statically ReadOnly FormattedText field.
It never reads the selected Request, a reference context, List or Union
member, or an expression. A Hidden caption source is allowed as an explicit
text source. Invalid paths and types fail Studio validation. An absent, null,
empty, or unresolvable current value falls back to the field's localized name
or platform Actions caption, never a blank or raw key. Caption updates do not
select or submit an action. <!-- rule: control.actions-caption-precedence class=hard enforced=no tasks=display-template --> For Actions, an explicit
`action_controls.Actions.label` takes precedence over `button_labels.actions`,
then an explicit Actions field name, then the platform caption.

#### Task-controlled section-hub action

<!-- rule: hub-action.single-action-pattern class=judgment enforced=no tasks=display-template -->
For a walkthrough with section hubs and a main Review page, declare the
continuation and review Requests in each hub's `NextRequest`, then let the
task's `ConstrainedBy` value offer exactly one of them. Use the
`single_action` caption source so its name and destination change together. <!-- rule: hub-action.link-only-rows class=judgment enforced=no tasks=display-template -->
Keep row-opening Requests `LinkOnly`; they remain available to their links
without becoming additional Actions choices. Clients need no walkthrough
flag, navigation history, or conditional caption rule.

For example, the tax task offers **Continue** while walkthrough
steps remain. Each section hub has its own continuation Request, which
records that section's completion and advances to its next applicable
section using the task's existing navigation. When no walkthrough step
remains, the task offers **Review**, opening the main Review
page. If an accepted edit creates unfinished walkthrough work, the task
offers continuation again. <!-- rule: hub-action.derive-from-completion-state class=judgment enforced=no tasks=display-template --> Derive this choice from existing completion state
rather than persisting a separate presentation flag. Remove a redundant
next-step or Review row when this button owns that operation.

<!-- rule: hub-action.separate-from-editor-next class=judgment enforced=no tasks=display-template -->
The section-hub Actions control and an editor's Next control remain separate.
An editor's **Save and continue** submits its answers and follows the task's
save route; a hub's **Continue** follows that section's onward route. The
routes may differ. A hub action cannot submit an open editor's uncommitted
fields. Preserve other required leading-page navigation when replacing a
chooser with one direct action. Keep full captions on phones and use the
responsive sizing and wrapping above.

#### Style tokens

| Token | Values |
| --- | --- |
| <!-- rule: style.tokens class=hard enforced=yes tasks=display-template title="Style tokens and their closed values: `density`, `emphasis`, `align`, `role`, `color` (platform roles only), `spacing` 0 to 4, `span` 1 to 4, `icon`" --> `density` | `compact`, `comfortable` |
| `emphasis` | `primary`, `secondary`, `muted` |
| `align` | `start`, `end`, `center` |
| `role` | Typography role: `title`, `body`, `label` |
| `color` | Platform color roles only: `primary`, `secondary`, `tertiary`, `error`, `surface`, `on-surface-variant` |
| `spacing` | A five-step scale, `0` to `4` |
| `span` | `1` to `4`: the columns a field takes inside a `form-grid` |
| <!-- rule: style.icon.first-line class=hard enforced=no tasks=client-runtime --> `icon` | One of `check`, `warning`, `info`, `lock`, `link`, drawn before the value on the same line: the icon aligns with the value's first line, and a value that wraps or spans several lines or paragraphs (such as FormattedText) keeps its lines indented to the text start beside the icon, never below it; with `:condition(...)` it makes a status badge |

<!-- rule: style.visual-only class=judgment enforced=no tasks=display-template -->
Style tokens are visual only. They never disable, hide, or change the
interaction of what they style: a `muted` row keeps its links and its editing,
and hiding stays the schema's Hidden. An App that wants a row out of reach
leaves it out of the data or its allowed choices instead.

Android maps tokens to Material 3 roles and window size classes, iOS to SwiftUI
roles and size classes, and Studio to its control-view variables. <!-- rule: style.media-not-a-token class=hard enforced=yes tasks=display-template title="Larger inline media is the `media` hint, not a style token" --> Larger
inline media is the [`media` hint](#behavior-hints), not a style token; the
media field and its preview are defined [below](#mime-aware-bytes-presentation).

#### Behavior hints

Each hint has exactly the meaning its source attribute has, and `promote`,
`when_single_choice`, `media`, and `chart`, which have no source attribute, the
meaning their sections define; the linked section keeps that meaning and any
authoring syntax.

| Hint | Value | Source attribute and meaning |
| --- | --- | --- |
| <!-- rule: hint.inline class=hard enforced=yes tasks=display-template title="`inline` (true or false): InlineChoices on an Enumeration or Union node" --> `inline` | `true` or `false` | [InlineChoices](#inline-choices) on an Enumeration or Union node. |
| <!-- rule: hint.fold class=hard enforced=yes tasks=display-template title="`fold`: true, false or `{ initially_open }` on a Table, Union or List; false makes the node unfoldable" --> `fold` | `true`, `false`, or `{ "initially_open": Boolean or "nondefault" }` | [Foldable](#folding-structs-and-unions) on a Table, Union, or List; `true` means `initially_open` true, `false` makes the node unfoldable; `"nondefault"` starts it open only when its value differs from its default. |
| <!-- rule: hint.promote class=hard enforced=yes tasks=display-template title="`promote`: true, false or `{ use_parent_name }`, only on a rule selecting a field of a struct" --> `promote` | `true`, `false`, or `{ "use_parent_name": Boolean }` | [Promotion](#promotion-flattening-and-union-selectors), with no source attribute. Valid only on a rule selecting a field of a struct, never a function parameter: that field becomes its parent row's value. `true` means `use_parent_name` true; `false` turns promotion off. |
| `when_single_choice` | `"show"` (the default), `"hide"` or `"payload"` | [Single-choice rows](#single-choice-rows), with no source attribute. Valid only on a rule selecting a struct field that carries `ConstrainedBy` or is a scalar `EnumSource` host: `"hide"` withholds the row while the field's allowed set leaves one answer without a payload. |
| `when_empty` | `"show"` (the default) or `"hide"` | <!-- rule: hint.when-empty class=hard enforced=yes tasks=display-template title="`when_empty` is show by default or hide: hide withholds the whole empty List row (title, count, header and content), restoring it with the first member; only ReadOnly/OutputOnly membership or no Add offered; values, validation, calculations and encoding unchanged" --> No source attribute. Valid only on a rule selecting a List a person cannot add to (its membership is ReadOnly or OutputOnly, or no Add is offered). `"hide"` withholds the List's whole row (title, count, header and content) while it has no member, and shows it again with the first member. Like every withheld row, it changes no value, validation, calculation or encoding. |
| `when_embedded` | `"show"` (the default) or `"hide"` | <!-- rule: hint.when-embedded class=hard enforced=yes tasks=display-template title="`when_embedded` is show by default or hide: hide withholds the node and its presentation subtree while its editor is embedded in a master-detail trailing pane; full-page placement restores it; values, validation, calculations and encoding are unchanged" --> Withholds the node and its presentation subtree while its editor occupies the trailing pane of a master-detail arrangement. Full-page placement restores it, including below the pane threshold. Collapsing the leading pane keeps the editor embedded. A child's `"show"` does not expose a withheld ancestor. This is presentation only: stored values, validation, CopyFrom/CopyTo, calculations, encoding, AI Import/Search, and explicit label/title/column reads remain unchanged. Device conditions may select the hint; data conditions, switches and pseudo-states may not. |
| <!-- rule: hint.suppress-label class=hard enforced=yes tasks=display-template title="`suppress_label` (true or false): omit the row label; on a List of structs the member names and positions" --> `suppress_label` | `true` or `false` | [SuppressLabel](#suppresslabel-label-free-value-rows): omit the selected value row’s label; on a List of structs omit the List-contributed member names and positions. |
| <!-- rule: hint.include-description class=hard enforced=yes tasks=display-template title="`include_description` (true or false): UIIncludeDescription" --> `include_description` | `true` or `false` | [UIIncludeDescription](#common-field-metadata). |
| <!-- rule: hint.format class=hard enforced=yes tasks=display-template title="`format`: one portable pattern checked against the node's effective type" --> `format` | One pattern string | [`Format`](#display-formats): only the portable pattern grammar that section defines, checked against the node's effective type; no other string is accepted. |
| <!-- rule: hint.scale class=hard enforced=yes tasks=display-template title="`scale`: the scale type of a scale-bearing chart field under the chart type matrix" --> `scale` | One scale type token | The chart scale type of a scale-bearing chart field, under the type matrix of the [chart specification](charts/2d.md#6-chartscale). |
| <!-- rule: hint.guide class=hard enforced=yes tasks=display-template title="`guide`: the position of the selected field's chart guide under the guide position matrix" --> `guide` | One guide position token | The position of the selected field's chart guide, under the [guide position matrix](charts/2d.md#64-chartguide-generic-guides). |
| <!-- rule: hint.palette class=hard enforced=yes tasks=display-template title="`palette`: `auto`, `categorical`, `sequential` or `diverging` on a chart root" --> `palette` | `auto`, `categorical`, `sequential`, or `diverging` | The discrete color list of a chart root's categorical paint, under the [chart palette rules](charts/2d.md#display-template-palette). |
| <!-- rule: hint.media class=hard enforced=yes tasks=display-template title="`media` `{ max_height?, aspect? }` (at least one) only on `Bytes` with effective `MimeTypes`; `aspect` is `square` (1:1), `landscape` (4:3) or `portrait` (3:4), and aspect alone uses `medium` height" --> `media` | `{ "max_height": token, "aspect": "square" \| "landscape" \| "portrait" }`, each optional | [Inline media](#inline-media-size), with no source attribute. Valid only on a `Bytes` node with effective `MimeTypes`: how tall the field's media draws inline, and optionally the fixed aspect box it fills. |
| <!-- rule: hint.chart class=hard enforced=yes tasks=display-template title="`chart` `{ aspect, max_height }`, at least one key, only on a two-dimensional chart root; `aspect` is `square`, `4:3` or `16:9`" --> `chart` | `{ "aspect": token, "max_height": token }`, at least one key | [Responsive chart preview](charts/2d.md#preview-chart-height), with no source attribute. Valid only on a two-dimensional chart root (`Chart`): `aspect` is `square`, `4:3`, or `16:9` and replaces the family aspect; `max_height` replaces the preview's height cap. Each key resolves independently. |

<!-- rule: hint.placement-follows-source class=hard enforced=no tasks=display-template -->
Every hint keeps its source attribute's placement rules, including the node
kinds it may select, and a hint never makes a read-only value editable.

<a id="height-tokens"></a>

<!-- rule: hint.height-tokens class=hard enforced=yes tasks=display-template title="`max_height` is `thumbnail`, `small`, `medium` or `large`, a share of the pane's visible height, never pixels" -->
**Height tokens.** `max_height` in `media` and `chart` names a share of the
visible height of the nearest enclosing pane, never pixels. Like the `height`
condition it has container semantics: a node in the right pane of a
`master-detail` layout measures that pane, and a node outside any pane
measures the usable window after safe areas, system bars, and persistent
application chrome. The value tracks the pane as it resizes or rotates.

| Token | Height cap |
| --- | --- |
| `thumbnail` | The compact, platform-bounded size: the current media thumbnail, or the `180`-point chart preview region. The default for `media`. |
| `small` | About 25% of the pane's visible height. |
| `medium` | About 40% of the pane's visible height. |
| `large` | About 60% of the pane's visible height. |

<!-- /rule -->
A rule may name `thumbnail` to undo a larger size that a lower-ranked rule
set. `media` and `chart` are device-dependent presentation like every other
hint: they never change a stored value, its validation, `CopyFrom` or
`CopyTo`, calculation requests, encoding, or AI Import, and they may appear
under `when` but not in a [data condition](#display-template-conditions) or switch case.

```json
{ "DisplayVersion": 1, "rules": [
  { "select": "^<Employee>.photo", "media": { "max_height": "medium" } },
  { "select": "^<Employee>.photo", "when": { "height": "compact" }, "media": { "max_height": "small" } },
  { "select": "^<EnergyHistory>", "chart": { "aspect": "16:9" } },
  { "select": "^<RoomShare>", "when": { "form": "phone" }, "chart": { "max_height": "large" } }
] }
```

<a id="presentation-locks"></a>

#### Presentation locks

<!-- rule: tpl.lock.meaning class=hard enforced=no tasks=display-template,field-presentation title="`lock`: the person cannot change the node; every other writer still writes it" -->
`lock` keeps the person from changing a value the App's business logic
already set. Its most common use is with [`AskOnAdd`](#add-time-values): the
person gives a member's defining value at Add, and the lock keeps it fixed
afterwards. It is presentation only and restricts the person alone:
initialization, `CopyFrom`, `EditRequest` values, calculation and link-return
`CopyTo` results, AI Import, draft restoration, and the App write a locked
value exactly as an unlocked one, and its validation and save-path handling
are unchanged.

| Value | Protected from the person |
| --- | --- |
| `true` | What schema [`ReadOnly`](#common-field-metadata) would protect on the node: the value and its subtree, or on a List its membership and order, its members keeping their own editability. |
| `"variant"` | On a Union only: the selected variant. Its payload keeps its own editability. |

- <!-- rule: tpl.lock.not-invariant class=hard enforced=no tasks=display-template,calculation --> **Not an invariant:** unlike `ReadOnly`, a lock never makes a
  [calculation input invariant](protocols.md#invariant-calculation-inputs); a
  locked input that triggers a calculation still declares `CalculationRequest`.
- **Errors stay visible:** a locked value that is unanswered or fails
  validation shows read-only with its error, as a ReadOnly value does, and
  blocks submission. A lock that leaves the person a value to fix is the App's
  error.
- <!-- rule: tpl.lock.which class=judgment enforced=no tasks=display-template --> **Which to use:** a field the person never changes anywhere is schema
  `ReadOnly`; `lock` is for a value the person must not change in the contexts
  its rule selects, such as a document's kind after
  [`AskOnAdd`](#add-time-values), with `lock: "variant"`.

<!-- rule: tpl.lock.placement class=hard enforced=yes tasks=display-template title="`lock` is true, or `variant` on a Union; never Hidden, no data condition or `when`" -->
Studio rejects `lock: "variant"` on a node that is not a Union, `lock` on a
Hidden node, and either on a rule with a data condition or `when`.

### Template validation

Studio's shared validator owns every template rule. It rejects a compiled
template unless:

- `DisplayVersion` is known and every rule has exactly one `select` and at
  least one presentation member;
- every selector resolves against the package schema under the
  [selector grammar](schema.md#display-template-selectors), including its
  name-collision rule;
- a pseudo-state appears only on a rule whose only presentation member is
  `style`;
- every data condition and switch case parses under the ValidationRules
  grammar, is Boolean, and reads only the selected node and the reference
  context, every switch meets the rules [above](#display-template-switch),
  and neither a data condition nor a switch case sets `label`, `title`, `action_controls`,
  `promote`, `flatten`, `when_single_choice`, `media`, or `chart`;
- no rule member is outside the catalog, including every schema-only
  attribute;
- <!-- rule: tpl.validation.master-detail class=hard enforced=yes tasks=display-template title="A master-detail relationship excludes a WorkflowRequest leading Response, and a `refresh` rule selects a Response without SubscriptionData" --> every layout selects a node kind its row allows; every `master-detail`
  relationship names a non-empty list of distinct Request variants of the same
  protocol, excludes a WorkflowRequest leading Response, has no overlapping
  other owner, and does not depend on data conditions or switches; each optional
  `detail` entry is a `^<…>` path within the selected Response, never a
  `$…` path or a descendant search, and resolves through struct fields only;
  no entry repeats another; no detail node is Hidden or lies under a Hidden
  node; and every detail node carries a `Link` whose request declares
  `NextResponse`; a `refresh` names a Request of the same protocol whose
  `NextResponse` admits the selected Response, whose every field is Hidden or
  ReadOnly and is filled by Fixed, Default, or a `CopyFrom` alternative that
  the selected Response and every listed editing Request supply, and which is neither a
  `SubscriptionRequest` nor marked `RequestConfirmation`, and a rule with
  `refresh` selects a Response that does not carry `SubscriptionData`; every `table` column is a string or an object with `value` and optional
  `header` only, its `value` resolves as a [table column](#display-template-catalog) path to a
  scalar field of the member type, Hidden allowed, no `value` repeats, a
  `header` names a package `StringResources` key or resolves as a column
  header field, and `sort` names one column's `value`; `sections`, `regions`, and `fields` name direct children; a section's
  `title` and a fixed `label` or `title` name a package `StringResources` key;
  a group's `layout` is a Table layout other than `categorized`; a region's `min_width` is a
  [size-class token](#width-based-layout) and appears only on a region of a
  `row` split; and `panes` appears only with `master-detail`, holds only
  `leading` and `trailing`, each with only a size-class `min_width` and a
  positive `ratio`, and is not set by a data condition or switch;
- every `label` or `title` selects a node its [placement](#labels-and-titles)
  allows, a `title` never a scalar; every `{ "field": "path" }` of either
  resolves below the selected node, or below the struct that holds a selected
  scalar field, through struct fields only, never through a List or a Union,
  and ends at a String or FormattedText field; the selected node is neither a
  Union nor reached only through a Union, nor a scalar that no struct holds,
  where only the `"KEY"` form is valid; no node has both `label` and
  `title`, and neither is combined with `suppress_label` on the same node;
  Studio reports these at the member's line of `display.json`;
- every `button_labels` is a non-empty object of the four slots above, with
  package resource-key or null values, selects only whole protocol page
  variants, and sets `go_back` or `next` only on Request variants;
- every `action_controls` key names a control declared on the selected whole
  Response or SubscriptionData page, or built-in Actions; its label source,
  placement, and style satisfy the named-control rule above;
- every `promote` rule selects a field of a struct, never a type, a List
  member, a Union branch, a function parameter, or a Response; every
  `flatten` rule selects a struct, a List, or a Union, by type or by field,
  never a List member or a Union branch, and its value is `true`, `false`, or
  `{ "name_row": false }`; `name_row` appears only on a struct; and no struct
  has both `title` and `flatten` with `{ "name_row": false }`;
- every `when_single_choice` rule selects, under the same placement rule as
  `promote`, a field of a struct that carries `ConstrainedBy` or `EnumSource`,
  and its value is `"show"` or `"hide"`;
- every `media` rule selects a `Bytes` node with effective `MimeTypes`, and its
  value is an object whose only key `max_height` is a
  [height token](#height-tokens); every `chart` rule selects a node carrying
  `Chart`, never a `Chart3D` root or a chart field, and its value is a
  non-empty object of `aspect` (`square`, `4:3`, or `16:9`) and `max_height` (a
  height token) only;
- every hint satisfies its source attribute's placement rules and value
  grammar, and every `when` key and value is from the closed sets.

<!-- rule: tpl.validation.equal-rank-warning class=hard enforced=no tasks=display-template -->
Rules of equal rank that set the same member on the same node are allowed, the
later one winning, and are reported as a warning. <!-- /rule --> Studio also warns, at the
member's line, when the source field of a `label` or `title` is not Hidden,
naming that field; when a struct's `label` shows nowhere because its
promoted child has `use_parent_name` false, when a field's `label` shows
nowhere because the field is promoted with `use_parent_name` true, and when a `promote` has no effect because its
struct has a `title`. <!-- rule: inline.lint.over-twelve class=judgment enforced=no tasks=field-presentation --> It warns when an `InlineChoices` field or type declares
more than 12 choices, which always use the [selector](#inline-choice-ladder). <!-- /rule --> <!-- rule: tpl.lint.view-screens class=judgment enforced=no tasks=display-template --> Studio additionally lints
more than about eight rendered (non-Hidden) fields
shown together on a screen reachable from a Request, Response, or
SubscriptionData payload or a function's configuration; a required field
(one the person must answer before saving: not Nullable, with no `Default`; a Hidden or ReadOnly field is never required, because the person cannot answer it)
folded closed while `form: phone` may apply; contrast by color
role; and a `split` whose regions leave a required field unlisted in a nested
region only.

<!-- rule: tpl.lint.fields-per-screen class=judgment enforced=no tasks=display-template -->
The fields-per-screen lint reports each Table once, under its own name and
source line; an inline Union branch is named `Union.Branch`. A screen sums
the fields of its nested Tables, so several small Tables add up. A field
whose content shows only on demand counts one row: a Union, whose chosen
branch is linted as its own screen, so a Union counts by its largest branch
and never the sum; a Nullable container; and a nested Table reported on its
own. The fields-per-screen lint applies only in `view` mode: an editor is
an ordinary scrolling form, where every field stays reachable in entry order,
so many fields on one editing page draw no warning. Studio decides the mode
from where a screen is reachable: from a Request whose node holds a field the
person can change it is `edit`; from a Response, SubscriptionData, or a
ReadOnly subtree it is `view`; a Table reachable both ways is linted for its
`view` use only. In `view` mode a node is exempt under `sections` with `tabs`
or `rows`, a `split`, or a fold that starts closed, from `display.json` or a
source `Foldable`. Exemption holds only on the forms its rule's `when.form`
names, so a rule limited to tablet and desktop still leaves one phone warning,
which says so, and only in the modes its rule's `when.mode` names; a rule
without `mode` applies in both.

## Common field metadata

<!-- rule: field.size-bounds class=hard enforced=no tasks=field-presentation -->
`SizeMin`/`SizeMax` follow the [shared size contract](schema.md#43-value-constraint-and-presentation-attributes):
List element count, String UTF-8 byte count, and Bytes byte count. Both TypeScript
`@LibertasSize(min=..., max=...)` and Rust
`#[libertas_size(min=..., max=...)]` emit these same attributes. The bounds are
inclusive; size zero is valid unless excluded by the minimum. Nullable null
remains separate from an empty value.

<!-- rule: field.multiline-editing class=hard enforced=no tasks=client-runtime title="A Multiline String is edited in a box that grows with its lines and captures Enter as a line break; any other String is edited on one line" -->
A [`Multiline`](schema.md#43-value-constraint-and-presentation-attributes) String
is edited in a text box that grows on demand: it starts at one line, gains a line
as the text needs it and shrinks when lines are removed. The box captures the
Enter (line-break) key and inserts a line break; Enter never submits, moves focus
or triggers another control while the box is focused. Every display keeps the
value's line breaks. Any other String is edited on one line:
the line-break key never inserts a break, and pasted text has each run of line
breaks replaced by one space before it is committed. Showing more lines changes
no value, validation or encoding.

<a id="secret-values"></a>

<!-- rule: field.secret-display class=hard enforced=no tasks=client-runtime title="A Secret String is masked wherever it is shown; only the Show control on its own editor or value reveals it" -->
A [`Secret`](schema.md#43-value-constraint-and-presentation-attributes) String
is masked wherever a client shows it. Its editor uses the platform's secure
text entry (a password field), which shows one mask character per character
typed and turns off suggestions, autocorrection, spell checking and keyboard
learning. Every other display of a nonempty value, read-only or not, shows
eight mask characters (`••••••••`) whatever its length: the value's own row,
a table cell, a summary or promoted value, and a title or heading that carries
it. An empty value shows as empty, and null as its null text. The editor and
the read-only row of the value each offer a Show control that reveals that one
value as plain text until the person hides it again or the page is rebuilt;
nothing else reveals it. Masking changes no value, validation or encoding, and
a Secret is edited on one line like any String without `Multiline`.

<!-- rule: field.picker-no-implicit-default class=hard enforced=no tasks=field-presentation -->
User and group pickers do not implicitly select the current user or a singleton
candidate. Declare an explicit default using the
[User and group default contract](schema.md#user-and-group-defaults).

| Meaning | TypeScript | Rust |
| --- | --- | --- |
| Initial value | `@LibertasDefault(value)` | `#[libertas_default(value)]` |
| Inline long description | `@LibertasUIIncludeDescription` | `#[libertas_ui_include_description]` |
| Not editable | `@LibertasReadOnly` | `#[libertas_read_only]` |
| Hidden from ordinary editor | `@LibertasHidden` | `#[libertas_hidden]` |
| Unordered List | `@LibertasUnordered` | `#[libertas_unordered]` |
| Multi-line text | `@LibertasMultiline` | `#[libertas_multiline]` |
| Masked confidential text | `@LibertasSecret` | `#[libertas_secret]` |
| Scoped uniqueness | `@LibertasUnique(level)` | `#[libertas_unique(level)]` |
| Size bounds | `@LibertasSize(min=...,max=...)` | `#[libertas_size(min=...,max=...)]` |
| Live Number/DateTime minimum | `@LibertasRangeMin("expression")` | `#[libertas_range_min("expression")]` |
| Live Number/DateTime maximum | `@LibertasRangeMax("expression")` | `#[libertas_range_max("expression")]` |
| Data-backed choice (index or key) | `@LibertasEnumSource("$.items")` | `#[libertas_enum_source("$.items")]` |
| Stable choice key of a List member | `@LibertasKey` | `#[libertas_key]` |
| Asked before a List member is added | `@LibertasAskOnAdd` | `#[libertas_ask_on_add]` |

The inline-description row is compiled into the
[Display Template](#display-template); the others are schema. A summary value
is a template [`promote`](#promotion-flattening-and-union-selectors)
rule, and flattening a [`flatten`](#flatten) rule; neither has a source
attribute.

<!-- rule: field.hidden-subtree class=hard enforced=no tasks=field-presentation -->
`Hidden` suppresses the complete value subtree, including list elements and
controls, nested fields, summaries, and promoted values. Hidden data remains
part of the typed value for validation and protocol encoding. `flatten` does
not expose hidden descendants. <!-- rule: field.promote-flatten-not-inherited class=hard enforced=no tasks=display-template --> `promote` and `flatten` apply to the node their
rule selects, are not inherited through ordinary attribute resolution, and
never override `Hidden`.

<!-- rule: field.readonly-user-only class=hard enforced=no tasks=field-presentation -->
`ReadOnly` restricts the user only: clients present the value and offer no
ordinary editing control for it. It never restricts the platform or the
application. Schema initialization (Fixed, Default, localized defaults),
`CopyFrom` from a Response or the reference context, `EditRequest` values,
calculation and link-return `CopyTo` results, draft restoration and the application's own
handling of the submitted value all write a ReadOnly field exactly as they
write an editable one. A value that must not change after initialization is
`Fixed`, not ReadOnly. Hidden fields are ReadOnly in effect, since no editor
shows them, and are written the same way.

<!-- rule: field.readonly-list-membership class=hard enforced=no tasks=field-presentation -->
`ReadOnly` on an Array/List prevents changing its membership or order: no Add,
Remove, or reorder action may mutate that list. It does not recursively make
member values read-only; members and their fields keep their own editability.
To prohibit member edits, mark the applicable member type or fields as well.
Context menus, swipe actions, pickers, and deferred actions must enforce the
same list-owner rule at both presentation and execution. A promoted row does
not change which list owns a removal or reorder operation.

For promotion, flattening, and nested Union layout, see
[Promotion, flattening, and Union selectors](#promotion-flattening-and-union-selectors).

<a id="add-time-values"></a>

<!-- rule: field.ask-on-add.meaning class=hard enforced=no tasks=field-presentation,client-runtime title="`AskOnAdd`: Add asks the marked values first; without them no member is added" -->
**Add-time values.** `AskOnAdd` marks a value the person gives before a List
member exists: a field of a List's member Table, reached from the member
through struct fields, never through another List; or a List field whose member
has a value of its own, which asks the member itself. A field's marker applies
wherever its Table is such a member. Any number of a member's fields may carry
it, of any type that has a value of its own: a scalar (String, Number, Boolean,
DateOnly, DateTime, Bytes), an Enumeration, Bitflags, or a Union. When the
person chooses **Add** on the List, the client first asks for every marked
value; confirming creates the member with those values and every other field at
its normal initialization, then shows the new member as the List's layout shows
any new member: a [summary](#summary-array-items) opens its item page, and
inline rows, a table, or an accordion show it in place. Without every value, no
member is added, and Cancel adds nothing.

<!-- rule: field.ask-on-add.generated-ui class=hard enforced=no tasks=client-runtime title="The platform builds the Add step from the marked nodes alone; the App declares no Add page" -->
**The platform builds the Add step.** The App declares nothing beyond the
markers. One marked choice (a Union, Enumeration, or Bitflags) is asked in the
[one-field popup](protocols.md#single-action-confirmation-and-input-presentation)
form of its chooser; anything else, one scalar or several values, is one
compact Add editor holding exactly the marked nodes, in the member's
declaration order, each drawn, labeled, and validated as in the member's own
editor, with its allowed choices, `RangeMin`/`RangeMax`, `Pattern`, validation
rules, and calculations, and the member's display template applied. A marked
Union asks its variant only; its payload is edited after Add.

<!-- rule: field.ask-on-add.value class=hard enforced=no tasks=client-runtime title="An add-time value is a real answer: never null, never empty, at least one flag" -->
- **A value is a real answer:** a Union variant, an Enumeration member, at
  least one Bitflags flag, or a scalar that is neither null nor an empty
  String or Bytes and that passes the field's own validation. A Nullable's
  null answer is not a value here.
- **Defaults prefill:** a marked value's `Default` is preselected in the Add
  step; the person still confirms it or changes it.
- **Allowed sets:** a marked choice whose allowed set is a singleton takes that
  choice, as [single-choice rows](#single-choice-rows) do, and is not asked;
  when nothing is left to ask, Add adds at once. An empty allowed set offers no
  Add.
- <!-- rule: field.ask-on-add.person-only class=hard enforced=no tasks=field-presentation,client-runtime --> **The person's Add only:** a member that AI Import, CopyTo, an EditRequest,
  or the App adds is unaffected, and after Add each value is an ordinary field;
  whether the person may change it is a template
  [`lock`](#presentation-locks)'s, a separate concept.
- The [categorized layout](#categorized-layout)'s **Manually add document** is this rule
  applied to its kind.
- <!-- rule: field.ask-on-add.with-lock class=judgment enforced=no tasks=field-presentation,display-template --> **Commonly paired with [`lock`](#presentation-locks).** The usual purpose of
  `AskOnAdd` is a value that decides what the member is, such as a document's
  form: the person gives it once at Add, and a template `lock` (on a Union,
  `lock: "variant"`) then keeps the person from changing it, so the item
  editor shows the chosen form and never offers to switch it.

<!-- rule: field.ask-on-add.placement class=hard enforced=yes tasks=data-model title="`AskOnAdd` goes on a field with a value of its own in a List member Table, or on a List whose member has one; never Hidden, ReadOnly, Fixed or OutputOnly" -->
Studio rejects `AskOnAdd` on a Table or List field of the member, on a field
below another List of the member, on a node that is itself Hidden, ReadOnly,
Fixed, or OutputOnly, on a List whose own membership is ReadOnly or whose member
is a Table, and on a marker no editable List member reaches. Where a marked
type is also used below a protected node, such as inside a ReadOnly view, no
person can Add there and the marker has no effect in that use, so one natural
type serves both.

<!-- /rule -->
[`EnumSource`](schema.md#7-path-based-object-and-choice-binding) belongs on an unsigned integer that stores the selected List
item's zero-based index. When the List's member declares a
[key field](schema.md#enumsource-keys), the host instead has the key's type
and stores the selected member's key value. It never stores the item itself.
A stored index or key that matches no current member leaves the field
unanswered, like a required field without a value; clients never remap it to
another member by position, key, or label.
AI tool-call descriptions carry its [value-representation hint](llm-document-transformation.md#value-representation-hints), which names the source List path
without enumerating its current items. <!-- rule: field.enumsource.picker-labels class=judgment enforced=no tasks=client-runtime -->
Android, iOS, and Studio present a single-choice picker in source order, select the
member at the stored index or with the stored key, and write the chosen
member's index or key. Every choice is labelled by its text alone, with no
position number: in precedence, the
member's Display Template [`title`](#labels-and-titles); else its promoted
field's value or, for a scalar member, the scalar's display value, formatted
as the tree formats it (money and explicit formats keep their normal display);
else the member's Display Template `label`; else its display name. A composite
member never reads through a nested value. A Hidden member is still listed so
that indexes stay stable: Hidden affects the member's own row, not the picker.

<!-- rule: field.enumsource.picker-entry-display class=judgment enforced=no tasks=field-presentation -->
**A picker entry draws its source member as the source List draws it.** When the
source List has a `summary` or `table` rule, each entry shows that rule's paths
or columns for its member, with the text above as the entry's first line, and
the filter matches that text too. The rule applies to the picker even when the
List itself is Hidden and never drawn. Without such a rule the entry is the text
alone. A host's chosen value, drawn outside the picker, shows only that text; a
hosting item that needs the fuller display carries it as its own ReadOnly,
OutputOnly field, normally of the same display-only type the source member uses,
so the item and the picker entry read the same.

<!-- rule: field.enumsource.list-host-chooser class=hard enforced=no tasks=client-runtime -->
A [List host](schema.md#enumsource-list-hosts) is edited with a
multiple-choice chooser over the same members, in the same order and with the
same labels, instead of List Add, Remove, and reorder controls. Each member the
host holds is checked. Choosing a member appends its index or key; unchoosing
one removes that element; the other elements and their order are unchanged.
An element that names no current member is listed after the members as an
unavailable, checked entry, so the person can remove it; it is never remapped.
A ReadOnly host shows the same chooser disabled.

<!-- rule: field.chooser-filter class=hard enforced=no tasks=client-runtime -->
A chooser with more than twelve permitted choices — this EnumSource picker, an
Enumeration picker, a [Bitflags](#bitflags) multiple-choice picker, or the
[Union variant chooser](#promotion-flattening-and-union-selectors) that
opens when a promoted or unpromoted value is tapped — shows a text filter
field above its list. Typing filters the visible choices by a case-insensitive
substring match against each choice's localized display name and description;
it never changes the permitted set, its order, or the selection, and clearing
the field shows every choice again. The current selection stays visible and
selected even when the query would otherwise filter it out.

<!-- rule: fields.compact-choice.preserve-semantics class=hard enforced=no tasks=client-runtime -->
The [hierarchical single-field request builder](protocols.md#hierarchical-single-field-requests)
also presents a choice-only Union/Enumeration hierarchy as expandable groups.
It derives grouping from the App's types and removes groups with no eligible
leaves recursively. This is the default compact request presentation for that
shape; ordinary content folding and the Display Template do not change the
underlying selection type, constraints or submission behavior.

<!-- rule: field.enumsource.source-path-and-singleton class=hard enforced=no tasks=display-template title="The `EnumSource` path reads the current data tree, never the reference context; a sole selectable member fills an unanswered non-nullable host" -->
The source uses a [Libertas Schema Path](schema.md#context-and-syntax) in the
current data tree; it does not read the interaction reference context. An explicit Default or
Fixed retains ordinary initialization; otherwise the selection is unanswered.
A non-nullable host whose source offers exactly one selectable member (a
member the picker offers: with its key set, within the host's bounds and step)
takes that member when it is unanswered, under the
[singleton rule](protocols.md#backend-constrained-choices) of allowed sets:
when its editor initializes it, and whenever the source changes while it is
still unanswered. A stored index or key is never replaced, and a List host is
never filled this way.
Nullable absence remains valid. A present selection must identify a current
List item and satisfy the field's numeric bounds and step, including dynamic
bounds. An empty or unavailable source offers no selectable options.

Path traversal follows the [shared resolution rules](schema.md#context-and-syntax),
including reusable named types, List/Union levels, and each enclosing use.
A selector in a reusable type is validated in configuration, protocol, and stored
data contexts where it occurs; its declaration does not introduce another root.

<!-- rule: template.runtime.row-replacement-stale-actions class=hard enforced=no tasks=client-runtime -->
Cancel preserves the previous value. A source-row replacement, reorder, removal,
label change, or change of a member's key value while the picker is open rejects that pending selection;
reopen the picker to choose from the current list. <!-- rule: field.enumsource.index-not-identity class=judgment enforced=no tasks=field-presentation --> A saved index remains a
position, not a persistent record identity. Apps that need record identity
must resolve the selection against the appropriate accepted-data revision.

<!-- /rule -->
`RangeMin` and `RangeMax` constrain numeric controls and date/time choosers
with the same inclusive limits used for final validation. Their expressions
must produce bounds compatible with the control’s semantic type; see
[dynamic numeric ranges](validation-rules.md#31-dynamic-numeric-ranges) for
result typing, unavailable bounds, and intersection with static constraints.

<!-- /rule -->
[`UIIncludeDescription`](schema.md#43-value-constraint-and-presentation-attributes) is a no-argument, client-only presentation marker. The
UI node keeps its localized short `DisplayName` and also renders its localized
long `Description` inline. Without the marker, clients retain their ordinary
compact short-name presentation, including a description tooltip where one is
available. <!-- rule: pres.include-description.no-help-icon class=hard enforced=no tasks=client-runtime title="A node whose description is rendered inline shows no description tooltip or help icon" --> When the description is rendered inline, the client shows no separate
description tooltip or help icon for that node, because the text is already on
the page; the inline text then uses the row's full width. The marker does not affect stored values, requests, or wire encoding.
Studio compiles it into the [Display Template](#display-template) as the
`include_description` hint, which clients read at runtime.

For `Unordered`, AI descriptions carry its [value-representation hint](llm-document-transformation.md#value-representation-hints).

## MIME-aware Bytes presentation

<!-- rule: media.control class=hard enforced=no tasks=client-runtime -->
A `Bytes` field with effective [`MimeTypes`](schema.md#mime-aware-bytes) uses
one media control in Android,
iOS, Studio, and the pure Web client. Its compact row shows the field name, a
bounded thumbnail for a safely decodable raster image or a generic media
icon, the exact MIME and content size when defined, and no local path.
Undefined media shows “No file selected.” It never displays Base64 or hex.
Ordinary Bytes keep their existing editor.

<!-- rule: media.invalid.generic-icon-and-error class=hard enforced=no tasks=client-runtime -->
Editable media rows offer Choose file, Remove, and Take photo when the field
accepts an image format that the client's camera can produce. The camera
stores its output as a private temporary file and the editor holds a local
link until outbound encoding. Camera permission denial or cancellation leaves
the previous value intact; Choose file remains available. Read-only rows
offer preview when supported. Tapping or clicking the thumbnail opens a larger
platform preview. A corrupt or unsupported image shows the generic icon and
an error when previewed; clients do not render SVG or arbitrary HTML as an
inline image. Local paths and temporary link identifiers are not displayed or
sent to the Hub. Camera availability is conditional on device, browser, and
host permissions.

A missing temporary file during draft recovery makes the field undefined and
identifies it for reselection. A file that disappears at submission produces a
field error and stops that attempt. The [draft and cleanup rules](annotations.md#mime-aware-bytes-in-workflow-drafts)
and [LMB1 contract](schema.md#mime-aware-bytes) own those behaviors. The
standard media control and explicit preview require no template rule.

<!-- rule: media.inline.pdf.authoring class=hard enforced=no tasks=field-presentation -->
Choose media and size with the [platform preview limits](#inline-media-size) in mind: PDF previews vary by client and video uses an icon.

<!-- /rule -->

<a id="inline-media-size"></a>

### Inline media size

A Display Template [`media`](#behavior-hints) rule draws the field's content
larger, inline in the same media row. Without a rule, or with
`"max_height": "thumbnail"`, the row keeps the bounded thumbnail above. With
`small`, `medium`, or `large`, the media draws on its own line below the
field's label and summary, before its controls:

- <!-- rule: media.inline.intrinsic-size-limit class=hard enforced=no tasks=client-runtime --> **Fit.** It uses the full width available to the value in its pane,
  `form-grid` cell or summary grid cell, preserves its aspect ratio, and is never drawn larger than
  its intrinsic size, one image pixel per logical point (dp on Android, pt on
  iOS, CSS px in Studio). Its height is capped by the
  [height token](#height-tokens): about 25%, 40%, or 60% of the nearest
  pane's visible height. A wide image therefore fills the width; a tall one
  stops at the cap and is centered. The cap follows the pane as it resizes.
- <!-- rule: media.inline.aspect class=hard enforced=no tasks=client-runtime --> **Aspect box.** With `aspect` (`square` 1:1, `landscape` 4:3, `portrait`
  3:4), the media draws on its own line in a box of the available width at
  that aspect, still capped by `max_height` when one is given (`aspect` alone
  implies `medium`). An image larger than the box is scaled down to fill it and
  cropped at the center, never stretched; a smaller one is centered at its
  intrinsic size. An absent or undrawable value reserves the same box with a
  neutral placeholder, so aligned layouts such as a [summary grid](#summary-grid)
  keep their rows even.
- <!-- rule: media.inline.images class=judgment enforced=no tasks=field-presentation --> **Images.** The raster formats the thumbnail decodes draw inline. SVG and
  every other vector or markup format are never rendered, at any size.
- <!-- rule: media.inline.pdf class=judgment enforced=no tasks=client-runtime --> **PDF.** `application/pdf` draws its first page, rasterized at the fitted
  size, under the same fit and cap; the page's intrinsic size is its media box
  in PDF points. Android uses the platform `PdfRenderer` and iOS PDFKit. Studio
  has no PDF page renderer and shows the generic media icon for PDF. A PDF
  that is encrypted, has no page, or fails to render shows the generic icon.
  The `thumbnail` size keeps its existing behavior and does not render PDF
  pages.
- <!-- rule: media.inline.video class=judgment enforced=no tasks=client-runtime --> **Video.** Video is reserved for a later inline player. Until a client
  implements it, a `video/*` value keeps the thumbnail-size generic media icon
  under every token.
- **Interaction.** Tapping or clicking the inline media still opens the full
  platform preview, and the row's Choose file, Remove, Take photo, and Preview
  controls are unchanged. Corrupt or unsupported content keeps the generic icon
  and the existing preview error. Decoding happens off the main thread, and a
  client may show the generic icon until the bytes are decoded.

<!-- rule: media.presentation.preserve-business-values class=hard enforced=no tasks=client-runtime -->
`media` is presentation only: it never changes the stored bytes, their MIME
type, validation, `CopyFrom` or `CopyTo`, encoding, or AI Import.

## Promotion, flattening, and Union selectors

Two Display Template members change how a node's children sit beside its row:
`promote` puts one child field's value on its parent row, and
[`flatten`](#flatten) draws a node's children at that node's own indentation.
Neither changes the encoded data hierarchy. <!-- /rule -->
`when` may change either per
device; no [data condition](#display-template-conditions), switch case, or pseudo-state sets either.

<!-- rule: promote.placement class=hard enforced=yes tasks=display-template title="`promote` is valid only on a rule selecting a field of a struct; rejected on a type, List member, Union branch, function parameter or Response" -->
`promote` is valid only on a rule that selects a field of a struct (a Table),
for example `^<TopicRow>.status`, which selects that field wherever the
`TopicRow` type is used, or a strict path to one field. Studio rejects
`promote` on a rule that selects a type, a List member, a Union branch, a
function parameter, which has no parent row, or a Response. Its value is `true`, `false`, or `{ "use_parent_name": Boolean }`; `true` means
`use_parent_name` true, and `false` turns promotion off for the field, so a
higher-ranked rule can undo a lower one. <!-- rule: promote.does-not-flatten class=hard enforced=no tasks=display-template --> `promote` never flattens the field's
children: select the node with `flatten` when wanted. Both members are authored
only in `display.json`. See [removed annotations](annotations.md#removed-annotations)
when migrating older source.

### Field promotion

<!-- rule: promote.first-visible-field class=hard enforced=no tasks=display-template -->
On a field directly owned by a **Table**, the first visible promoted field in
schema order becomes the Table row's value instead of occupying a separate
child row. Additional promoted fields remain ordinary children. Promotion has
no effect when the field's owner is not a Table, including a Union's selected
branch.

The promoted field supplies the row value. Its `use_parent_name` option chooses
the row label, uniformly for scalar, Table, List, and Union fields:

- <!-- rule: promote.use-parent-name.true class=hard enforced=no tasks=display-template --> `true` (the default): use the owning struct's displayed name, which is its
  [`label`](#labels-and-titles) when a rule gives it one.
- <!-- rule: promote.use-parent-name.false class=hard enforced=no tasks=display-template --> `false`: use the promoted field's own displayed name, which is its `label`
  when a rule gives it one. A `label` on the owning struct then shows
  nowhere, and Studio warns.

When the owning struct has a [`title`](#labels-and-titles), the field is not
promoted and keeps its own row, and Studio warns that the `promote` has no
effect.

For a promoted Union, the selected variant's display name supplies the row value;
the option changes the label, not that selected value. Tapping that value opens
the choice selector, unless the Union shows [inline choices](#inline-choices).
Selecting another variant replaces the applicable fields. <!-- rule: promote.inline-union-placement class=hard enforced=no tasks=client-runtime --> Promotion never
moves a Union's selected fields: with inline choices they stand below the
chosen choice or after the whole group exactly as
[inline choices](#inline-choices) places them for an unpromoted Union. <!-- /rule -->
The selector remains available even when the selected variant has no fields. <!-- rule: promote.list-member-position class=hard enforced=no tasks=display-template -->
A List member whose chosen row label is static text (a displayed name, or a
`label` in the `"KEY"` form) appends its one-based position to it (for example,
“Household #1” with the default or “Marital status #1” with `use_parent_name`
false), including for a Union. A label taken from the member's data (the
`{ "field": … }` form) already tells the members apart, so it gets no
position. <!-- /rule -->
A scalar promoted value keeps its normal formatting and editor. A composite
one keeps its children, and a List retains its entries and Add action.

An unpromoted Union retains its field title after selection. For example,
“Marital situation” remains the label and “Married” is the selectable value.
A Union with a scalar payload also identifies the selected branch while keeping
that payload editable. Branch selection remains available independently of
promotion, including after choosing a fieldless variant.

### Name option

```json
{ "select": "^<Schedule>.starts_at", "promote": true }
{ "select": "^<HouseHold>.marital_status", "promote": { "use_parent_name": false } }
```

Both rules promote a field, the first labelled by its owner's name and the
second by its own.

### Example: use the parent name to identify the record

A struct has the display name **Next schedule**. Its **Starts at** field is
promoted with `use_parent_name` true (or `promote: true`), and **duration** is
an ordinary sibling field. For a start time displayed as `09:00` and a duration
of `30 minutes`, the rows are:

```text
Next schedule: 09:00
  duration: 30 minutes
```

The parent name tells the reader what the time and duration describe: the next
schedule. Using the field name instead would produce `Starts at: 09:00`, which
describes the time but loses the record's identity when several schedules or
other timed activities appear on the same screen. The promoted start time acts
as the schedule's summary value, so the parent name is useful here.

### Example: use the field name to identify the property

A struct has the display name **HouseHold**. Its **Marital status** field is a
Union promoted with `use_parent_name` false. The selected variant has the
display name **Married**, and **Number of dependents** is an ordinary sibling
field. The rows are:

```text
Marital status: Married
  Number of dependents: 2
```

The field name tells the reader exactly which property “Married” answers.
The default would produce `HouseHold: Married`; “HouseHold” identifies the
record but does not name that property. Keeping **Marital status** makes the
selected choice understandable without requiring the reader to infer the
question from its answer.

In both examples, the option changes only the promoted row's label. The field
still supplies its value, sibling fields retain their normal placement, and
the Union selector still changes the selected variant.

<a id="flatten"></a>

### Flattening

The rule member `flatten` applies to a struct, an array (List), or a Union. Its
rule selects the node by type (`^<Address>`, applying wherever the type is
used) or by field (`^<Household>.address`). It is a rule member like `layout`,
not a `style` member, so no data condition, switch case, or pseudo-state can
change it; `when` may. Its value is `true`, `false`, or
`{ "name_row": false }`. <!-- rule: flatten.false-undoes class=hard enforced=no tasks=display-template --> `false` undoes a flattening that a lower-ranked rule
set in the [cascade](#display-template-cascade), such as a dependency
package's template for its own type: the node draws with normal indentation
and its name row.

- <!-- rule: flatten.struct class=hard enforced=no tasks=display-template --> **Struct.** `true` draws the struct's fields at the struct row's indentation
  and keeps the struct's name row. `{ "name_row": false }` also removes that
  name-only row; the fields stay visible and editable at the flattened level.
  A List member whose struct row is removed keeps its position and membership
  controls, and the fold control of a removed row passes to the first row the
  struct still produces, as [folding](#folding-structs-and-unions) describes.
- <!-- rule: flatten.list class=hard enforced=no tasks=display-template --> **Array (List).** The List row keeps its title, count, and Add action; its
  members sit at the List's indentation instead of one level in. Each member
  keeps its own row, position, and actions.
- <!-- rule: flatten.union class=hard enforced=no tasks=display-template --> **Union.** The selected branch's fields sit at the Union row's indentation;
  the selector stays visible and editable.

<!-- rule: flatten.placement class=hard enforced=yes tasks=display-template title="`flatten` selects a struct, List or Union, never a List member or Union branch; `name_row` only on a struct without `title`" -->
Studio rejects `flatten` on any other node kind; on a List member (`…[]`)
and on a Union branch, because neither has a row of its own to flatten, with
a message that names the owning List or Union; `name_row` on a List or a
Union; and `{ "name_row": false }` on a struct that has a
[`title`](#labels-and-titles), because one removes the name row and the other
replaces it. <!-- rule: flatten.default-indentation class=hard enforced=no tasks=display-template --> Without a `flatten` rule, children keep normal indentation;
protocol page roots follow the [automatic root alignment](#protocol-page-heading-and-top-bar).
`flatten` never promotes a field, including a field that references the
flattened type: select that field with `promote` when wanted. `Hidden` still
suppresses the whole subtree.

```json
{ "select": "^<Dependents>", "flatten": true }
{ "select": "^<Zone>", "flatten": { "name_row": false } }
```

For example, these independent rules combine promotion and flattening:

```text
Household (Table)
  dependents (display name: Dependents; type: Dependents)

Dependents (Union)
  No
  Yes (Table)
    count (Number)
```

```json
{ "select": "^<Household>.dependents", "promote": true }
{ "select": "^<Dependents>", "flatten": true }
```

With `Yes` selected, the schematic rows are:

```text
Household: Yes             [tap value to change choice]
Count: ...
```

Removing the `flatten` rule indents `Count` under `Household`. Removing the
`promote` rule restores a separate Union row under `Household`. The choices
can be repeated at each nesting level. An unrelated sibling is not flattened
because another field is a promoted or flattened Union. Existing expansion,
visibility, and read-only rules still apply; neither member makes a read-only
selector editable.

### Actions on a promoted or suppressed row

<!-- rule: promote.actions-follow-value class=hard enforced=no tasks=display-template -->
Attributes that add an interaction to a value's row, currently `Link` and
`LLMDocTrans` (AI Import), follow that value when `promote` or `flatten`
changes its presentation. A promoted field's action appears on the owning
Table's visible row. If a Table has no row of its own, because `flatten` with
`{ "name_row": false }` removed it, its action appears on the first visible
row that represents that Table; when the Table is the selected payload of a
promoted Union, this is the Union's owning row. A List member keeps its own
row and its own actions, also inside a flattened List. If several values
represented by one row declare actions, the row offers each applicable action;
a child `Link` takes precedence over an enclosing `Link` for the row tap
target. With a [`label`](#labels-and-titles) on the struct, a promoted value's
link stays on the row; with a `title`, a link on the struct stays on the title
row.

<!-- rule: promote.actions.only-affordance-moves class=hard enforced=no tasks=display-template -->
Only the **affordance** moves. Each action retains its declaring value, path,
request context, eligibility, and pending state. Neither `promote` nor
`flatten` copies an attribute to another schema or value node. `Hidden`
suppresses the action with its value subtree; `ReadOnly`, `Fixed`, and other
action-specific restrictions still apply. Data, validation, access, and
lifecycle attributes do not move.

<!-- rule: promote.classifier-import-action class=judgment enforced=no tasks=display-template -->
The **AI Import** action of a
[classifier](llm-document-transformation.md#categorized-import) follows this
placement. A classifier that is the
Request root has no row of its own, so its action is the editor's first row,
a full-width action. Outside [categorized layout](#categorized-layout),
category Lists keep their ordinary layout and controls. Categorized layout
centralizes import and manual document creation at its outer classifier.

### Presentation only

<!-- rule: promote.presentation-only class=hard enforced=no tasks=display-template -->
Promotion and flattening do not change the schema or encoded data hierarchy.
`CopyFrom`, `CopyTo`, `EnumSource`, validation paths, and backend field access
still use the original structural paths, including every nested Union/table
level. They do not supply a default, make required fields optional, or disable
fields based on another answer.

<!-- rule: promote.protocol-actions-owner class=hard enforced=no tasks=client-runtime -->
Protocol actions remain owned by the enclosing protocol variant, even when its
header is another Union. See [protocol transaction stack](endpoint-presentation.md#protocol-transaction-stack).
Separate same-indent Union rows retain their own edit/removal ownership;
sharing an indentation level does not make them the same data object.

## Data-titled rows

<!-- rule: label.data-titled-rows class=judgment enforced=no tasks=display-template -->
A row is **data-titled** when its label comes from the data it shows, such as
a calculated line labelled by its description with its amount as the value.
Two independent rules express it:

- <!-- rule: label.data-titled.label-rule class=judgment enforced=no tasks=display-template --> a Display Template [`label`](#labels-and-titles) rule on the row's struct, usually
  `"select": "^<Type>"` or, for a List's members, `"^<Response.list>[]"`, names
  the field whose value labels the row;
- <!-- rule: label.data-titled.promote-rule class=judgment enforced=no tasks=display-template --> a [`promote`](#field-promotion) rule on a child of the struct promotes that
  child's value into the row.

```json
{ "select": "^<LineItem>", "label": { "field": "description" } }
{ "select": "^<LineItem>.amount", "promote": true }
```

With `description` Hidden, each `LineItem` renders as one row, "W-2 wages
$60,000.00", and its other visible children render beneath it. The names are
schematic.

Each rule also works alone: a label without a promoted child labels a row
whose children render beneath it, and a promoted child without a label keeps
the struct's display name. With the default `use_parent_name`, the row's
label is the struct's displayed name, which is the `label` text; with
`use_parent_name` false it is the child's own name, the `label` shows nowhere,
and Studio warns. Folding, flattening,
links, and a List member's membership controls stay on the row as they do on
any other row. <!-- rule: label.title-for-text-only-rows class=judgment enforced=no tasks=display-template --> A row that should show only text, with no value, such as a
heading taken from the data, uses [`title`](#labels-and-titles) instead; a
`title` takes no promoted child, so the child keeps its own row.

Both rules are authored only in `display.json`. See
[removed annotations](annotations.md#removed-annotations) when migrating older source.

### SuppressLabel: label-free value rows

<!-- rule: suppress.source-and-hint class=hard enforced=no tasks=field-presentation -->
`@LibertasSuppressLabel` in TypeScript and `#[libertas_suppress_label]` in
Rust suppress the visual label contributed by the selected node. The parser
result carries `{"Name":"SuppressLabel","Value":"true"}`; Studio compiles
it into the [Display Template](#display-template) as the Boolean
`suppress_label` hint and removes it from the final schema. A template rule
can set `false` to restore the ordinary label under the
[cascade](#display-template-cascade). The rule does not propagate to children.

| Selected node | With `suppress_label: true` |
| --- | --- |
| Scalar value, including an Enumeration | Omit its row label; keep its value and editor. |
| Struct | Omit its own row label; keep its promoted value, visible children, and actions. A struct without a promoted value keeps its children and any row controls. |
| Union | Omit the Union's own label; keep the localized selected variant value, selector, visible payload, and actions. An unselected editable Union keeps its selection affordance. |
| List whose effective member type is a struct (Table) | Omit the member name and position contributed by the List to each member row; keep each member's own template label or title, promoted value, visible children, and controls. Keep the List's heading, count, and membership actions. |

<!-- /rule -->
[Other List member types and non-value declarations](schema.md#43-value-constraint-and-presentation-attributes) are invalid hosts. Studio
rejects them; source syntax does not extend the placement rules. List
suppression retains the previous List-of-struct behavior, rather than
suppressing the List's own heading.

<!-- rule: suppress.label-only class=hard enforced=no tasks=field-presentation -->
The hint suppresses a visual label, never a value or subtree. It does not
change folding, editing, links, menus, membership permissions, help, or the
semantic name used for accessibility and editor dialogs. A control without a
visible label still has its localized accessible name. On a promoted row,
only its displayed label is omitted; the promoted value and its declaring
link remain available under the [action rule](#actions-on-a-promoted-or-suppressed-row).
`Hidden` continues to suppress the entire presentation subtree and wins
before every template rule.

<!-- rule: suppress.empty-row-no-space class=hard enforced=no tasks=client-runtime title="A row left with nothing to draw takes no space and draws no divider" -->
A row that is left with nothing to draw takes no space and draws no divider.
The usual case is a flattened struct's name row whose label is suppressed and
which has no promoted value, row control, fold control or message. Its
children keep their own rows, and nothing else about the struct changes.

<!-- /rule -->
[An explicit `label` or `title`](#labels-and-titles) and `suppress_label` are mutually exclusive on
one selected node, including `false`; List suppression can still coexist
with a `label` or `title` on its distinct member nodes. <!-- rule: suppress.suppress-title-retired class=hard enforced=yes tasks=display-template title="`SuppressTitle` and `suppress_title` are retired and rejected" --> The old source names
`@LibertasSuppressTitle` and `#[libertas_suppress_title]`, parser attribute
`SuppressTitle`, and template member `suppress_title` are retired authoring
forms. Modern compilation rejects them and names their corresponding
replacement; they are not aliases.

A reader of an already validated, retained compiled Display Template accepts
its historical `suppress_title` Boolean solely for the former List-of-struct
member-name and position behavior. At document ingress, translate that member
to `suppress_label` only when the rule has no `suppress_label`; an explicit
modern member, including `false`, wins. Normalize a runtime copy without
rewriting the retained document bytes or their digest. This isolated artifact
reader does not accept the old spelling in new source or `display.json`, and
renderers consume only the modern hint.

#### Protocol page heading and top bar

<!-- rule: heading.platform-owned class=hard enforced=no tasks=client-runtime -->
The title naming the active protocol page is its screen heading on every
client, including editable Requests, workflow editors and Responses. The first
content row shows it in the platform's heading typography: larger,
semibold text in the normal foreground color, wrapping for long localized
titles, and extra space before the fields below. Omit the ordinary field-row
divider and expose the title as an accessibility heading. Keep its existing
controls and the typography of nested fields. This treatment is platform-owned
and requires no App attribute.

<!-- rule: heading.content-always class=hard enforced=no tasks=client-runtime title="The first content row always shows the complete page title and scrolls with the content; the top bar repeats it at all times and may shorten it" -->
The first content row always shows the complete title, on every page, so
pages look alike whatever the length of their titles. It scrolls with the
content. The top bar also shows the title at all times, as a window title
does, and may shorten it with an end ellipsis. A title that fits in the top
bar does not remove the content heading. The fit rules below decide only the
top-bar title's form and where Actions goes.

<!-- rule: heading.root-alignment class=hard enforced=no tasks=client-runtime -->
The content heading and the page's top-level content share the
same leading edge within normal page or pane padding. Neither the root protocol
Union nor its selected Request or Response payload adds an indentation level. This automatic root
alignment applies to full-page Requests, workflow editors, and Responses, and
independently inside each master-detail pane. The master-detail arrangement
uses the full content width within the working surface's normal padding,
without a root indent around the pair of panes. Root alignment is independent
of `label`, `title`, and `suppress_label`, and requires no `flatten` rule;
`flatten: false` retains this normal root alignment. Nested structs, Lists,
and Unions continue to follow their ordinary indentation and template rules.

<!-- rule: heading.root-suppressed-variant-title class=hard enforced=no tasks=display-template -->
When the displayed root protocol Union has `suppress_label: true`, the page
heading and its native top bar use the localized selected variant name,
without the protocol label. Resolve an explicit `title` or `label` rule on
the displayed Request or Response variant first, using the same rendering
and empty-value fallback as [labels and titles](#labels-and-titles). Otherwise
use that variant's localized display name. With no selected variant, retain
the ordinary localized protocol title rather than a blank heading.

<!-- rule: heading.active-value-owns-title.authoring class=hard enforced=no tasks=display-template -->
Target the displayed Request or Response variant with title rules under the [active-page title contract](#protocol-page-heading-and-top-bar).

<!-- /rule -->

<!-- rule: heading.active-value-owns-title class=judgment enforced=no tasks=client-runtime -->
The title belongs to the active displayed value. Apply this rule to editable
Request pages, including workflow editors, as well as Response pages; do not
take an editor's title from its originating Response. An explicit variant
rule also titles that active page when root suppression is absent.
For example, a protocol labelled “Tax return” with selected variant “About
you” uses “About you” as its page title under:

```json
{ "select": "^<TaxProtocol>", "suppress_label": true }
```

<!-- rule: heading.actual-protocol-type-selector class=judgment enforced=no tasks=display-template -->
The selector in an App must use its actual protocol type name. No duplicated
resource key or field is needed solely to remove “Tax return”.

<!-- rule: heading.adaptive-actions-placement class=judgment enforced=no tasks=client-runtime -->
**Page buttons and the top bar.** Apply this rule on every client
to a protocol page with its own screen top bar, at any screen width. It is
independent of the pane threshold for two-panel layouts. **Actions** is the
control for the displayed variant's eligible `NextRequest` choices, under the
[request-selection rules](endpoint-presentation.md#protocol-transaction-stack);
it is separate from an editor's primary Next/Submit and previous-action
controls. Render its full localized caption, resolved by the
[Actions caption precedence](#named-action-controls) (Actions by default, or
the active page's [button label](#button-labels)), and retain the platform's normal
button target and spacing. Actions and the optional
[named controls](#named-action-controls) form one group, placed by the rules
below; an `action_controls` placement of `secondary` keeps the group in the
[named-control region](#named-action-controls) instead.

<!-- rule: heading.buttons-before-title class=hard enforced=no tasks=client-runtime title="The complete button group goes in the top bar whenever it fits there; the top-bar title takes the space the buttons leave" -->
**Buttons come before the top-bar title.** The top-bar title is a window
title; the page title is the content heading. Put the complete button group in
the top bar whenever the complete group fits there after native navigation,
system insets and the bar's other controls take their space. The top-bar title
then takes whatever space remains, with an end ellipsis, and is omitted when
none remains. The title never keeps a button out of the top bar.

<!-- rule: heading.fit.preserve-font-and-targets class=hard enforced=no tasks=client-runtime -->
Measure the buttons with their actual localized captions, current font,
weight, and accessibility text size. **Fits** means every button of the group
renders its complete caption on one line at its normal target size in the
normal top-bar layout; never reduce a font size or a button target, shorten a
caption, or promote a smaller subset to make the group fit.

<!-- rule: heading.buttons-pinned class=hard enforced=no tasks=client-runtime title="Page buttons never scroll away: the top bar is pinned and a button row below the title stays under it when the title scrolls out" -->
**Page buttons never scroll away.** The top bar stays in place while the
content scrolls. When the group does not fit the top bar, the complete group
goes on the [secondary row](#named-action-controls) below the content heading;
that row stays directly under the top bar when the heading scrolls out, so
only the title leaves the screen. A button is in exactly one of these two
places, never both, and never shares the content title row. With no eligible
button the group is absent and takes no space. The content heading row keeps
any Union selector, promoted value, other row control, validation message, and
requested inline description, with their ordinary behavior, and content keeps
the root alignment and normal page padding above.

<!-- rule: heading.description.accessible-action class=hard enforced=no tasks=client-runtime -->
**Description and accessibility.** A top-bar title supports the same long
press as a regular title row and opens the same localized description popup.
Keep a keyboard-accessible and screen-reader-accessible **Show description**
action when a description is available; help must not depend on hover or
long press alone. The content heading is the page heading for assistive
technology, with its full accessible name; the top-bar title adds no second
heading. Description lookup remains bound to the same active page when its
top-bar title is shortened.

<!-- rule: heading.width.measure-native-slots class=hard enforced=no tasks=client-runtime -->
In a [master-detail arrangement](#master-detail-panes), only the leading page
uses the working surface's top bar. The trailing page has no bar of its own:
it keeps its own content heading, and its eligible buttons use the secondary
row below that heading, which stays at the top of the pane when the heading
scrolls out; selecting it does not take over the shared title or toolbar. A
page shown on the whole working surface after narrowing uses the same
placement rule for its own title and buttons. Re-evaluate the placement when
the available width, control availability, template, locale,
font, text size or device conditions change, including rotation and window
resizing. Restore the group to the top bar when it fits again. Measure actual
localized controls and native slots, never a device-category breakpoint or a
fixed estimate for system controls. Conditional captions reserve the
[widest possible caption](#button-labels). Preserve logical order in right-to-left
layouts and the full accessible names. Re-layout preserves values, focus,
interaction and reference-context ownership, and pending-operation
restrictions, and sends no request.

### Presentation only

Encoding, paths, `CopyFrom`, `CopyTo`, `EnumSource`, validation, comparison,
digests, ordering, uniqueness and generated documents are unchanged. The
marker supplies no default and makes no field optional.

## Date and time

| Type | Raw meaning | Presentation |
| --- | --- | --- |
| <!-- rule: datetime.types class=hard enforced=no tasks=formatted-output title="`LibertasDateTime` epoch seconds (viewer-local); `LibertasTimeOnly` seconds within a day; `DateOnly` integer `yyyyMMdd`; `TimeInterval` duration seconds" --> `LibertasDateTime` | Epoch seconds | Viewer-local date/time; optional timezone in `Format`. |
| `LibertasTimeOnly` | `0 <= seconds < 86400` | Time chooser, no date. |
| integer plus `DateOnly` | Valid decimal Gregorian `yyyyMMdd` | Calendar-date chooser, no timezone. |
| number plus `TimeInterval` | Duration seconds | Duration editor/label. |

<!-- rule: datetime.date-only-calendar class=hard enforced=no tasks=formatted-output title="Never convert `DateOnly` through an epoch or timezone; every value is a valid calendar date" -->
Never convert `DateOnly` through an epoch or timezone. Defaults, fixed values,
requests, and runtime data must all be valid calendar dates.

AI descriptions carry each type's [value-representation hint](llm-document-transformation.md#value-representation-hints),
such as a `YYYY-MM-DD` string for `DateOnly`. These representations leave the
raw App values in the table above unchanged.

## Display formats

<!-- rule: fmt.one-portable-pattern class=hard enforced=yes tasks=formatted-output title="`Format` is one portable Unicode LDML pattern that never changes parsing, storage or chart geometry; it compiles into the template `format` hint" -->
`Format` accepts one double-quoted, programming-language-neutral Unicode LDML
pattern. It never changes parsing, storage, comparison, bounds, defaults,
requests, persistence, or chart geometry. Studio validates the pattern, compiles
it into the [Display Template](#display-template) as the `format` hint, and
removes it from the final schema; clients read the display pattern only from
the template, where a rule may choose another portable pattern per device.

```typescript
/**
 * Invoice total
 * Shows the billed total with two decimal places and parentheses when it is negative.
 * @LibertasNumber(type=f64)
 * @LibertasFormat("#,##0.00;(#,##0.00)")
 */
amount: number;

/**
 * Service date
 * Identifies the calendar day when the service occurs.
 * @LibertasNumber(type=u32)
 * @LibertasDateOnly
 * @LibertasFormat("yyyy-MM-dd")
 */
serviceDay: number;
```

```rust
/// Invoice total
/// Shows the billed total with two decimal places and parentheses when it is negative.
#[libertas_format("#,##0.00;(#,##0.00)")]
pub amount: f64,

/// Service date
/// Identifies the calendar day when the service occurs.
#[libertas_date_only]
#[libertas_format("yyyy-MM-dd")]
pub service_day: u32,
```

<!-- rule: fmt.number-grammar class=hard enforced=yes tasks=formatted-output title="Portable number grammar: `0 # . , ;` subpatterns, `%`, `‰`, `E`, affixes, apostrophe quoting; no `@`, `*`, wider currency tokens or rounding increments" -->
The portable number grammar supports `0`, `#`, `.`, `,`, positive/negative
subpatterns separated by `;`, `%`, `‰`, `E`, affixes, and apostrophe quoting. <!-- rule: fmt.currency-needs-money class=hard enforced=yes tasks=formatted-output title="`¤` and `¤¤` require explicit `Money` metadata" -->
Currency symbol `¤` and currency code `¤¤` are supported with explicit
`Money` metadata. Significant digit `@`, padding `*`, wider currency tokens,
and rounding-increment digits are not supported.

<!-- rule: fmt.date-time-fields class=hard enforced=yes tasks=formatted-output title="Date fields `G y M d E`, time fields `H h m s S a`, `z` only for DateTime; DateOnly date fields only, TimeOnly time fields only, never `H` with `h`" -->
Date fields are `G y M d E`; time fields are `H h m s S a`; `z` is available
only for DateTime. DateOnly accepts only date fields, TimeOnly only time fields,
and one pattern cannot combine `H` with `h`.

References:

- [UTS #35 number patterns](https://unicode.org/reports/tr35/tr35-numbers.html#Number_Format_Patterns)
- [UTS #35 date/time patterns](https://unicode.org/reports/tr35/tr35-dates.html#Date_Format_Patterns)

## Money

<!-- rule: money.i64-minor-units class=hard enforced=yes tasks=formatted-output title="`Money` goes on a signed `i64` Number storing minor units; the currency code and scale are schema metadata" -->
Use `Money` on a signed `i64` Number to store an exact amount in minor units.
The currency code and scale are explicit schema metadata: `USD/2` means that
integer `12345` represents USD 123.45. Rust storage and Avro remain `i64`,
not a decimal string or floating-point amount. Clients may retain an integer
string internally to avoid a JavaScript `Number` conversion.

AI descriptions carry the Money [value-representation hint](llm-document-transformation.md#value-representation-hints) with the declared currency and scale.

```rust
/// Wage amount in USD cents.
#[libertas_money("USD", 2)]
pub wages: i64,
```

```typescript
/** @LibertasNumber(type=i64) @LibertasMoney("USD", 2) */
wages: number;
```

<!-- rule: money.code-and-scale class=hard enforced=yes tasks=formatted-output title="The currency code is three uppercase letters and the scale 0 through 6; choose the currency's minor-unit scale" -->
Both annotations emit `{"Name":"Money","Value":"USD/2"}`. The code is three
uppercase ASCII letters identifying the currency; the scale is an integer
from 0 through 6. Choose the currency's applicable minor-unit scale, such as
USD/2, JPY/0, or KWD/3. Locale and currency-data updates never alter that saved
meaning. This uses [CLDR currency formats and metadata](https://unicode.org/reports/tr35/tr35-numbers.html#Currencies);
CLDR does not prescribe the application's numeric storage type.

<!-- rule: money.currency-pattern class=hard enforced=yes tasks=formatted-output title="A currency pattern controls presentation only, rounds half up for display, and cannot use percent, per-mille or scientific notation" -->
Without `Format`, the viewer's locale supplies currency placement, symbol,
digits, and separators, with the declared scale. An optional currency pattern,
for example `¤#,##0.00;(¤#,##0.00)` or `¤¤ #,##0.00`, controls presentation only.
Currency-pattern display rounding uses half up; it never rounds the stored
amount or implements tax-law rounding. Quoted `¤` remains literal text.
Money patterns cannot use percent, per-mille, or scientific notation.

<!-- rule: money.readonly.visible-value class=hard enforced=no tasks=client-runtime -->
Android and iOS edit money in place. The editor shows the currency code beside
an ungrouped amount using locale digits and its decimal separator. Committing
`123.45` for USD/2 stores `12345`. Grouping separators, currency symbols inside
the amount, excess fractional digits, and values outside signed i64 are
rejected without replacing the entered text. `ReadOnly` amounts remain visible
and cannot be edited. Nullability follows the ordinary schema contract.

<!-- rule: money.bounds-in-minor-units class=hard enforced=yes tasks=formatted-output title="Money `Default`, `Fixed`, `NumberMin`, `NumberMax` and a positive `NumberStep` are integer minor units; bounds and rules operate on minor units" -->
`Default`, `Fixed`, `NumberMin`, `NumberMax`, and `NumberStep` are integer minor
units; the step must be positive. Dynamic bounds and ValidationRules also
operate on minor units and preserve exact numeric values. <!-- rule: money.no-conversion class=hard enforced=no tasks=formatted-output title="Money introduces no conversion; keep unrelated currencies separate; the numeric semantic exclusions apply" --> Currency identity
does not introduce conversion: applications own exchange rates and must keep
unrelated currencies separate. Money follows the
[exclusions of numeric semantic attributes](schema.md#numeric-semantic-exclusions).

Charts retain each source integer for labels, tooltips, category identity, and
accessibility. Coordinates and synthesized scale ticks use the chart's normal
floating-point geometry; their presentation applies the declared currency and
scale. <!-- rule: money.shared-scales-match class=hard enforced=yes tasks=charts title="Shared chart scales and paired interval endpoints require matching Money metadata" --> Shared scales and paired interval endpoints require matching Money
metadata. A display pattern can intentionally hide lower digits without
changing the source amount.

## Physical units

<!-- /rule -->
[`PhysicalUnit`](schema.md#43-value-constraint-and-presentation-attributes) accepts a canonical CLDR core unit identifier on a Number:

```typescript
/**
 * Daily water depth
 * Sets the depth of water applied during one day.
 * @LibertasNumber(type=f64)
 * @LibertasPhysicalUnit("millimeter-per-day")
 */
rate: number;
```

```rust
/// Daily water depth
/// Sets the depth of water applied during one day.
#[libertas_physical_unit("millimeter-per-day")]
pub rate: f64,
```

Simple, powered, product, and generated compound identifiers are valid under
the [UTS #35 unit-identifier contract](https://unicode.org/reports/tr35/tr35-general.html#Unit_Identifiers).
All physical units, including compound units, are localizable. Keep the canonical
identifier in the schema; the viewing locale supplies unit presentation in
ordinary UI and charts. <!-- rule: unit.translate-text-not-identifiers class=judgment enforced=no tasks=formatted-output --> Translate field names, descriptions, chart titles, and
other authored text through the App's locale documents, not by changing unit
identifiers or appending translated unit strings to numeric values. <!-- /rule -->
Number-wheel editors show the localized unit in place of the generic “Level”
label, including when the field has no initial value. Prefer the full unit name
when it fits the available label width; otherwise use its localized short form.
Schema validity is independent of a particular client ICU catalog. A client may
omit unit decoration when it cannot format an otherwise valid identifier; the
numeric value and metadata remain intact.

<!-- rule: unit.no-conversion class=hard enforced=no tasks=formatted-output -->
The identifier states the provided unit and never performs conversion.
`percent` raw `50` means fifty percent; it does not rescale `0.5`. `Format` may
coexist and formats only the numeric portion. PhysicalUnit follows the
[exclusions of numeric semantic attributes](schema.md#numeric-semantic-exclusions).

AI descriptions carry the PhysicalUnit [value-representation hint](llm-document-transformation.md#value-representation-hints) with the exact identifier.

## Bitflags

<!-- rule: bitflags.single-bits class=hard enforced=yes tasks=formatted-output title="Each nonzero Bitflags member is one unique bit, one zero label is allowed, and values combine only declared bits" -->
Bitflags project a localized Enumeration onto an ordinary integer. Each
nonzero catalog member is a unique single bit and one zero label is allowed.
The field may combine declared bits but cannot contain undeclared bits. Clients
show localized selected labels and use a multiple-choice editor. The integer
wire value is unchanged.

<!-- rule: bitflags.zero-label.answer class=hard enforced=yes tasks=client-runtime,app-design,data-model title="A Bitflags field is never Nullable; a required one answers none with its zero label" -->
A required Bitflags field with no `Default` starts unanswered. When its catalog
declares a zero label (such as "None of these"), the multiple-choice editor
offers that label as an exclusive choice: choosing it clears every flag and
answers 0, and choosing any flag clears it. Ticking nothing leaves the field
unanswered. A Bitflags field is never wrapped in a Nullable: 0 already is the
empty selection, so a null adds no meaning; one "None of these" answers the
question.

<!-- /rule -->

AI descriptions carry the Bitflags [value-representation hint](llm-document-transformation.md#value-representation-hints) with the declared flag values and
available localized member descriptions. <!-- rule: bitflags.document-members class=judgment enforced=no tasks=formatted-output --> Author the meaning and consequences
of each flag in its member comments.

Bitflags follows the
[exclusions of numeric semantic attributes](schema.md#numeric-semantic-exclusions).

## String resources and formatted output

One package-owned catalog supplies notification templates, permission reasons,
data-name templates, `FormattedText`, and formatted chart labels for all
functions and types in the App. Task package context identifies the catalog;
encoded tuples carry no function/type owner or resource FQN.

### Expanding long displayed text

<!-- rule: text.expansion.preserve-size class=hard enforced=no tasks=client-runtime -->
Clients that shorten long displayed text behind **Show more** measure the full
text using its actual font, styling and available width. If it fits within the
combined height of the shortened preview, disclosure row and their spacing,
display the full text and omit the disclosure. Do not reduce the text size to
make it fit.

Otherwise, **Show more** reveals the complete text and **Show less** restores
the preview. Reevaluate the fit when the value, locale, width or text size
changes; a disclosure is unnecessary whenever the complete text fits. This
behavior concerns displayed text, independently of structural folding in the
Display Template. Clients that already display the complete wrapped text need
no disclosure.

### AI tool-call output

<!-- rule: ai.output.unresolved-formatted-error class=hard enforced=no tasks=client-runtime -->
The Hub renders each `FormattedText` tool result as a localized JSON string
using the emitting package's resources and locale. Templates and the resource
catalog remain internal to rendering; they are not included in the AI tool
description or result. If a value cannot be resolved or rendered, its result
is `{"error":"Unresolved FormattedText"}` rather than guessed text.

<!-- rule: ai.output.preserve-readonly-bytes class=hard enforced=no tasks=client-runtime -->
This output view does not change the canonical LMF1 tuple. Read-only request
fields retain their original Base64 Bytes representation in tool arguments;
rendered text cannot replace that payload for copying or resubmission.
Ordinary Bytes remain unchanged.

<!-- rule: ai.output.secret-withheld class=hard enforced=no tasks=client-runtime title="A Secret String in an AI tool result is replaced by the withheld placeholder object; null stays null" -->
A [`Secret`](schema.md#43-value-constraint-and-presentation-attributes) String
never reaches the model: in every tool result, including a returned Request
value, the Hub replaces each nonnull Secret value with the JSON object
`{"withheld":"Secret"}` and keeps null as null. The placeholder is not a
String, so a tool call that sends it back as a value is rejected and never
stores it. A tool parameter remains an ordinary String the model supplies
when the person gives it the value. <!-- /rule --> <!-- rule: text.document-purpose-not-templates class=judgment enforced=no tasks=formatted-output --> Declare `FormattedText` and document the
message's domain purpose; do not repeat templates or rendering instructions in
field comments for the AI.

<!-- rule: text.value-has-no-label class=hard enforced=no tasks=client-runtime title="A FormattedText value is drawn as its rendered text, with no label of its own" -->
A `FormattedText` value has no label of its own: the client draws the
rendered text as the row. The field's display name stays its accessible name
and its description source. A struct row that [promotes](#field-promotion) a
`FormattedText` child keeps its own label, with the text as its value.

### Static read-only placement and request values

<!-- rule: text.static-readonly class=hard enforced=yes tasks=formatted-output title="Every FormattedText node is statically ReadOnly: on the field, its type, or an enclosing read-only Table (a ReadOnly List does not count)" -->
Every `FormattedText` node MUST be statically `ReadOnly`. Declare it on the
field or its effective declared type, or place the node inside a statically
read-only Table. `ReadOnly` on a List protects membership and order only; it
does not make its elements or their fields read-only. Mark the applicable
element type or field when that is the only enclosing protection.

<!-- /rule -->
This [requirement](#static-read-only-placement-and-request-values) applies to every use, including Response and SubscriptionData
output and dormant or `TypesOnly` declarations. An output role, hidden field,
or runtime editing restriction does not replace the static declaration. Shared
Studio validation checks resolved types, inheritance, containers, and
materialized external dependencies. Language parsers extract the attributes.

<!-- rule: text.request-values class=hard enforced=no tasks=formatted-output -->
Request and SubscriptionRequest fields may carry these display-only values.
Their values come from `CopyFrom` or `CopyTo`, including a copy of an enclosing
record or request root; supplied `EditRequest` values; or `Default`/`Fixed`
initialization. A leaf does not need its own `CopyFrom` when one of those paths
supplies it. Apply the [shared copy rules](schema.md#shared-copyfrom-and-copyto-rules)
to copied values. The client never asks the user to edit a formatted tuple.
An unanswered local field may remain undefined; a required undefined field
blocks submission. Null is valid only where the destination permits it.

<!-- rule: text.not-in-function-arguments class=hard enforced=yes tasks=formatted-output title="FormattedText is forbidden in function and configuration arguments, even when read-only" -->
`FormattedText` remains forbidden in function/configuration arguments, even
when statically read-only. Reusable types must satisfy this placement rule and
that runtime-use restriction on every path.

### Persisting formatted record payloads

<!-- rule: text.persist-discouraged class=judgment enforced=no tasks=formatted-output -->
`FunctionDataSchema` record payloads may contain statically read-only
`FormattedText`. Persistence is supported, but generally discouraged for these
payload fields: `FormattedText` supplies runtime rendering, so prefer storing
structured domain values and constructing the formatted value at presentation
time. This is design guidance, not a schema rejection rule.

Formatted [DataName](message-format.md#data-and-database-names) labels remain
supported outside this payload-design recommendation. Records use the
[transactional Data API's declared binary keys](protocols.md#transactional-key-value-api)
for identity, with localized human-readable labels constructed independently.

### Declaring and encoding formatted values

```typescript
/// Room condition reports
/// Gives occupants clear summaries of current room conditions.
/// @LibertasStringResources(STATUS_STRINGS)

const STATUS_STRINGS = {
    READY: "Room condition reporting is ready.",
    ROOM_STATUS: "{0,LibertasDevice} reports a room temperature of {1,number,0.0}."
} as const;

/**
 * Room condition
 * Describes the latest reported condition in one room.
 */
export interface RoomState {
    /**
     * Room summary
     * Gives occupants a short overview of the room's latest condition.
     * @LibertasReadOnly
     * @LibertasFormattedText
     */
    summary: Uint8Array;

    /**
     * Current condition
     * Tells occupants what is happening in the room and includes the relevant measurement.
     * @LibertasReadOnly
     * @LibertasFormattedText
     */
    status: Uint8Array;
}
```

```rust
//! Room condition reports
//! Gives occupants clear summaries of current room conditions.
//! #[libertas_string_resources(STATUS_STRINGS)]

pub const STATUS_STRINGS: &[(&str, &str)] = &[
    ("READY", "Room condition reporting is ready."),
    ("ROOM_STATUS", "{0,LibertasDevice} reports a room temperature of {1,number,0.0}."),
];

/// Room condition
/// Describes the latest reported condition in one room.
#[derive(LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub struct RoomState {
    /// Room summary
    /// Gives occupants a short overview of the room's latest condition.
    #[libertas_read_only]
    #[libertas_formatted_text]
    pub summary: Vec<u8>,

    /// Current condition
    /// Tells occupants what is happening in the room and includes the relevant measurement.
    #[libertas_read_only]
    #[libertas_formatted_text]
    pub status: Vec<u8>,
}
```

<!-- rule: text.enum-arguments-typed class=judgment enforced=no tasks=formatted-output -->
For `ROOM_STATUS`, encode argument 0 as the `LibertasDevice` object and
argument 1 as a unit-qualified temperature number. Do not pass a translated
state as a literal or resource-text grammar fragment. A closed Enumeration
instead uses `{1,LibertasEnum,ActualEnumType}` and an integer argument encoded
with that Enumeration's semantic type.

<!-- rule: formatted.encoding.no-extra-base64 class=hard enforced=no tasks=formatted-output -->
FormattedText always uses the public SDK's canonical Bytes encoding of a
resource key plus ordered typed Libertas Message Arguments, including an empty
argument list. Do not Base64-wrap it in App code. A String host would only
encode the tuple and force the renderer to decode it again, providing no
benefit; Bytes also keeps a sharp type distinction from literal String data and
prevents authoring mistakes. A generic JSON byte codec may use Base64 as a
transport spelling without changing the field's Bytes type. Runtime payloads
never carry format templates.

TypeScript calls `libertasFormattedText(resourceName, arguments)` and Rust calls
`libertas_formatted_text(resource_name, arguments)`. These public encoders
return the byte value directly. App code ordinarily does not decode it; the
presentation client resolves and renders LMF1.

Use zero-based LMF1 positions so translations may reorder arguments:

| Value | Form |
| --- | --- |
| Untyped value | `{0}` |
| Libertas device | `{0,LibertasDevice}` |
| Libertas enum | `{0,LibertasEnum,StatusEnum}` |
| Styled number | `{0,number,0.00}` |
| Plural | `{0,plural,one {# item} other {# items}}` |

Every translation preserves argument positions, types, styles, selector keys,
plural offsets, braces, and apostrophe escapes. The encoded Libertas Message
Argument type takes precedence over a conflicting template type.

## Charts

This is a compact authoring summary. The normative vocabulary, validation
matrix, renderer behavior, and audited text-field dispositions are in
[Client-side charting](charts/2d.md). Language parsers own native
syntax and extraction; Studio's shared chart validator owns portable semantic
validation.

Chart metadata belongs on a Response or SubscriptionData payload. A primitive
chart is a List of row Tables; the List declares the mark, and direct scalar row
fields declare channels.

```typescript
/**
 * Temperature reading
 * Records the temperature observed at one time.
 */
export interface Sample {
    /**
     * Observation time
     * Identifies when the temperature was observed.
     * @LibertasChartChannel(x, tooltip)
     * @LibertasChartScale(id=time, kind=utc)
     */
    at: LibertasDateTime;

    /**
     * Observed temperature
     * Reports the temperature measured at the observation time.
     * @LibertasNumber(type=f64)
     * @LibertasPhysicalUnit("celsius")
     * @LibertasChartChannel(y, tooltip)
     * @LibertasChartScale(id=temp, kind=linear)
     */
    temperature: number;
}

/**
 * Temperature history
 * Lists temperature readings in observation order.
 * ----
 * Temperature reading
 * Records the temperature observed at one time.
 * @LibertasResponse
 * @LibertasSubscriptionData
 * @LibertasChart(line)
 */
export type History = Sample[];
```

```rust
/// Temperature reading
/// Records the temperature observed at one time.
#[derive(LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub struct Sample {
    /// Observation time
    /// Identifies when the temperature was observed.
    #[libertas_chart_channel(x, tooltip)]
    #[libertas_chart_scale(id = time, kind = utc)]
    pub at: LibertasDateTime,

    /// Observed temperature
    /// Reports the temperature measured at the observation time.
    #[libertas_physical_unit("celsius")]
    #[libertas_chart_channel(y, tooltip)]
    #[libertas_chart_scale(id = temp, kind = linear)]
    pub temperature: f64,
}

/// Temperature updates
/// Describes the temperature observations that can arrive over time.
#[derive(LibertasExport, LibertasAvroDecode, LibertasAvroEncode)]
pub enum ChartProtocol {
    /// Temperature history
    /// Lists temperature readings in observation order.
    /// ----
    /// Temperature reading
    /// Records the temperature observed at one time.
    #[libertas_response]
    #[libertas_subscription_data]
    #[libertas_chart(line)]
    History(Vec<Sample>),
}
```

Marks are `point`, `line`, `area`, `bar`, `rect`, `rule`, `arc`, `text`, and
`polygon`. Compositions are `layer`, `hconcat`, and `vconcat`; each direct
composition child is independently a chart.

The closed channel vocabulary is:

- positions: `x`, `x2`, `y`, `y2`, `xOffset`, `yOffset`, `theta`, `theta2`,
  `radius`, `radius2`;
- appearance: `color`, `fill`, `stroke`, `opacity`, `fillOpacity`,
  `strokeOpacity`, `size`, `shape`, `angle`, `strokeWidth`, `strokeDash`;
- text: `text`, `tooltip`;
- structure and identity: `detail`, `order`, `key`; and
- facets: `row`, `column`.

A field has at most one scale-bearing channel. Secondary endpoints require
their primary and cannot own a scale. `tooltip` and `detail` may appear on
multiple fields.

Scale types are `linear`, `log`, `symlog`, `sqrt`, `time`, `utc`, `ordinal`,
`band`, `point`, and `identity`. Use continuous numeric scales for physical
quantities. DateTime position infers time; DateOnly uses timezone-free calendar
day geometry. Categorical Enumeration and Libertas object references retain
their raw identity while clients resolve localized labels. Plain Bytes are not chart scalars.

Every value that can become chart text keeps its semantic presentation source.
Use an Enumeration for a closed App vocabulary, FormattedText Bytes for dynamic
App-authored wording, and `LibertasDevice`, `LibertasVirtualDevice`,
`LibertasUser`, `LibertasEndpoint`, or `LibertasLanDevice` when the value is a
system object. A visible plain String is only genuine literal domain/user data,
such as a proper name or free-form annotation. This applies to axes, facets,
legends, text marks, tooltips, guides, and accessibility in every chart family.
Key-only and guide-less identity appearance Strings are the non-textual
exceptions. Never flatten an Enumeration or copy an object display name into a
chart-only String.

FormattedText remains a byte blob end to end. Its tuple carries a package-local
resource key and typed arguments, not a type owner or resource FQN; the task
identifies the package's one `StringResources` catalog. Generic JSON may encode
the Bytes value as Base64 at that transport boundary, but the chart field is
never a String. Encoding it as a String only to decode it for rendering has no
benefit and would erase the sharp schema distinction from literal text.

Source scale properties are `id`, `kind`, `min`, `max`, `mid`, `zero`, and
`reverse`; the extracted and final language-neutral spelling of source `kind`
is `type`.
Fields sharing a nonempty scale ID must have compatible scalar type, channel,
Format, and PhysicalUnit. Different but convertible units are still different
scales unless the App emits one common unit.

<!-- rule: chart.appearance-in-template class=hard enforced=no tasks=charts -->
Chart appearance belongs to the [Display Template](#display-template). Studio
validates the complete chart metadata on the parser result, then compiles the
scale `type` of each `ChartScale` and `Chart3DScale` into a `scale` hint and
the `position` of each `ChartGuide` into a `guide` hint, and removes those
properties from the final schema; an attribute left with no property is
removed. Channels, scale IDs, domains, and every other chart property remain
schema. A template rule may also choose a chart `palette`. Clients read the
effective scale type, guide position, and palette only from the template, and
a template value is validated under the same chart matrices as the source.

<!-- rule: chart.explicit-guides class=judgment enforced=no tasks=charts -->
Use `@LibertasChartGuide(target=..., source=...)` or
`#[libertas_chart_guide(target = ..., source = ...)]` only for an explicit
guide. Targets are `x`, `y`, `xOffset`, `yOffset`, `theta`, `radius`, `color`,
`fill`, `stroke`, `opacity`, `fillOpacity`, `strokeOpacity`, `size`, `shape`,
`angle`, `strokeWidth`, `strokeDash`, `row`, and `column`. Sources are `scale`,
`value`, `span`, or `none`; `value` is limited to `x`, `y`, `theta`, or
`radius`, while `span` is limited to continuous Cartesian `x/x2` or `y/y2`.
The required properties are `target` and `source`. Optional properties are
`position`, `title`, `domain`, `ticks`, `labels`, and `grid`; positions are
`auto`, `top`, `right`, `bottom`, or `left`, and component switches are `auto`
or `none`. Prefer inferred scale guides unless the App needs an explicit
override.

### 3D charts

Use the parallel five-attribute namespace for 3D output:
`@LibertasChart3D`, `@LibertasChart3DChannel`, `@LibertasChart3DScale`,
`@LibertasChart3DGuide`, and `@LibertasChart3DView` in TypeScript, or
`#[libertas_chart3d(...)]`, `#[libertas_chart3d_channel(...)]`,
`#[libertas_chart3d_scale(...)]`, `#[libertas_chart3d_guide(...)]`, and
`#[libertas_chart3d_view(...)]` in Rust. Do not substitute the 2D attributes or
mix the two namespaces on one chart path.

Primitive marks are `point`, `line`, `bar`, `surface`, `wireframe`, and `stem`;
compositions are `layer`, `hconcat`, and `vconcat`. The channels are `x`, `x2`,
`y`, `y2`, `z`, `z2`, `u`, `v`, `color`, `fill`, `stroke`, `opacity`,
`fillOpacity`, `strokeOpacity`, `size`, `shape`, `strokeWidth`, `strokeDash`,
`tooltip`, `detail`, `order`, `key`, `row`, and `column`. Scale properties
mirror 2D scale properties; write `kind` in source and let extraction emit
schema `type`. A 3D scale also accepts `logBase > 1` for `kind=log`,
non-negative `growMin/growMax` fractions for automatic continuous x/y/z domain
growth, and inclusive `[0,1]` `padding` on a `bar` base axis.

Map a requested chart family to geometry instead of inventing a mark token:
scatter and point clouds use `point`; bubble charts add `size`; 3D lines,
waterfalls, and spectrum traces use `line`; columns use `bar`; height maps,
non-uniform grids, and sampled mathematical functions use `surface`; wire grids
use `wireframe`; contours layer `line` over `surface`; and impulses use `stem`.
Use `detail` for homogeneous multiple series and `layer` for heterogeneous
series.

Bars accept discrete or sparse continuous x/y bases with a continuous z value.
`x2`, `y2`, and `z2` may appear independently; omitted x/y extents and the z
baseline are inferred, while all six coordinates form an explicit cuboid. The
legacy any-orientation form remains valid when exactly two primary axes are
band scales and the sole continuous primary is the value axis. Surface and
wireframe `u,v` coordinates may be arbitrary finite topology values and may
contain holes.

View properties, in canonical order, are
`projection,azimuth,elevation,zoom,fov,orbit,aspectX,aspectY,aspectZ,targetX,targetY,targetZ,pan,zoomEnabled`.
Use `perspective|orthographic` for projection and booleans for orbit, pan, and
`zoomEnabled`. Set `zoomEnabled=false` to disable user zoom independently
without changing the authored initial `zoom`.
Azimuth, elevation, and target coordinates are finite; zoom and aspects are
greater than zero; and `0 < fov < 180`. An orthographic view may carry `fov`,
which is ignored while orthographic. Keep camera choices in one `Chart3DView`
on a physical chart scene. Guide properties are required `target,source` plus
optional `title,domain,ticks,labels,grid,tickCount,tickStep`; `detail` is a
valid target. Do not invent fixed schema count caps; ordinary payload and
runtime resource admission still applies. Follow
[Client-side 3D charting](charts/3d.md) for the exact geometry,
topology, composition, validation, interaction, fallback, and resource rules.

<!-- rule: chart3d.no-gesture-modes class=judgment enforced=no tasks=charts-3d -->
Do not invent a camera or gesture `mode`, encode touch mappings in response
data, or add sensitivity or control-label fields. `orbit`, `pan`, and
`zoomEnabled` only gate independent client capabilities. The preview is
gesture-neutral; **Explore 3D chart** opens full-screen Chart Explorer, whose
fixed client vocabulary maps one-finger drag to Rotate, two-finger translation
to Move, and pinch/spread to Zoom. Tap-only and accessibility actions expose
Rotate, Move, Zoom, and Reset separately, without a Rotate/Move selector.

Point and stem-tip shape values use
`circle,square,triangle,diamond,sphere,cube,pyramid,cylinder` in that automatic
order; bar shapes use `box,cylinder,pyramid,sphere`. Size zero intentionally
suppresses or degenerates geometry while retaining the source row.

<!-- rule: chart.structured-view-meaningful class=judgment enforced=no tasks=charts -->
Always keep the ordinary structured data view meaningful. Charts are an
additional presentation, not the only representation of returned data.

## Translation workflow

Studio owns structural synchronization. A translation changes localized
strings only. Preserve:

- `DocumentVersion`, structure, arrays, ordering, numbers, and native names;
- string-resource keys;
- schema identifiers; and
- every LMF1 argument position, type, style, selector, plural offset, brace,
  and apostrophe escape.

<!-- rule: translation.output.generated-structure class=hard enforced=no tasks=translation -->
Use Studio's locale chooser and translation export/import workflow. Do not
manually invent a locale entry or restructure a generated `.doc.json` file.

## Boolean answer controls

<!-- rule: boolean.answer-controls class=hard enforced=no tasks=field-presentation -->
Required Booleans without a default are explicit, initially unselected choices.
A non-nullable Boolean with an explicit Default permits a switch only when no
custom labels exist. In an editor, nullable Booleans use one inline radio group
for true, false, and explicit null. All three choices remain visible when null is
selected; choosing the null option stores actual null. Use localized
[TrueText, FalseText, and NullText document properties](documentation.md#standard-properties)
for their captions, with localized Yes, No, and None when omitted. Display labels
never change wire values. Every client arranges the choices by rungs 1 to 3
of the [inline-choice ladder](#inline-choice-ladder): beside the title, on one
line below it, or one per line. ReadOnly and Fixed keep the current selection
visible and disable editing.

<!-- rule: boolean.switch-on-row class=hard enforced=no tasks=client-runtime title="A Boolean switch or checkbox stays on its label's row; a long label wraps beside it" -->
A Boolean shown as a switch or checkbox keeps the control on its label's row; a
long label wraps beside the control and the row grows, and the control is never
pushed to a line of its own.

## Inline choices

`InlineChoices` shows an Enumeration's or Union's choices in its row as radio
controls, the way [Boolean answer controls](#boolean-answer-controls) render,
instead of a value that opens the choice selector. Author it as
`@LibertasInlineChoices` in TypeScript or `#[libertas_inline_choices]` in
Rust. It takes no argument and emits `{"Name":"InlineChoices","Value":"true"}`
into the parser result; Studio compiles it into the
[Display Template](#display-template) as the `inline` hint and removes it from
the final schema.

<!-- rule: inline.placement class=hard enforced=yes tasks=display-template title="`InlineChoices` goes on an Enumeration or Union field or type, read directly; rejected elsewhere and on a Bitflags catalog" -->
Place it on an Enumeration or Union field, or on an Enumeration or Union type
declaration. Each declaration is read directly, not through effective-attribute resolution:
a marker on either the field or its type enables inline choices, and a field
cannot switch off its type's marker. Studio rejects the marker on any other
effective type and on a Bitflags catalog, including a field typed by one.

- The radio controls offer the choices that remain after
  [ConstrainedBy](protocols.md#backend-constrained-choices) filtering, with
  their localized display names. Selecting one is the same edit as choosing it
  in the selector; an unanswered field shows no selected control.
- <!-- rule: inline.union-one-per-line class=judgment enforced=no tasks=field-presentation --> For a Union, the selected branch's fields render directly below its
  chosen radio control, before the next choice and indented to the choice's
  caption, only while the branch has exactly one visible field whose type is
  Number (a `DateOnly` date included), String, Boolean, `DateTime` or
  `TimeOnly`, where a Number or String carrying Bitflags or EnumSource counts as
  an Enumeration; a branch whose payload is itself such a scalar is that one
  field. Otherwise its fields render after the whole choice group: with no
  visible field, with two or more, or with one field of any other type, such
  as an Enumeration (Bitflags and EnumSource included), Union, struct, List, or Bytes (FormattedText included).
  A field is visible when its row is shown, under the same rules that show or
  withhold that row (Hidden, and template withholding such as
  [`when_single_choice`](#single-choice-rows)), and the count is taken again
  whenever that visibility changes. Promotion of the Union or of its owner
  never changes this placement. Selecting another choice replaces the
  applicable fields. A Union with any declared branch that carries data,
  every branch but a `Nil` one, stands one per line, as the
  [ladder](#inline-choice-ladder) below states. Authors who want the choices
  on one line with the selected content below use the
  [`tabs` layout](#display-template-catalog) instead.
- The singleton allowed set and answered-Union rules of
  [backend-constrained choices](protocols.md#backend-constrained-choices) are
  unchanged.
- Visibility, read-only, promotion, and folding rules still apply;
  the marker does not make a read-only value editable.

<a id="inline-choice-ladder"></a>

Every client arranges the permitted choices by the first rung of this ladder
that fits, measuring the localized captions and the title in the current font
and text size:

1. **Beside.** The title's readable minimum, the 16 dp gap, and one line of
   all choices with a 16 dp gap between each pair fit the row width: title and
   choices share the line. The readable minimum is the title's full width, or,
   if that is wider, the larger of its longest word and 40% of the row width.
2. **Below, one line.** That line of choices fits the row width: the choices
   stand on one line beneath the title.
3. <!-- rule: choices.inline.no-line-wrap class=hard enforced=no tasks=client-runtime --> **One per line.** Each choice gets its own line beneath the title, and a
   long caption wraps within its line. Choices never wrap onto a second
   horizontal line. <!-- rule: choices.inline.wrapped-caption-layout class=hard enforced=no tasks=client-runtime --> A wrapped caption's lines use a compact label line
   height of about 1.3 times its text size, never paragraph spacing; its
   choice's radio aligns with the caption's first line, and the wrapped
   lines stay indented to the caption's start. While any choice of the group
   wraps, every pair of adjacent choices is separated by at least 12 dp
   between their caption blocks, so each choice reads as one unit (a field
   placed below its choice stays inside that choice's unit). Each choice's tap area is
   at least the platform minimum (48 dp, 44 pt) and absorbs the gap: its row is
   the larger of that minimum and its caption plus 12 dp, so a long caption gets
   exactly 12 dp and a short one grows to the minimum. Tap areas never overlap;
   a client without a touch minimum keeps exactly 12 dp. A
   group whose choices all fit on one line keeps its compact one-line rows.
4. <!-- rule: inline.ladder.selector-above-twelve class=hard enforced=no tasks=client-runtime --> **Selector.** More than 12 permitted choices use the ordinary choice
   selector with its filter, at any width.

<!-- rule: inline.ladder.count-and-unions class=hard enforced=no tasks=client-runtime -->
The count includes the null answer of a nullable Enumeration and follows the
current ConstrainedBy set; the rung is chosen again when the width, text size,
or permitted set changes. A Union with any declared branch that carries data
skips the beside and one-line rungs: its choices stand one per line at every
width, so a single simple field has a place directly below its choice and the
choices never move when one is chosen. Only the declared branches decide this,
never the current selection or permitted set. The selected branch's fields
render below the chosen choice or after the whole group, as
[inline choices](#inline-choices) states; changing rung keeps the selected
branch and its fields. A Union whose branches are all `Nil` uses every rung, like an
Enumeration. Every rung is one radio group named by the title, in the
permitted order, with arrow keys moving within it (mirrored for right-to-left).
Arrow keys move only between choices; a field placed below its choice follows
that choice in reading and focus order, and fields placed after the group
follow the whole group. <!-- rule: inline.false-and-forward-tolerance class=hard enforced=no tasks=field-presentation --> A template rule with
`inline: false` still turns inline choices off. The marker changes presentation
only, never data, encoding, or paths. A client without inline support keeps
the choice selector.

## Link rendering

A linked row may be highlighted through its [enum/protocol association](schema.md#enumeration-protocol-associations); the [pane contract](#master-detail-panes) owns that use.

A [link](protocols.md#links) adds an affordance to its row. The upcoming
`stacked=true` form keeps this rendering and displays its reply above the live
sending surface with the mandatory system Close specified by the Links owner. <!-- rule: link.row-rendering class=hard enforced=no tasks=client-runtime -->
A link never removes the row's label or value and never replaces the value with a button. Whether
the value is editable decides how the link appears:

- <!-- rule: link.not-editable-value class=hard enforced=no tasks=client-runtime --> **Value not editable:** every Response or SubscriptionData row, and a
  ReadOnly row in a request editor. The label stays plain, the value renders as
  link text in link styling, and the whole row is the tap target. <!-- rule: link.container-header-text class=hard enforced=no tasks=client-runtime --> On a struct,
  Union branch, or List member, the header text is the link text, and an
  expanded child row taps the nearest enclosing link unless the child row
  declares its own link, which wins. <!-- rule: link.formatted-text-block class=hard enforced=no tasks=field-presentation --> On a FormattedText field the whole block
  is the tap target; links inside FormattedText markup are not supported.
- <!-- rule: link.editable-value class=hard enforced=no tasks=client-runtime --> **Value editable:** the label, value, and editing are unchanged. Tapping the
  value still edits it, and direct controls such as Boolean answers,
  [inline choices](#inline-choices), and a Bitflags editor still work. The
  link is a trailing link button beside the value, named after the target
  Request's localized name. This is the only case that renders a button.

<!-- rule: links.transition.single-owner class=hard enforced=no tasks=client-runtime -->
A primitive with no label of its own, such as an item of a List of scalars,
shows its value as the link text. A link retains transition ownership while
its composer is open, so repeated taps never open or send twice. Editing the linked request shows no waiting
spinner. Once it is submitted, the owning panel shows shared blocking progress
until the reply or terminal failure, under the
[waiting presentation contract](protocols.md#waiting-presentation); rows do not
show a separate link spinner. A link adds no chevron
or other glyph and never edits its field. <!-- rule: link.hidden-rows-rejected class=hard enforced=yes tasks=field-presentation title="Hidden rows have no link; Studio rejects that placement" --> Hidden rows have no link, since
Studio rejects that placement. Promotion, flattening, folding, and
read-only rules are unchanged by a link.

## Folding structs and unions

<!-- rule: fold.placement class=hard enforced=yes tasks=field-presentation title="`Foldable` (optional `initially_open` true, false or nondefault) is permitted only on an effective Table, Union or List" -->
Structs (`Table`), unions, and lists remain expanded unless `Foldable` is declared
on their type or field. Author it as `@LibertasFoldable` in TypeScript or
`#[libertas_foldable]` in Rust, with an optional `initially_open` that is
true (the default), false, or `"nondefault"`:
`@LibertasFoldable(initially_open=false)`,
`#[libertas_foldable(initially_open = false)]` or
`#[libertas_foldable(initially_open = "nondefault")]`. Studio permits this marker
only on an effective Table, Union, or List. It changes presentation, not encoded data.

In the parser result the attribute's string value carries the option:
`{"Name":"Foldable","Value":"true"}` starts expanded, `"false"` starts
collapsed, and `"nondefault"` starts collapsed unless the value differs from
its default (below). <!-- rule: fold.presence-enables class=hard enforced=no tasks=field-presentation --> Presence enables folding; `false` does **not** disable it. Studio
compiles it into the [Display Template](#display-template) as the `fold` hint
and removes it from the final schema; clients read folding only from the
template, where a rule may also change it per device.

<!-- rule: fold.disclosure-control class=hard enforced=no tasks=client-runtime -->
An unmarked struct, union, or list shows no disclosure control and cannot be
collapsed. A marked value shows one disclosure chevron at the leading edge of
its title line, pointing toward the text when collapsed and down when expanded
(mirrored for right-to-left text), and exposes its expanded state to
assistive technology. The title line toggles the fold, except on a row that is
also a [link](protocols.md#links), where only the chevron toggles and the rest
of the row follows the link. A marked value starts expanded, or collapsed
when `initially_open` is false, and permits manual collapse and expansion; a
collapsed value shows only its row, its title and promoted value
included. The initial state applies to rows that exist when a page is
displayed. <!-- rule: fold.created-values-open class=hard enforced=no tasks=client-runtime --> A value the user creates while the page is open, such as a new
List member or a materialized nullable, always opens, whatever its marker
says, so its fields are visible to fill in. <!-- rule: fold.nondefault class=hard enforced=no tasks=field-presentation --> With `initially_open` `"nondefault"`, the value starts expanded
when anything under it differs from its default and collapsed otherwise, so
rarely used fields stay out of the way until they hold something. Its default
is the value the App's default document gives that node; for a List member,
or a node the default document does not reach, it is the default value of its
type. Values are compared decoded, at every depth, including a selected Union
variant and a List's member count, never by their displayed text, so the result
is independent of locale. Values the platform generates rather than the person,
such as a virtual device identity, are left out of the comparison, and working
out the default registers nothing (no identity, name or icon). Like every initial state, it is decided when the
page is displayed and does not close or open the value while the person edits. <!-- rule: fold.per-value-permission class=hard enforced=no tasks=field-presentation -->
Each nested value owns its folding permission;
a marker does not grant permission to its descendants. Lists follow the same explicit-permission rule. Changing a union variant reveals its new fields. Union selectors
remain editable independently of folding.

<!-- rule: fold.promoted-union-row class=hard enforced=no tasks=field-presentation -->
For a promoted union, the shared owning row may fold when either
the owning Table or the promoted union has Foldable on its type or field.
Promotion and [`flatten`](#flatten) remain independent of folding.

<!-- rule: fold.collapsed-summary class=hard enforced=no tasks=client-runtime -->
The collapsed task editor value is a short presentation summary, never a
recursive JSON-like rendering of its children. A Table with a promoted value
field shows that field's normal formatted value. A Union shows its selected
variant name, including when the variant has a scalar payload. A List shows its
visible item count (for example, “3 items” or “0 items” when empty); a List
whose members are all hidden shows no count. An ordinary Table with no promoted
value shows no value text beside its title and fold control. Null remains
distinct from an empty List or a Table without a summary.

When [`flatten`](#flatten) with `{ "name_row": false }` removes a struct's
own row, the struct still assigns a dropdown owner, in this order: the row of its promoted child, which is
the struct's surviving value row, else the row of its first visible child.
When the struct is Foldable, that row carries its disclosure chevron and
collapses the struct's remaining rows beneath it; its own value stays editable
as usual. An unmarked struct gives that row no chevron.
Hidden fields produce no row and are skipped; a struct that produces no rows
folds nothing. A List member
whose struct row is suppressed keeps its position and membership controls on
the row that owns its dropdown.

## Unanswered fields and null

<!-- rule: fields.unanswered.semantic-placeholders class=hard enforced=no tasks=client-runtime -->
Required fields without an encodable value use subdued italic placeholders:
“Select…” for choices, “Enter…” for typed input, “Choose date…” or “Choose time…”
for temporal input, and “Set up…” for a non-nullable container needing
initialization.
Readonly fields without a value show “Not set.” Boolean choices retain their
unselected Yes/No controls. These labels are localized, with English fallbacks.

<!-- rule: nullable.presentation.explicit-answer-choice class=hard enforced=no tasks=client-runtime -->
A displayed nullable null is shown inside an outlined capsule using the field
document's `NullText`, or localized “None” when omitted. An editable Nullable
is the [Nullable choice](#nullable-choices) below, never a capsule with
unlabelled set and clear controls. <!-- rule: nullable.empty-string-is-value class=hard enforced=no tasks=field-presentation --> An empty String is a present value,
distinct from null; incomplete native input remains invalid.

### Nullable choices

**A Nullable (optional) value is a two-state Union:** one null state and one
state holding a value of its type, each with default label text unless the
field document names it (`[NullText]`, default “None”; `[SetText]`, default
“Specify”).

An editable Nullable asks one question with two answers: the **null answer**,
labelled by the field document's [`NullText`](documentation.md#standard-properties),
and the **value answer**, labelled by `SetText`, which opens the field's
ordinary editor. `NullText` is required on every editable Nullable, and
`SetText` wherever this control draws a value answer: on a Number, String,
Bytes, date or time, Table or List
([Nullable choices](app-contract.md#nullable-choices)). A Boolean's and a
picker's value answers are their own members, so they never show `SetText`. A
Nullable is editable when a Request variant or a function's configuration
reaches it without passing through a ReadOnly, Hidden or Fixed node; a type
owned by another package is checked by that package. A
client that meets a schema without a label falls back to localized “None” and
“Specify”. Choosing the
null answer discards the value, as switching Union branches does.

Every client draws the question with exactly this control, so the same field
looks and behaves the same on Studio, Android, and iOS:

| Value type | Control |
| --- | --- |
| Boolean | Three radio choices on the field's row, in this order: `TrueText` (Yes), `FalseText` (No), `NullText`. |
| Enumeration, EnumSource, User, Endpoint | The type's single-select picker; its first entry is `NullText`, followed by the members in their declared order. |
| Number, String, Bytes, DateOnly, TimeOnly, DateTime | Two radio choices on the field's row, `NullText` first and `SetText` second; once `SetText` is chosen, the value editor appears directly below them, full width. |
| Table, List | Two radio choices on the field's header row, `NullText` first and `SetText` second; once `SetText` is chosen, the Table's fields or the List's members appear below the header. |
| Union | The Union's own branch chooser, with `NullText` as its first choice before the branches. |

<!-- rule: nullable.blank-value.unanswered class=hard enforced=no tasks=client-runtime,field-presentation -->
The null answer always comes first. While unanswered, no choice is selected and
a picker shows “Select…”. A value answer chosen with its value still blank stays
unanswered; null is never inferred from a blank editor. Read-only display keeps
the caption described above.

<!-- rule: nullable.initialization class=hard enforced=no tasks=field-presentation -->
Initialization follows the Union
([type initialization](schema.md#shared-copyfrom-and-copyto-rules)): with no
Default the question starts with neither answer selected, shows “Select…”, and
blocks submission until the person chooses; the reserved Default
[`$null`](schema.md#reserved-default-values) starts it on the null answer; any
other Default starts it on the value answer with that value. The stored value,
its encoding, CopyFrom, CopyTo, validation rules, AI Import, and calculations
still see `null` or the value: the unanswered state exists only in the editor
and is never encoded.

## Revealed rows

When a person's own choice adds rows to an editor (choosing a Nullable's value
answer, a Union branch, an Inline choice, or any edit that reveals fields
below), the client brings the first new row into view if it would otherwise
start below the visible area. Every client follows the same rule:

- <!-- rule: editor.reveal.only-local-control-scroll class=hard enforced=no tasks=client-runtime --> Scroll only for an edit the person made on that control: a tap, click, or
  keypress. A calculation reply, a server-supplied value, a restored draft, a
  rule, or another row's change never scrolls the editor.
- Scroll only when the first new row starts below the visible area; when it is
  already visible, nothing moves.
- <!-- rule: editor.reveal.keep-control-visible class=hard enforced=no tasks=client-runtime --> Scroll the least distance that shows the new rows, bringing as many of them
  into view as fit while the changed control stays visible. The changed control
  never leaves the visible area, and the editor never scrolls up for revealed
  rows.
- <!-- rule: editor.reveal.preserve-focus class=hard enforced=no tasks=client-runtime --> Follow the platform's reduced-motion setting: move without animation when it
  is on. Focus stays on the changed control; scrolling never moves it.

<!-- rule: editor.hide.no-scroll class=hard enforced=no tasks=client-runtime -->
Rows that disappear, such as a Table's fields after choosing a Nullable's null
answer, never trigger scrolling.

## Single-choice rows

<!-- rule: single-choice.member class=hard enforced=no tasks=display-template -->
`when_single_choice` is a Display Template member with no source attribute.
`"hide"` withholds the row of a choice field while its allowed set leaves only
one answer, and that answer needs no input; `"show"`, the default, draws the
row, so a higher-ranked rule can undo a lower one.

```json
{ "select": "^<Filing>.status", "when_single_choice": "hide" }
```

<!-- rule: single-choice.placement class=hard enforced=yes tasks=display-template title="`when_single_choice` selects a struct field carrying `ConstrainedBy` or a scalar `EnumSource` host, value `show`, `hide` or `payload` (Union only), never in a data condition or switch" -->
The rule selects a field of a struct that carries
[ConstrainedBy](protocols.md#backend-constrained-choices) or a scalar
[EnumSource](#common-field-metadata) host, by a relaxed `^<Type>.field` or a strict
path, under the placement rule of
[`promote`](#promotion-flattening-and-union-selectors). Studio rejects it on
any other node, an EnumSource [List host](schema.md#enumsource-list-hosts)
included, with any other value (and `"payload"` on a field that is not a
Union), and in a data condition or switch case;
`when` may change it per device.

<!-- rule: single-choice.withhold-when class=hard enforced=no tasks=client-runtime -->
The client does not draw the row while all of these hold:

- <!-- rule: single-choice.condition.one-answer class=hard enforced=no tasks=client-runtime --> its resolved allowed set, evaluated live as the chooser evaluates it, leaves
  exactly one answer: for a field that is not Nullable, one permitted declared
  choice for ConstrainedBy or one selectable List member for EnumSource; for a
  Nullable ConstrainedBy field, an empty set, whose only answer is null (a
  nonempty set always leaves null as a second answer);
- <!-- rule: single-choice.condition.no-payload class=hard enforced=no tasks=client-runtime --> that answer carries no payload: null, an Enumeration value, a Union variant
  without a payload, or the EnumSource member;
- <!-- rule: single-choice.condition.field-holds-it class=hard enforced=no tasks=client-runtime --> the field holds that answer, which the
  [singleton rule](protocols.md#backend-constrained-choices) writes when the
  field is unanswered.

<!-- rule: single-choice.row-kept class=hard enforced=no tasks=client-runtime -->
Otherwise the row is drawn. A sole Union variant with a payload always keeps
its row, so the payload stays reachable. An empty set on a field that is not
Nullable, a prefill outside the set, or an unavailable source keeps the row and
its field error visible. A
calculated set that widens brings the row back, and one that shrinks to a
single answer withholds it again. Read-only displays follow the same rule.

<!-- rule: single-choice.withholding-changes-nothing class=hard enforced=no tasks=client-runtime -->
Withholding a row changes nothing else. The field keeps its value and is not
Hidden; validation rules, `CopyFrom`, `CopyTo`, calculation requests,
encoding, AI Import, and AI Search read and write it exactly as if its row
were shown. Like a Hidden field, a withheld field is not promoted into its
parent row and leaves its place in a layout; a `label`, `title`, or table
column naming it still reads its value. The sole-member answer is the
ConstrainedBy and EnumSource data rule, not this member's, so a client that
ignores the member still stores the same value and simply draws the row.

<!-- rule: single-choice.payload class=hard enforced=no tasks=display-template,client-runtime title="`when_single_choice: payload` draws a sole payload-bearing Union variant as its payload under the field's own label, with no selector, variant name or inner label" -->
`"payload"` applies only to a Union field. While the field's allowed set leaves
exactly one answer, counted as [the single-answer rule](#single-choice-rows) counts
it (so for a Nullable field, null is a second answer and the field draws as under
`"show"`), that answer is a variant carrying a payload, and the field holds it, the
row draws **only that payload under the field's own label**: no variant selector,
no variant name, and no label of its own for a payload that is one scalar or one
Union (the field's label names it). A payload Table draws its fields in place
under the field's label, as a flattened Table does. Otherwise, including when the
set leaves several variants or a variant without a payload, the field draws as
under `"show"`. Like `"hide"`, it changes only drawing: the value, validation,
`CopyFrom`, `CopyTo`, calculations, encoding, AI Import and AI Search are
unaffected, and a client that does not support it draws the row as `"show"`.
