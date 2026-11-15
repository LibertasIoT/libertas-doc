# Writing a Display Template

Load this card when a task creates or changes `display.json`: a layout, a
master-detail arrangement, labels and titles, page controls, promotion or
flattening, a device or data condition, a style token or a behavior hint. Its
rules are generated from the
[Display Template contract](../reference/presentation.md#display-template),
which owns them, with the [selector grammar](../reference/schema.md#display-template-selectors)
in the schema contract. Source presentation attributes that compile into the
template (`Foldable`, `InlineChoices`, `SuppressLabel`, `UIIncludeDescription`,
`Format`) are on the [field presentation card](field_presentation.md); chart
metadata is on the [chart](charts.md) and [3D chart](charts_3d.md) cards. The
[annotation inventory](schema_annotations.md) gives each attribute's Rust and
TypeScript spelling.

## Order of work

1. Decide what belongs in the schema and what in the template
   (`tpl.boundary.*`, `tpl.presentation-only`, `tpl.withheld-row-unchanged`,
   `tpl.ai-descriptions-untouched`).
2. Write the rules in `display.json` at the project root (`tpl.file.*`,
   `tpl.compiled.*`), and know how they resolve (`tpl.cascade.*`).
3. Select nodes and conditions: `when` and `mode` (`tpl.when.*`, `tpl.mode.*`),
   pseudo-states, data conditions and `switch` (`tpl.condition.*`,
   `tpl.switch.*`).
4. Choose each arrangement from the closed catalog (`tpl.catalog.*`,
   `layout.*`), and for a hub with editors beside it the pane rules
   (`master-detail.*`).
5. Name and control the page: labels and titles (`label.*`), button captions
   (`button.*`), named controls (`control.*`, `hub-action.*`) and the heading
   (`heading.*`).
6. Shape rows: promotion and flattening (`promote.*`, `flatten.*`,
   `field.promote-flatten-not-inherited`), hints and style (`hint.*`,
   `style.*`), single-choice rows (`single-choice.*`).
7. Compile with the Studio MCP `compile_schema` tool, fix every reported rule
   and warning (`tpl.validation.*`, `tpl.lint.*`), then call `preview_display`
   so the user sees the screen on a device.

## Choose an arrangement

Decide arrangement with this procedure; each step names the rules it applies:

- A List of small records: `table` (`layout.table`, `layout.table.*`),
  optionally `card` members with `form: phone`.
- A List whose items need previews and a complete item editor: `summary`
  (`layout.summary`, `layout.summary.*`).
- An Enumeration or Union with a few record-or-unit choices: `inline`
  (`hint.inline`; the ladder and the twelve-choice limit are on the
  [field presentation card](field_presentation.md)).
- A hub Response with a List whose member type has a `Link`: `master-detail`
  with `right_pane` and `refresh` (`master-detail.*`); it stacks below the
  pane threshold.
- Many fields on a `view` page: `sections` with `tabs` or `rows`, a `split`,
  or a fold that starts closed (`layout.sections`, `layout.split`,
  `tpl.lint.view-screens`, `tpl.lint.fields-per-screen`). An `edit` page stays
  one scrolling form (`tpl.mode.no-tabs-in-edit`).
- Numbers compared down a column: `align: end` (`style.tokens`).
- A status value: `:condition(...)` or `switch` with `color` and `icon`
  (`tpl.condition.status-badge`).

## Display presets

**Start from a public preset.** The following files are published beside this
documentation; each documents its own `^<Type>`-style placeholder names at the
top under `_placeholders` for you to rename. Copy its `DisplayVersion` and
`rules` into the project's source `display.json`, merge with existing rules
where needed, and remove the explanatory `_comment` and `_placeholders`
members. Start from [form.json](../examples/display/form.json) for a request
editor: a two-column `form-grid` that becomes one column on a phone, a folded
optional group, and an inline choice. Start from
[hub.json](../examples/display/hub.json) for a Response with a linked List:
`master-detail` with its `right_pane` editors and `refresh` loader, which
stacks on a phone, and status badges via `:condition(...)`. Start from
[dashboard.json](../examples/display/dashboard.json) for a Response of
summaries, a chart field, and a long line List: `split` with a `card` region,
`card` on phone, and `table` for the line List where its columns fit.

<!-- generated:codebot-rules task=display-template -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Categorized import

Owner: [llm-document-transformation.md#categorized-import](../reference/llm-document-transformation.md#categorized-import)

- `doctrans.categorized.review-summary`: **Ready summary.** When the compiled [Display Template](../reference/presentation.md#summary-array-items) gives the kind's category List a `summary` layout, the row summary is that rule's `paths` read from each imported item, in path order, joined by " · " (an empty value contributes nothing). Otherwise the summary is the first three answered scalars of the import in declaration order, skipping Booleans, null and empty values, and a field named `year`, `tax_year` or `calendar_year`, which is the paper's period and does not identify it. A document that imports several forms shows each form's summary, separated by "; ". The summary is at most 160 characters.

### AI tool-call descriptions

Owner: [presentation.md#ai-tool-call-descriptions](../reference/presentation.md#ai-tool-call-descriptions)

- `tpl.ai-descriptions-untouched`: Display Template presentation never reaches AI descriptions: no template rule, `format` pattern, `label`, or `title` changes what a tool or AI Import description says.

### Display Template

Owner: [presentation.md#display-template](../reference/presentation.md#display-template)

- `tpl.not-schema`: **Not part of the schema.** The template is absent from the schema JSON, its revision digest, and the App version. A template change is a document-only revision: it never rebuilds the App or allocates an App version.
- `tpl.rows-default`: **Opt-in arrangement.** `rows`, one row per child, is the default layout. Every other layout applies only where a rule asks for it. A `table` or table-arranged `summary` whose columns do not fit, and a `master-detail` below the pane threshold, fall back under the [width-based layout](../reference/presentation.md#width-based-layout).
- `tpl.presentation-only`: **Presentation only, a layer apart from business logic.** Template rules are not on the same level as the App's business logic and never reach it. No rule, condition, or device state changes what a field stores, its validation rules, `CopyFrom` or `CopyTo`, calculation requests, encoding, or the [AI Import](../reference/llm-document-transformation.md) and [AI Search](../reference/protocols.md#ai-navigation-and-ai-search) contracts; none makes a field Hidden or read-only or changes a user's submitted payload. No rule reaches an [AI description](../reference/presentation.md#ai-tool-call-descriptions).
- `tpl.withheld-row-unchanged`: A rule that withholds a row, such as [`when_single_choice`](../reference/presentation.md#single-choice-rows), leaves the value and every rule that reads it exactly as if the row were shown. The [pane arrangement](../reference/presentation.md#master-detail-panes) may load or refresh a leading page through a declared pure navigation Request. A condition may change style, layout, fold state, or existing page-control captions only. Hidden, ReadOnly, and every other schema rule apply before and regardless of the template.

### Template boundary

Owner: [presentation.md#template-boundary](../reference/presentation.md#template-boundary)

- `tpl.boundary.tests` (judgment): An attribute belongs in the schema when changing it would change what the App accepts, sends, computes, protects, or means. It belongs in the template when changing it would only change how the same value is drawn on one device. Apply three tests in order:
- `tpl.boundary.wire` (judgment): **Does the server or the wire depend on it?** Encoding, defaults, nullability, ranges, uniqueness, validation, copies, calculations, action sets, access, and identity are schema.
- `tpl.boundary.meaning` (judgment): **Does it state what the value is or means?** Money and currency, units, dates, formatted text, labels, help text, and which fields a chart encodes are schema; meaning is content, localized with the strings.
- `tpl.boundary.device` (judgment): **Would a different choice be right on another screen size?** Promotion, flattening, folding, inline choices versus a chooser, an inline description, density, arrangement, and chart appearance are template, with a source attribute, where one exists, as the default rule.
- `template.boundary.source-attribute-roles`:

  | Schema only | Template (a source attribute, where one exists, is the default rule) |
  | --- | --- |
  | AccessPrivilege, CacheIdentity | `Format` display pattern and precision, never the value or currency |

- `tpl.boundary.unordered-excludeui`: Unordered and ExcludeUI look presentational, but the wire and the protocol depend on them, so they stay in the schema. `Format` looks semantic, but `Money` already carries the currency and the precision that matter to the value, so the display pattern moves. [`label` and `title`](../reference/presentation.md#labels-and-titles) look like schema labels, but the node's display name stays its meaning in the schema; they change only what a person sees, so they are template.

### Display Template file

Owner: [presentation.md#display-template-file](../reference/presentation.md#display-template-file)

- `tpl.file.source-of-truth`: The source of truth is `display.json` at the project root, beside `Cargo.toml` or `package.json`, under version control. There is one file per project; device variance is expressed inside it as conditions, never as a second template for a locale or platform.
- `tpl.file.missing-valid`: A missing file is valid and compiles to a template holding only the rules derived from source attributes. Studio's editor writes this file in place through the same validated writer as a hand edit, never a second copy.
- `tpl.file.lock`:

  | Member | Required | Meaning |
  | --- | --- | --- |
  | `lock` | No | `true` or, on a Union, `"variant"`: the person cannot change the value, while every other writer still can ([presentation locks](../reference/presentation.md#presentation-locks)). |

### Compiled template

Owner: [presentation.md#compiled-template](../reference/presentation.md#compiled-template)

- `tpl.compiled.field-over-type`: Type-derived defaults precede field-derived ones, so a field's own attribute wins where both set the same hint.
- `tpl.compiled.direct-lookup`: Each derived rule reproduces its source attribute's meaning exactly, including direct declaration lookup: a Foldable on a field yields a field `fold` rule and one on a type yields a type `fold` rule, and neither is inherited through effective-attribute resolution.
- `tpl.compiled.never-edited`: The compiled template is derived and never edited by hand. Its identity is the SHA-256 digest of its canonical JSON, computed with the same canonicalization as the [schema revision digest](../reference/schema.md#113-revision-digest). `compile_schema` with `includeArtifacts` returns it beside each package's default document.

### Display Template cascade

Owner: [presentation.md#display-template-cascade](../reference/presentation.md#display-template-cascade)

- `tpl.cascade.member-resolution`: Every presentation member resolves independently: the highest-ranked applicable rule that sets that member wins, and a layout resolves together with its layout-specific members.
- `tpl.cascade.button-slot-independent`: Each `button_labels` slot resolves independently; setting one slot leaves the others' cascade intact.
- `tpl.cascade.later-wins`: Among rules of equal rank, the later rule in compiled order wins.
- `tpl.cascade.conditions-gate`: Data conditions, pseudo-states, and switches gate a rule; they never change its rank.
- `tpl.cascade.inheritance`: The style tokens `density` and `emphasis` inherit down the value tree unless a descendant resolves its own; no other member inherits, and a layout never does.
- `tpl.cascade.dependency-templates`: A screen uses the full template of the package that owns its displayed root: the Endpoint protocol Union's package for a protocol screen, and the function's package for a configuration editor or stored data. Nodes declared by another package additionally receive that package's compiled template, its `display.json` rules included, at rank 2, below every rule of the screen's own package. A dependency's rule applies where its selector, resolved against that package's schema, matches a node that package declares. A type therefore keeps its authored presentation wherever it is shown, the using package can still change it, and a TypesOnly package may ship a `display.json` for its types. Every package template a screen needs arrives with its schema.

### Display Template conditions

Owner: [presentation.md#display-template-conditions](../reference/presentation.md#display-template-conditions)

- `tpl.when.size-classes`: Conditions use size classes and a form factor, never pixels. A 2160 px phone in landscape is about 800 dp wide (`width: medium`, `height: compact`, `form: phone`); a 32-inch monitor is `width: extra-large`, `height: expanded`, `form: desktop`. A rule for 840 dp and wider lists `["expanded", "large", "extra-large"]`.
- `tpl.when.widen-by-form-or-height` (judgment): A rule that widens a layout reads `form`, or `width` together with `height`, never `width` alone.
- `tpl.when.no-condition-for-fit` (judgment): Pane count, split lines, `form-grid` columns, and table fit need no condition: they follow the [width-based layout](../reference/presentation.md#width-based-layout) rules at every size.
- `tpl.when.container-semantics`: Conditions have container semantics: `width`, `height`, and `orientation` are evaluated against the nearest enclosing pane, not the window, while `form`, `pointer`, and `theme` describe the device. A list rendered in the left pane of a `master-detail` layout on a tablet sees that pane's width class.

### Editing and viewing

Owner: [presentation.md#editing-and-viewing](../reference/presentation.md#editing-and-viewing)

- `tpl.mode.first-and-reopen-both-edit`: Composing a Request for the first time and reopening it to review or change it are both `edit`; a rule that must tell them apart reads the [interaction reference context](../reference/protocols.md#interaction-reference-context) with `:condition`.
- `tpl.mode.unset-applies-both`: A rule without `mode` applies in both modes.
- `tpl.condition.status-badge` (judgment): `^<TopicRow>:condition([status] is NeedsAttention)` with `color` and `icon` declares a status badge; no schema attribute is needed.
- `tpl.condition.recheck-combined` (judgment): **Re-check a combined condition when it or its data changes.** The client evaluates a condition on the value being displayed or edited, including rows added or removed before any save, so nothing on the server keeps it correct. A condition that combines several fields names each one: a page-is-empty test, for example, counts every List of the page. When you change such a condition, or add, remove, rename, or retype a field or List that it reads or should read, read the whole condition again against the current types, update it, and view each outcome in the Display Template preview. Studio checks that every path resolves and that the expression is Boolean; it cannot tell that a newly added field belongs in the condition.

### Display Template switch

Owner: [presentation.md#display-template-switch](../reference/presentation.md#display-template-switch)

- `tpl.switch.first-true-case`: Cases are evaluated in file order; the first true case applies and evaluation stops, so a later case never repeats an earlier test. When no case is true, `else` applies, and without `else` the rule does not apply. `when` stays at the rule level and gates the whole switch. `:condition(...)` on the selector is the one-case shorthand.

### Display Template catalog

Owner: [presentation.md#display-template-catalog](../reference/presentation.md#display-template-catalog)

- `tpl.catalog.sequential-screens-are-workflow` (judgment): Sequential screens are a workflow, not a layout. A step-by-step layout would add a second Previous and Next inside screens that already move through a workflow's Previous and Next, which confuses users; an App that wants sequential screens splits the request into a [workflow](../reference/protocols.md#workflow-interactions), where the service validates, branches, and saves each step. `tabs` is how one request is split into sections.
- `layout.table.hidden-value-source`: A Hidden field remains a valid `value` source, as it is for `label` and `title`: the column shows its value read-only, while the field still has no row of its own. Hidden stays a schema attribute that the template never sets or overrides.

### Summary array items

Owner: [presentation.md#summary-array-items](../reference/presentation.md#summary-array-items)

- `layout.summary.when` (judgment): **Summary array items.** Use `summary` when a List needs brief item previews and a separate complete item editor or viewer:

  ```json
  {
    "select": "^<SaveIncomes>.items",
    "layout": "summary",
    "paths": [{"path": "document", "show": "variant"}, "owner", "document.W2.wages", "details"],
    "arrangement": "list"
  }
  ```

### Display Template catalog

Owner: [presentation.md#display-template-catalog](../reference/presentation.md#display-template-catalog)

- `layout.typed-path.header-and-presentation-only`: Without a `header`, an array-valued column is headed by the localized display name of its first path's field, whatever the rows hold; the same holds for a summary `arrangement: "table"` column built from an array entry. The `path` of a `{"path": …, "show": "variant"}` projection stays a plain path; a typed or array path there is refused. A table's `sort` names a column by its whole `value`, so for a column whose `value` is an array, `sort` is that same array; each row sorts by the value its first resolving path supplies, and rows whose cell is empty sort last. A typed path is presentation only: it reads a value and never changes what is stored, validated, copied, calculated, encoded or imported.
- `layout.summary.hidden-and-inactive`: Hidden nodes and their subtrees remain absent regardless of a path. Withholding an editor row through `when_single_choice` does not withhold its selected summary value. A path into an inactive Union variant contributes no list field and an empty table cell.
- `layout.summary.grid-cell`: **Cells.** Each cell is one bordered block drawn by the layout the template cascade selects for the member (`rows` by default, or any layout the catalog allows for the member's kind) with its ordinary labels, `suppress_label`, `format` and [`media`](../reference/presentation.md#inline-media-size) rules; Hidden nodes stay absent. The grid adds no title, line limit or image slot of its own: a title comes from a `card` layout's header, an image from a media field.
- `master-detail.app-owns-navigation` (judgment): **Complete navigation is the App's responsibility.** The leading page must expose the navigation needed to revisit editable sections and leave the arrangement. Its links and actions construct from its own context, never the right editor's current values or private workflow context. Selecting another destination closes an idle editor before opening it; selection does not submit or save the edit.
- `master-detail.context-dependent-previous-stays-full-page` (judgment): A flow requiring a previous action that depends on the editing context must remain a full-page flow.
- `master-detail.detail.selected-style` (judgment): The matching node exposes the [`:selected` pseudo-state](../reference/presentation.md#display-template-conditions); the template supplies its visible style, for example `"style": { "emphasis": "primary", "color": "primary" }`. Restrict `detail` to the intended navigation rows: including a summary link that repeats a topic creates an ambiguous match. Missing or ambiguous matches clear highlighting and never determine pane membership or require a full-page fallback. Clients do not compare link-request fields or choose the first row to infer selection.
- `layout.sections.tabs-and-rows-pair` (judgment): The same `sections` serve `tabs` and `rows`, so a pair of rules can show one Table as stacked sections in `edit` mode and as tabs in `view` mode with the same titles. Under `rows`, each section title is a heading above its fields; nothing is hidden, and moving to the next field crosses section boundaries in order.
- `layout.split.row-needs-no-condition` (judgment): A `row` split places its regions in lines under the [width-based layout](../reference/presentation.md#split-lines) rules, so it needs no device condition: on a phone its regions stack, and where they fit they share a line.

### Form grid columns

Owner: [presentation.md#form-grid-columns](../reference/presentation.md#form-grid-columns)

- `layout.form-grid.columns-maximum`: **`form-grid` columns.** `columns` is a maximum. The grid uses as many columns as fit its width at its `min_width` each (`compact` by default), with the gutter between them, at least one and at most the smaller of `columns` and the window's column cap. Fields flow in declaration order as before, and a `span` larger than the columns in use is reduced to them. For example, `columns: 3` in an `expanded` window shows one column below 736 dp, two from 736 dp, and three from 1112 dp.

### Labels and titles

Owner: [presentation.md#labels-and-titles](../reference/presentation.md#labels-and-titles)

- `label.label-replaces-name`: `label` replaces only the node's displayed name. The row keeps its value: the node's own value, or the child a [`promote`](../reference/presentation.md#promotion-flattening-and-union-selectors) rule puts on the row.
- `label.title-replaces-row`: `title` replaces the whole row with its text. The row's label and value are both removed, and nothing is promoted into it, because it has no value; a child that a `promote` rule names keeps its own row, and Studio warns that the `promote` has no effect.
- `label.text-not-typography` (judgment): Both change only the text, never its typography. The node keeps the typography of its position: a page's title is drawn as a page title, a group header inside a page as a group header, and a List member's row as a member's row. A [`role`](../reference/presentation.md#style-tokens) style token changes that look where an App wants another one, for example `role: label` for a quieter group header; no client reads `label` or `title` as a typography instruction.
- `label.typed-ancestor-occurrence`: A rule selecting a field below a typed ancestor, such as `^<FarmRental>.expenses.chemicals`, labels only the scalar reached that way. The same struct reused elsewhere, such as `^<Farm>.expenses`, keeps the schema's display name:

  ```json
  { "select": "^<FarmRental>.expenses.chemicals", "label": "DISPLAY_4835_LINE9_CHEMICALS" }
  ```

- `label.empty-source-fallback`: When the field is null, unanswered, or empty, the node draws as it would without the member, with its position for a List member, so a row is never blank. The display name stays the schema's meaning: AI tool descriptions and every declaration keep it, and `label` and `title` change only what a person sees.
- `label.source-hidden` (judgment): The source field is normally declared Hidden in the schema, so its value is not drawn twice; a Hidden field remains a valid source. Hidden stays a schema attribute that the template never sets: Studio neither hides a visible source field nor rejects it, but reports a warning at the `label` or `title` line of `display.json` that names the visible field and suggests Hidden. A visible source field keeps its own row.
- `label.on-promoted-field`: A `label` on a promoted field labels the row only with `use_parent_name` false; with `true` the row takes its struct's displayed name, the field's `label` shows nowhere, and Studio warns. A Hidden field has no row, so its `label` shows nowhere.
- `label.condition-limits`: Both resolve through the [cascade](../reference/presentation.md#display-template-cascade) and `when` like every other member; a [data condition](../reference/presentation.md#display-template-conditions), switch case, or pseudo-state never sets either; see the [condition limits](../reference/presentation.md#display-template-conditions).
- `label.combine.label-promote-child-name`:

  | On the struct | With | Result |
  | --- | --- | --- |
  | `label` | `promote` on a child, `use_parent_name` false | The row's label is the child's own name. Studio warns that the struct's `label` shows nowhere. |

- `label.combine.title-promote`:

  | On the struct | With | Result |
  | --- | --- | --- |
  | `title` | `promote` on a child | The child is not promoted and keeps its own row. Studio warns that the `promote` has no effect. |

- `label.links-stay`: With `label`, the link of a promoted value stays on the row; with `title`, a link on the struct stays on the title row ([actions](../reference/presentation.md#actions-on-a-promoted-or-suppressed-row)).

### Button labels

Owner: [presentation.md#button-labels](../reference/presentation.md#button-labels)

- `template.close.recapture-only`:

  | Slot | Existing control | Behavior retained |
  | --- | --- | --- |
  | `close` | Page, detail-pane, or stacked-link page Close | Performs the existing local dismissal without submitting; a stacked page's system Close may only be recaptioned, never removed. |

- `button.localization-and-no-inheritance`: Localization uses the package that authored the winning rule and the ordinary resource fallback. If a client cannot resolve a non-empty caption, it uses the platform caption, never a blank button or raw resource key. Omission preserves a lower-ranked override or the platform default; `null` overrides lower-ranked captions with that default. Slots do not inherit into children or another page.
- `button.single-action-source`: The `actions` slot also accepts exactly `{ "source": "single_action" }`. It uses the sole offered action's localized schema variant `Name`. Count the final offered set after `NextRequest`, `ConstrainedBy`, access privileges, `ExcludeUI`, and `LinkOnly` filtering, and an explicit `Actions` array when the page supplies one ([named action controls](../reference/protocols.md#named-action-controls)). With several choices, or a missing or blank name, the source supplies no caption, and the next source in the [Actions caption precedence](../reference/presentation.md#named-action-controls) applies: an explicit Actions field name, then the localized platform Actions caption, never a raw native name or payload type name. With no choices, retain the existing hidden control. This source uses the action's owning document, not the template author's resources or a display title. No other slot accepts it. An explicit `null` resets this source just like a resource override.
- `button.conditional.slot-cascade`: Inside a condition or switch branch, slots resolve independently under the [ordinary cascade](../reference/presentation.md#display-template-cascade). An omitted slot preserves the lower-ranked caption; `null` restores the platform caption. The `actions` slot still accepts `{ "source": "single_action" }`. A switch applies its first true case, otherwise `else`, otherwise nothing.
- `button.truthful-wording` (judgment): Choose wording that remains truthful for every state in which the page uses it. For example, "Go to…" can describe a sole destination-chooser request; "Return to review" is appropriate only when the offered operation actually returns to review. Renaming Actions does not change its target. "Save changes" does not promise a particular next page when the service can ask follow-up questions; Close must not imply submission or saving.

### Named action controls

Owner: [presentation.md#named-action-controls](../reference/presentation.md#named-action-controls)

- `control.declared-by-attribute`: The [ActionControl schema attribute](../reference/protocols.md#named-action-controls) declares the controls and their choices. No authored template is required. A named control uses its field's ordinary localized display name; this is a document name. The built-in Actions control keeps its platform caption when implicit, and an explicit Actions field can supply its default name. The selected Requests keep their own localized names inside a chooser.
- `control.actions-caption-precedence`: For Actions, an explicit `action_controls.Actions.label` takes precedence over `button_labels.actions`, then an explicit Actions field name, then the platform caption.

### Task-controlled section-hub action

Owner: [presentation.md#task-controlled-section-hub-action](../reference/presentation.md#task-controlled-section-hub-action)

- `hub-action.single-action-pattern` (judgment): For a walkthrough with section hubs and a main Review page, declare the continuation and review Requests in each hub's `NextRequest`, then let the task's `ConstrainedBy` value offer exactly one of them. Use the `single_action` caption source so its name and destination change together.
- `hub-action.link-only-rows` (judgment): Keep row-opening Requests `LinkOnly`; they remain available to their links without becoming additional Actions choices. Clients need no walkthrough flag, navigation history, or conditional caption rule.
- `hub-action.derive-from-completion-state` (judgment): Derive this choice from existing completion state rather than persisting a separate presentation flag. Remove a redundant next-step or Review row when this button owns that operation.
- `hub-action.separate-from-editor-next` (judgment): The section-hub Actions control and an editor's Next control remain separate. An editor's **Save and continue** submits its answers and follows the task's save route; a hub's **Continue** follows that section's onward route. The routes may differ. A hub action cannot submit an open editor's uncommitted fields. Preserve other required leading-page navigation when replacing a chooser with one direct action. Keep full captions on phones and use the responsive sizing and wrapping above.

### Style tokens

Owner: [presentation.md#style-tokens](../reference/presentation.md#style-tokens)

- `style.visual-only` (judgment): Style tokens are visual only. They never disable, hide, or change the interaction of what they style: a `muted` row keeps its links and its editing, and hiding stays the schema's Hidden. An App that wants a row out of reach leaves it out of the data or its allowed choices instead.

### Behavior hints

Owner: [presentation.md#behavior-hints](../reference/presentation.md#behavior-hints)

- `hint.placement-follows-source`: Every hint keeps its source attribute's placement rules, including the node kinds it may select, and a hint never makes a read-only value editable.

### Presentation locks

Owner: [presentation.md#presentation-locks](../reference/presentation.md#presentation-locks)

- `tpl.lock.meaning`: `lock` keeps the person from changing a value the App's business logic already set. Its most common use is with [`AskOnAdd`](../reference/presentation.md#add-time-values): the person gives a member's defining value at Add, and the lock keeps it fixed afterwards. It is presentation only and restricts the person alone: initialization, `CopyFrom`, `EditRequest` values, calculation and link-return `CopyTo` results, AI Import, draft restoration, and the App write a locked value exactly as an unlocked one, and its validation and save-path handling are unchanged.
- `tpl.lock.not-invariant`: **Not an invariant:** unlike `ReadOnly`, a lock never makes a [calculation input invariant](../reference/protocols.md#invariant-calculation-inputs); a locked input that triggers a calculation still declares `CalculationRequest`.
- `tpl.lock.which` (judgment): **Which to use:** a field the person never changes anywhere is schema `ReadOnly`; `lock` is for a value the person must not change in the contexts its rule selects, such as a document's kind after [`AskOnAdd`](../reference/presentation.md#add-time-values), with `lock: "variant"`.

### Template validation

Owner: [presentation.md#template-validation](../reference/presentation.md#template-validation)

- `tpl.validation.equal-rank-warning`: Rules of equal rank that set the same member on the same node are allowed, the later one winning, and are reported as a warning.
- `tpl.lint.view-screens` (judgment): Studio additionally lints more than about eight rendered (non-Hidden) fields shown together on a screen reachable from a Request, Response, or SubscriptionData payload or a function's configuration; a required field (one the person must answer before saving: not Nullable, with no `Default`; a Hidden or ReadOnly field is never required, because the person cannot answer it) folded closed while `form: phone` may apply; contrast by color role; and a `split` whose regions leave a required field unlisted in a nested region only.
- `tpl.lint.fields-per-screen` (judgment): The fields-per-screen lint reports each Table once, under its own name and source line; an inline Union branch is named `Union.Branch`. A screen sums the fields of its nested Tables, so several small Tables add up. A field whose content shows only on demand counts one row: a Union, whose chosen branch is linted as its own screen, so a Union counts by its largest branch and never the sum; a Nullable container; and a nested Table reported on its own. The fields-per-screen lint applies only in `view` mode: an editor is an ordinary scrolling form, where every field stays reachable in entry order, so many fields on one editing page draw no warning. Studio decides the mode from where a screen is reachable: from a Request whose node holds a field the person can change it is `edit`; from a Response, SubscriptionData, or a ReadOnly subtree it is `view`; a Table reachable both ways is linted for its `view` use only. In `view` mode a node is exempt under `sections` with `tabs` or `rows`, a `split`, or a fold that starts closed, from `display.json` or a source `Foldable`. Exemption holds only on the forms its rule's `when.form` names, so a rule limited to tablet and desktop still leaves one phone warning, which says so, and only in the modes its rule's `when.mode` names; a rule without `mode` applies in both.

### Common field metadata

Owner: [presentation.md#common-field-metadata](../reference/presentation.md#common-field-metadata)

- `field.promote-flatten-not-inherited`: `promote` and `flatten` apply to the node their rule selects, are not inherited through ordinary attribute resolution, and never override `Hidden`.
- `field.ask-on-add.with-lock` (judgment): **Commonly paired with [`lock`](../reference/presentation.md#presentation-locks).** The usual purpose of `AskOnAdd` is a value that decides what the member is, such as a document's form: the person gives it once at Add, and a template `lock` (on a Union, `lock: "variant"`) then keeps the person from changing it, so the item editor shows the chosen form and never offers to switch it.
- `field.enumsource.source-path-and-singleton`: The source uses a [Libertas Schema Path](../reference/schema.md#context-and-syntax) in the current data tree; it does not read the interaction reference context. An explicit Default or Fixed retains ordinary initialization; otherwise the selection is unanswered. A non-nullable host whose source offers exactly one selectable member (a member the picker offers: with its key set, within the host's bounds and step) takes that member when it is unanswered, under the [singleton rule](../reference/protocols.md#backend-constrained-choices) of allowed sets: when its editor initializes it, and whenever the source changes while it is still unanswered. A stored index or key is never replaced, and a List host is never filled this way. Nullable absence remains valid. A present selection must identify a current List item and satisfy the field's numeric bounds and step, including dynamic bounds. An empty or unavailable source offers no selectable options.

### Promotion, flattening, and Union selectors

Owner: [presentation.md#promotion-flattening-and-union-selectors](../reference/presentation.md#promotion-flattening-and-union-selectors)

- `promote.does-not-flatten`: `promote` never flattens the field's children: select the node with `flatten` when wanted. Both members are authored only in `display.json`. See [removed annotations](../reference/annotations.md#removed-annotations) when migrating older source.

### Field promotion

Owner: [presentation.md#field-promotion](../reference/presentation.md#field-promotion)

- `promote.first-visible-field`: On a field directly owned by a **Table**, the first visible promoted field in schema order becomes the Table row's value instead of occupying a separate child row. Additional promoted fields remain ordinary children. Promotion has no effect when the field's owner is not a Table, including a Union's selected branch.
- `promote.use-parent-name.true`: `true` (the default): use the owning struct's displayed name, which is its [`label`](../reference/presentation.md#labels-and-titles) when a rule gives it one.
- `promote.use-parent-name.false`: `false`: use the promoted field's own displayed name, which is its `label` when a rule gives it one. A `label` on the owning struct then shows nowhere, and Studio warns.
- `promote.list-member-position`: A List member whose chosen row label is static text (a displayed name, or a `label` in the `"KEY"` form) appends its one-based position to it (for example, “Household #1” with the default or “Marital status #1” with `use_parent_name` false), including for a Union. A label taken from the member's data (the `{ "field": … }` form) already tells the members apart, so it gets no position.

### Flattening

Owner: [presentation.md#flattening](../reference/presentation.md#flattening)

- `flatten.false-undoes`: `false` undoes a flattening that a lower-ranked rule set in the [cascade](../reference/presentation.md#display-template-cascade), such as a dependency package's template for its own type: the node draws with normal indentation and its name row.
- `flatten.struct`: **Struct.** `true` draws the struct's fields at the struct row's indentation and keeps the struct's name row. `{ "name_row": false }` also removes that name-only row; the fields stay visible and editable at the flattened level. A List member whose struct row is removed keeps its position and membership controls, and the fold control of a removed row passes to the first row the struct still produces, as [folding](../reference/presentation.md#folding-structs-and-unions) describes.
- `flatten.list`: **Array (List).** The List row keeps its title, count, and Add action; its members sit at the List's indentation instead of one level in. Each member keeps its own row, position, and actions.
- `flatten.union`: **Union.** The selected branch's fields sit at the Union row's indentation; the selector stays visible and editable.
- `flatten.default-indentation`: Without a `flatten` rule, children keep normal indentation; protocol page roots follow the [automatic root alignment](../reference/presentation.md#protocol-page-heading-and-top-bar). `flatten` never promotes a field, including a field that references the flattened type: select that field with `promote` when wanted. `Hidden` still suppresses the whole subtree.

### Actions on a promoted or suppressed row

Owner: [presentation.md#actions-on-a-promoted-or-suppressed-row](../reference/presentation.md#actions-on-a-promoted-or-suppressed-row)

- `promote.actions-follow-value`: Attributes that add an interaction to a value's row, currently `Link` and `LLMDocTrans` (AI Import), follow that value when `promote` or `flatten` changes its presentation. A promoted field's action appears on the owning Table's visible row. If a Table has no row of its own, because `flatten` with `{ "name_row": false }` removed it, its action appears on the first visible row that represents that Table; when the Table is the selected payload of a promoted Union, this is the Union's owning row. A List member keeps its own row and its own actions, also inside a flattened List. If several values represented by one row declare actions, the row offers each applicable action; a child `Link` takes precedence over an enclosing `Link` for the row tap target. With a [`label`](../reference/presentation.md#labels-and-titles) on the struct, a promoted value's link stays on the row; with a `title`, a link on the struct stays on the title row.
- `promote.actions.only-affordance-moves`: Only the **affordance** moves. Each action retains its declaring value, path, request context, eligibility, and pending state. Neither `promote` nor `flatten` copies an attribute to another schema or value node. `Hidden` suppresses the action with its value subtree; `ReadOnly`, `Fixed`, and other action-specific restrictions still apply. Data, validation, access, and lifecycle attributes do not move.
- `promote.classifier-import-action` (judgment): The **AI Import** action of a [classifier](../reference/llm-document-transformation.md#categorized-import) follows this placement. A classifier that is the Request root has no row of its own, so its action is the editor's first row, a full-width action. Outside [categorized layout](../reference/presentation.md#categorized-layout), category Lists keep their ordinary layout and controls. Categorized layout centralizes import and manual document creation at its outer classifier.

### Presentation only

Owner: [presentation.md#presentation-only](../reference/presentation.md#presentation-only)

- `promote.presentation-only`: Promotion and flattening do not change the schema or encoded data hierarchy. `CopyFrom`, `CopyTo`, `EnumSource`, validation paths, and backend field access still use the original structural paths, including every nested Union/table level. They do not supply a default, make required fields optional, or disable fields based on another answer.

### Data-titled rows

Owner: [presentation.md#data-titled-rows](../reference/presentation.md#data-titled-rows)

- `label.data-titled-rows` (judgment): A row is **data-titled** when its label comes from the data it shows, such as a calculated line labelled by its description with its amount as the value. Two independent rules express it:
- `label.data-titled.label-rule` (judgment): a Display Template [`label`](../reference/presentation.md#labels-and-titles) rule on the row's struct, usually `"select": "^<Type>"` or, for a List's members, `"^<Response.list>[]"`, names the field whose value labels the row;
- `label.data-titled.promote-rule` (judgment): a [`promote`](../reference/presentation.md#field-promotion) rule on a child of the struct promotes that child's value into the row.
- `label.title-for-text-only-rows` (judgment): A row that should show only text, with no value, such as a heading taken from the data, uses [`title`](../reference/presentation.md#labels-and-titles) instead; a `title` takes no promoted child, so the child keeps its own row.

### Protocol page heading and top bar

Owner: [presentation.md#protocol-page-heading-and-top-bar](../reference/presentation.md#protocol-page-heading-and-top-bar)

- `heading.root-suppressed-variant-title`: When the displayed root protocol Union has `suppress_label: true`, the page heading and its native top bar use the localized selected variant name, without the protocol label. Resolve an explicit `title` or `label` rule on the displayed Request or Response variant first, using the same rendering and empty-value fallback as [labels and titles](../reference/presentation.md#labels-and-titles). Otherwise use that variant's localized display name. With no selected variant, retain the ordinary localized protocol title rather than a blank heading.
- `heading.active-value-owns-title.authoring`: Target the displayed Request or Response variant with title rules under the [active-page title contract](../reference/presentation.md#protocol-page-heading-and-top-bar).
- `heading.actual-protocol-type-selector` (judgment): The selector in an App must use its actual protocol type name. No duplicated resource key or field is needed solely to remove “Tax return”.

### Single-choice rows

Owner: [presentation.md#single-choice-rows](../reference/presentation.md#single-choice-rows)

- `single-choice.member`: `when_single_choice` is a Display Template member with no source attribute. `"hide"` withholds the row of a choice field while its allowed set leaves only one answer, and that answer needs no input; `"show"`, the default, draws the row, so a higher-ranked rule can undo a lower one.
- `single-choice.payload`: `"payload"` applies only to a Union field. While the field's allowed set leaves exactly one answer, counted as [the single-answer rule](../reference/presentation.md#single-choice-rows) counts it (so for a Nullable field, null is a second answer and the field draws as under `"show"`), that answer is a variant carrying a payload, and the field holds it, the row draws **only that payload under the field's own label**: no variant selector, no variant name, and no label of its own for a payload that is one scalar or one Union (the field's label names it). A payload Table draws its fields in place under the field's label, as a flattened Table does. Otherwise, including when the set leaves several variants or a variant without a payload, the field draws as under `"show"`. Like `"hide"`, it changes only drawing: the value, validation, `CopyFrom`, `CopyTo`, calculations, encoding, AI Import and AI Search are unaffected, and a client that does not support it draws the row as `"show"`.

### Display Template selectors

Owner: [schema.md#display-template-selectors](../reference/schema.md#display-template-selectors)

- `template.state-selectors.style-only`:

  | Extension | Meaning |
  | --- | --- |
  | `:pending`, `:error`, `:selected` | A runtime state that no data expresses, as the [conditions](../reference/presentation.md#display-template-conditions) define. Style only; never on layout. |

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Display Template

Owner: [presentation.md#display-template](../reference/presentation.md#display-template)

- `tpl.template-only-members`: Promotion, flattening, labels and titles are authored only as template rules
- `tpl.source-attributes-compiled`: Source presentation attributes are compiled into the template at the lowest specificity and removed from the schema

### Template boundary

Owner: [presentation.md#template-boundary](../reference/presentation.md#template-boundary)

- `tpl.boundary.catalog-only`: A rule member outside the catalog, including every schema-only attribute, is rejected

### Display Template file

Owner: [presentation.md#display-template-file](../reference/presentation.md#display-template-file)

- `tpl.file.display-version`: `DisplayVersion` is 1 and `rules` is an ordered array
- `tpl.file.switch-exclusive`: A rule with `switch` has no other presentation member
- `tpl.file.rule-shape`: A rule has exactly one `select` and at least one presentation member; any other member is rejected

### Display Template conditions

Owner: [presentation.md#display-template-conditions](../reference/presentation.md#display-template-conditions)

- `tpl.when.keys`: `when` accepts only the closed keys and values; a list means any, several keys must all hold

### Editing and viewing

Owner: [presentation.md#editing-and-viewing](../reference/presentation.md#editing-and-viewing)

- `tpl.mode.no-tabs-in-edit`: An editable Table is never laid out as `tabs` while `edit` may apply
- `tpl.condition.pseudo-states-style-only`: `:pending`, `:error` and `:selected` change style only; `:condition(expression)` style, layout, fold state or existing page-control captions
- `tpl.condition.expression`: A data condition is a Boolean ValidationRules expression reading only the selected node and the reference context
- `tpl.condition.locale-independent`: A data condition or switch case never inspects text; compare a resource key or branch on typed data
- `tpl.condition.style-layout-fold-only`: A data condition may change style, layout, fold state or existing page-control captions only, never hide, lock or change a value

### Display Template switch

Owner: [presentation.md#display-template-switch](../reference/presentation.md#display-template-switch)

- `tpl.switch.shape`: A switch has at least one case, at most one `else`, Boolean conditions and no empty case; not beside a `:condition` selector

### Display Template catalog

Owner: [presentation.md#display-template-catalog](../reference/presentation.md#display-template-catalog)

- `tpl.catalog.closed`: The catalog is closed: no free-form CSS, pixel value, literal color or font
- `tpl.catalog.layout-node-kind`: A layout applies only to the node kinds its catalog row allows
- `tpl.catalog.no-stepper`: `stepper` is not a layout: Studio rejects it and clients draw `rows`
- `layout.form-grid`: `form-grid` on a Table flows fields into at most `columns` (1 to 4) per row; `span` takes more columns
- `layout.table`: `table` on a List of Tables: scalar `columns` in order and an optional display-only `sort` by one column
- `layout.summary`: `summary` on a List of Tables or Unions: previews from `paths`, `arrangement` `list` (default), `table` or `grid`
- `layout.grid`: `grid` on a List of Tables or Unions flows whole members in List order into cells using the member's layout and opening the item; optional `columns` maximum and `min_width`

### Categorized layout

Owner: [presentation.md#categorized-layout](../reference/presentation.md#categorized-layout)

- `layout.categorized`: `categorized` applies to a classifier Table, draws populated category Lists, and places one horizontal AI Import / Manually add document row before classifier headings and content, sticky below the page header while content scrolls, including when flattened

### Display Template catalog

Owner: [presentation.md#display-template-catalog](../reference/presentation.md#display-template-catalog)

- `layout.table.column-value`: A column `value` is a struct-field path, typed path or array of paths ending at a scalar; no two columns share one
- `layout.table.column-header`: A column `header` is a `StringResources` key or a field below the struct that holds the List
- `layout.summary.paths`: Summary `paths` are member-relative struct and variant paths that never cross a List or use an index
- `layout.summary.variant-projection`: A `show: variant` projection selects a Union and contributes its variant name; only `path` and `show` are accepted
- `layout.typed-path`: A typed path `^<Type>.rest` reads the nearest descendant of that type through struct fields and selected variants, never a List
- `layout.summary.grid-keys`: Optional grid `columns` (integer 1 to 8, a maximum) and `min_width` (width token, default `tile`) apply only to `grid` or summary `arrangement: grid`

### Master detail panes

Owner: [presentation.md#master-detail-panes](../reference/presentation.md#master-detail-panes)

- `master-detail.declaration`: A leading Response rule declares `right_pane` (editing Request variants of the same protocol) and `refresh` (a pure navigation Request)
- `master-detail.implies-layout`: `right_pane` implies `layout: master-detail`; an explicit layout must agree

### Display Template catalog

Owner: [presentation.md#display-template-catalog](../reference/presentation.md#display-template-catalog)

- `master-detail.panes`: `panes` takes only `leading` and `trailing` with a `min_width` token and a positive `ratio`, and never changes membership
- `master-detail.one-owner`: At most one leading Response may own an editing Request under overlapping device conditions
- `master-detail.no-data-condition`: A relationship cannot depend on a leading-page data condition, pseudo-state or switch
- `master-detail.editing-requests-only`: A relationship names editing Requests, not Response variants or navigation Requests
- `master-detail.detail`: `detail` names visible linked nodes reached through struct fields, each carrying a Link with NextResponse, none repeated
- `master-detail.refresh.loader`: The `refresh` loader is constructible from the leading Response and every listed editor, has no editable input, confirmation or subscription, and admits the leading Response
- `master-detail.right-pane-required`: New relationships declare `right_pane`; Studio rejects a `master-detail` rule without it
- `layout.sections`: `sections` lists direct children by native name with an optional `StringResources` title; a child appears in at most one section

### Layout groups

Owner: [presentation.md#layout-groups](../reference/presentation.md#layout-groups)

- `layout.groups`: Sections (`rows`/`tabs`/`accordion`) and split regions use `{fields,title?,layout?,style?}` plus layout members (`ratio`/`min_width` also on regions); only `rows` (default), `form-grid`, `card`, `split`, `tabs`, `accordion`, nested without a depth cap; titles belong to sections, fields stay in the parent pool without repeats, unlisted fields go to the last nested group; width fit stacks groups, never changing values, validation, calculation or encoding

### Display Template catalog

Owner: [presentation.md#display-template-catalog](../reference/presentation.md#display-template-catalog)

- `layout.split`: `split` has two or three regions, each a group with a positive `ratio` and direct-child `fields`

### Width-based layout

Owner: [presentation.md#width-based-layout](../reference/presentation.md#width-based-layout)

- `layout.width.size-classes`: Size classes are one vocabulary: `width` condition values and `min_width` tokens (compact 360, medium 600, expanded 840, large 1200, extra-large 1600 dp)
- `layout.width.tile-tokens`: `tile` (160 dp) and `tile-wide` (240 dp) are `min_width` tokens only for `form-grid` and `grid`, never width conditions, split regions or panes

### Labels and titles

Owner: [presentation.md#labels-and-titles](../reference/presentation.md#labels-and-titles)

- `label.field-source`: A `field` source is a struct-field path below the node (or below the struct holding a scalar field) ending at a String or FormattedText, Hidden or ReadOnly allowed
- `label.key-source`: `KEY` gives fixed text from a package `StringResources` key
- `label.union-key-only`: On a Union node or a node reached only through a Union, only the `KEY` form is valid
- `label.placement`: `label` selects a page, struct, Union field (`KEY`), List members or a scalar field; `title` never a scalar; neither a List itself or a scalar Union branch
- `label.exclusive`: `label` and `title` exclude each other, and neither combines with `suppress_label` on one node
- `label.combine.title-flatten-name-row`: `title` and `flatten` `{ name_row: false }` cannot combine on one struct

### Button labels

Owner: [presentation.md#button-labels](../reference/presentation.md#button-labels)

- `button.slot-values`: `button_labels` slots take a `StringResources` key or `null`; literal text, field sources, empty strings and unknown slots are invalid
- `button.whole-page-variants`: `button_labels` selects whole protocol page variants; `go_back` and `next` need a Request variant
- `button.conditional.allowed`: Data conditions and switch branches may set button_labels; pseudo-states may not

### Named action controls

Owner: [presentation.md#named-action-controls](../reference/presentation.md#named-action-controls)

- `control.action-controls`: `action_controls` on a whole Response or SubscriptionData page is keyed by a declared control name or `Actions` and sets `label`, `placement`, `emphasis` and `density`
- `control.placement`: `placement` is `secondary` or `header`; `secondary` keeps the complete visible group below the heading
- `control.label`: A control `label` is a `StringResources` key, a struct-field path to a String or ReadOnly FormattedText below the page, or `null`

### Style tokens

Owner: [presentation.md#style-tokens](../reference/presentation.md#style-tokens)

- `style.tokens`: Style tokens and their closed values: `density`, `emphasis`, `align`, `role`, `color` (platform roles only), `spacing` 0 to 4, `span` 1 to 4, `icon`
- `style.media-not-a-token`: Larger inline media is the `media` hint, not a style token

### Behavior hints

Owner: [presentation.md#behavior-hints](../reference/presentation.md#behavior-hints)

- `hint.inline`: `inline` (true or false): InlineChoices on an Enumeration or Union node
- `hint.fold`: `fold`: true, false or `{ initially_open }` on a Table, Union or List; false makes the node unfoldable
- `hint.promote`: `promote`: true, false or `{ use_parent_name }`, only on a rule selecting a field of a struct
- `hint.when-empty`: `when_empty` is show by default or hide: hide withholds the whole empty List row (title, count, header and content), restoring it with the first member; only ReadOnly/OutputOnly membership or no Add offered; values, validation, calculations and encoding unchanged
- `hint.when-embedded`: `when_embedded` is show by default or hide: hide withholds the node and its presentation subtree while its editor is embedded in a master-detail trailing pane; full-page placement restores it; values, validation, calculations and encoding are unchanged
- `hint.suppress-label`: `suppress_label` (true or false): omit the row label; on a List of structs the member names and positions
- `hint.include-description`: `include_description` (true or false): UIIncludeDescription
- `hint.format`: `format`: one portable pattern checked against the node's effective type
- `hint.scale`: `scale`: the scale type of a scale-bearing chart field under the chart type matrix
- `hint.guide`: `guide`: the position of the selected field's chart guide under the guide position matrix
- `hint.palette`: `palette`: `auto`, `categorical`, `sequential` or `diverging` on a chart root
- `hint.media`: `media` `{ max_height?, aspect? }` (at least one) only on `Bytes` with effective `MimeTypes`; `aspect` is `square` (1:1), `landscape` (4:3) or `portrait` (3:4), and aspect alone uses `medium` height
- `hint.chart`: `chart` `{ aspect, max_height }`, at least one key, only on a two-dimensional chart root; `aspect` is `square`, `4:3` or `16:9`

### Height tokens

Owner: [presentation.md#height-tokens](../reference/presentation.md#height-tokens)

- `hint.height-tokens`: `max_height` is `thumbnail`, `small`, `medium` or `large`, a share of the pane's visible height, never pixels

### Presentation locks

Owner: [presentation.md#presentation-locks](../reference/presentation.md#presentation-locks)

- `tpl.lock.placement`: `lock` is true, or `variant` on a Union; never Hidden, no data condition or `when`

### Template validation

Owner: [presentation.md#template-validation](../reference/presentation.md#template-validation)

- `tpl.validation.master-detail`: A master-detail relationship excludes a WorkflowRequest leading Response, and a `refresh` rule selects a Response without SubscriptionData

### Promotion, flattening, and Union selectors

Owner: [presentation.md#promotion-flattening-and-union-selectors](../reference/presentation.md#promotion-flattening-and-union-selectors)

- `promote.placement`: `promote` is valid only on a rule selecting a field of a struct; rejected on a type, List member, Union branch, function parameter or Response

### Flattening

Owner: [presentation.md#flattening](../reference/presentation.md#flattening)

- `flatten.placement`: `flatten` selects a struct, List or Union, never a List member or Union branch; `name_row` only on a struct without `title`

### SuppressLabel: label-free value rows

Owner: [presentation.md#suppresslabel-label-free-value-rows](../reference/presentation.md#suppresslabel-label-free-value-rows)

- `suppress.suppress-title-retired`: `SuppressTitle` and `suppress_title` are retired and rejected

### Inline choices

Owner: [presentation.md#inline-choices](../reference/presentation.md#inline-choices)

- `inline.placement`: `InlineChoices` goes on an Enumeration or Union field or type, read directly; rejected elsewhere and on a Bitflags catalog

### Single-choice rows

Owner: [presentation.md#single-choice-rows](../reference/presentation.md#single-choice-rows)

- `single-choice.placement`: `when_single_choice` selects a struct field carrying `ConstrainedBy` or a scalar `EnumSource` host, value `show`, `hide` or `payload` (Union only), never in a data condition or switch

### Display Template selectors

Owner: [schema.md#display-template-selectors](../reference/schema.md#display-template-selectors)

- `display.selector.schema-occurrences`: Studio validates a template selector like a CopyFrom path; it names schema occurrences, never one bound value
- `display.selector.forms`: Selector forms: strict `^<Variant...>`, `^<function...>` and `$.field`; relaxed `^<Type>`
- `display.selector.extensions`: Template-only selector extensions: `[]` List members, the descendant combinator, state pseudo-classes, `:condition(...)`
- `display.selector.no-index-alternatives`: Literal indices, comma alternatives and parent `^` steps are not selector syntax
- `display.selector.name-collisions`: A bare `^<Name>` naming a variant and a type is accepted only when the type occurs solely as that variant's payload

<!-- /generated -->
