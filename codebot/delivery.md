# Validation and delivery

## Validate every App change

For App-source work:

1. Save source and resolve the project's local dependencies. Run
   `cargo check --lib` or the project's TypeScript typecheck/build, plus the
   applicable behavior tests selected under the
   [change workflow](code_organization.md) (`org.change.*`).
2. After schema-affecting changes, compile with Studio's `compile_schema` tool
   under the [compile rules](app_design.md#schema-compilation-with-codex) until
   validation succeeds.
3. Inspect the generated schema, default document and compiled Display Template.
   Review every schema-visible node, permission, access host and localized reason
   against the App's behavior. Use the [App review checklist](app_contract.md#completion-checklist)
   for design, discovery, import and wire concerns.
4. For Matter, follow the [Matter card](matter.md) (descriptors come from
   Studio's editors).

App authoring never hand-edits generated schema/document JSON, dependency locks,
`AppDependencies`, `DistroDependencies`, target manifests or release versions.
Rust native build packages use only the supported
[Cargo metadata](../reference/languages/rust.md#declaring-native-build-packages).

## Deploy and publish when requested

Deploy or publish only when the user requests it, under the
[deployment and publication card](deployment_publication.md) and the current
[Studio](../guides/studio.md) and [Plaza](../guides/plaza.md) guides.

## Translate through the locale workflow

For a translation task, write only the designated handoff output and preserve
all structure and message arguments under the
[translation contract](../reference/translation.md). The
[workflow guide](../guides/translation.md) owns locale choices, immediate import
persistence, review and removal. Do not defer a Hub-connected import until OK.

## Report the result

State the changed files, functions, behavior and contracts; new permissions,
hosts and dependencies; wire/order impact; and checks actually run. Identify
remaining validation, deployment, translation or publication work. Do not report
compilation, installation or publication as proof of an untested user outcome.
