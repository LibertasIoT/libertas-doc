# Libertas CodeBot workflow

Use this workflow when writing or changing App source. The cards select the
context needed for the task; their linked references own the complete contracts.
Read the applicable owner before implementing a feature. Public documentation
and the project's public SDK provide this authoring context without private
maintainer instructions.

## Understand the App model

Read the [App interaction model](../reference/app-contract.md#application-model)
before choosing types and operations:

- **App is data interaction:** define the data, its meaning, the operations and
  how the App handles them.
- **Data is the GUI:** clients turn the declared data into editors, results and
  actions; source documentation explains them and the Display Template arranges
  them.
- **One interaction for humans, AI and other Apps:** all use the same declared
  Endpoint protocol and App behavior, under their applicable access checks.

This model supports Libertas's [collaboration goal](../principles.md#collaboration-goal).

## 1. Prepare the task

Read the App's own public development entry and contract when it has them.
Establish the requested behavior and current scope from its source and tests.
Preserve its source language unless the user requests a migration, and keep the
project's compatible dependency pins.

Then read:

1. [Source policy](sources.md): authority, SDK versions and missing contracts.
2. [App contract](app_contract.md) and its [App design card](app_design.md):
   design, source ownership, types, permissions, lifecycle and the compile loop.
3. [Code organization](code_organization.md): the generated rules for common
   responsibilities, the chosen language's layout, the change procedure and the
   review checklist when creating or reorganizing source; the
   [practice](../reference/code-organization.md) owns them.
4. One language: the [Rust](rust.md) or [TypeScript](typescript.md) quick guide
   and its generated rules card, [Rust App rules](rust_app.md) or
   [TypeScript App rules](typescript_app.md).
   [TypeScript is under development and not release ready](../reference/languages/typescript.md#development-status).
5. [Source documentation](source_documentation.md) for schema-visible
   declarations and for a public function, including changes to its behavior;
   its [quick reference](documentation_invariants.md) and
   [function-comment template](function_comment_invariants.md) give the shapes.
6. [Annotation inventory](schema_annotations.md) before using schema syntax.

## 2. Select the task contracts

Select every applicable row, including when extending an existing App. Open the
card first, then its linked owner for the feature being changed.

### Data and interaction

| The task changes | Required context |
| --- | --- |
| Types, fields, attributes, Enumerations, Unions, keys, defaults or Schema Paths | [Data model](data_model.md), with the [annotation inventory](schema_annotations.md) for each spelling. |
| An Endpoint, its protocol Union, roles, errors, access, hidden context or conformance | [Endpoint protocol](endpoint_protocol.md); the [protocols/runtime router](protocols_runtime.md) maps each Endpoint feature to its card. |
| Endpoint entry, Actions, `RequestConfirmation`, `ConstrainedBy`, action controls or native Back | [Actions, entry and choices](actions.md). |
| `Link` or `LinkOnly` | [Links](links.md). |
| Workflows: `EditRequest`, `WorkflowRequest` or `PrevRequest` | [Workflows](workflows.md). |
| Subscriptions and their reports | [Subscriptions](subscriptions.md). |
| AI Search | [AI Search](ai_search.md). |
| Endpoint callback code | [Endpoint callback code](endpoint_runtime.md) and the installed SDK. |
| Field calculations: `CalculationRequest`, `CalculationRoot`, calculation `CopyFrom`/`CopyTo` or a calculated allowed set | [Calculations](calculations.md), with the [Endpoint protocol card](endpoint_protocol.md). |
| Persistence, including Apps with no Endpoint | [Persistent data](data_api.md), [data names](../reference/message-format.md#data-and-database-names) and the installed SDK's Data API. |
| Existing protocol or stored representations | [Data-type evolution](data_type_design.md) and its full owner before changing names, cases, fields or ordering. |
| `CopyFrom`, `CopyTo` or constrained choices | The [data model card](data_model.md) for the [shared path contexts](../reference/schema.md#context-and-syntax) and copy rules, and the operation's card: [Endpoint protocol](endpoint_protocol.md), [links](links.md), [workflows](workflows.md), [subscriptions](subscriptions.md), [calculations](calculations.md) or, for constrained choices, [actions](actions.md). |
| Validation expressions, dynamic bounds or String formats | [Validation rules](validation.md), with the [ValidationRules quick reference](validation_rules.md) for the distinct rule, bound and Pattern forms. |
| `StringResources` templates, notifications, `FormattedText` values, data names or translated resources | [Messages (LMF1)](messages_lmf1.md); Studio does not check LMF1 templates. |

### Presentation and AI input

| The task changes | Required context |
| --- | --- |
| A field's text, metadata, answer controls, `Foldable`, `InlineChoices`, `SuppressLabel`, `UIIncludeDescription`, media or `Link` rendering | [Field presentation](field_presentation.md); the [presentation overview](presentation.md) routes the other presentation cards. |
| `display.json`: layouts, master-detail panes, labels and titles, page controls, promotion and flattening, conditions, hints and style | [Display Template](display_template.md) and its [display presets](display_template.md#display-presets). |
| Date and time types, `Money`, `PhysicalUnit`, `Bitflags`, `Format` or `FormattedText` placement | [Formatted output](formatted_output.md), with [Messages (LMF1)](messages_lmf1.md) for templates and [Translation](translation.md) for locale documents. |
| Charts | [Charts](charts.md), plus [3D charts](charts_3d.md) for a 3D chart; the cards link the full [2D](../reference/charts/2d.md) and [3D](../reference/charts/3d.md) contracts. |
| Document-sourced input, including a new field on an importing page | [AI Import](ai_import.md), including preservation of person-supplied values. |

### Dependencies and runtime APIs

| The task changes | Required context |
| --- | --- |
| External types or task dependencies | [App dependencies](app_design.md#external-types-and-app-dependencies) and the [task-creation dependency check](../reference/schema.md#62-task-pre-creation-static-dependency-check). |
| Permissions or TCP/UDP access | [Permissions](app_design.md#functions-permissions-and-singleton-tasks) and, for network access, the [network-access card](network_access.md). |
| Physical or virtual Matter devices | [Matter](matter.md), its descriptor/status contracts and the installed SDK's typed API. |
| Other runtime calls, such as timers or notifications | The chosen language reference and installed SDK declarations for the exact API and callback lifecycle. |
| Persistent keys, values, scans, batches or transactions | [Persistent data card](data_api.md), [key encoding](../reference/protocols.md#key-encoding-and-ordering) and [transactional KV API](../reference/protocols.md#transactional-key-value-api). |

## 3. Implement in source

Edit App source, ordinary project metadata such as `Cargo.toml` or
`package.json`, and the source Display Template (`display.json`). Generated
schemas, localized documents, dependency locks, target/release manifests and
allocated versions remain generated. A translation task writes only its
[designated handoff output](../reference/translation.md) under the
[translation card](translation.md).

Apply the required [App contract](app_contract.md) throughout implementation.
Use only public SDK types and listed annotations. Fix diagnostics in their
owning source; let Studio resolve dependencies instead of weakening types or
patching generated output. Report a missing public contract under the
[source policy](sources.md) before guessing behavior.

Use [recipes](recipes.md) and public [display presets](display_template.md#display-presets)
when a starting shape helps. Respect their stated limits: declaration examples
with empty functions still require runtime implementation.

## 4. Validate and hand off

Follow [delivery](delivery.md#validate-every-app-change) for native checking,
Studio validation and review of generated results. Compilation does not complete
deployment, translation or Plaza publication; follow those delivery sections
when the task includes them, with the
[deployment and publication card](deployment_publication.md). Deploy only when
requested by the user.

Finish with the [handoff report](delivery.md#report-the-result): changed behavior
and contracts, permissions/hosts/dependencies, wire/order impact, checks and
remaining work.

For guided learning, start at the [human homepage](../index.md).
