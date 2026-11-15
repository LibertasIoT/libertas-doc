# Rust App quick guide

The installed public
[`libertas-rs`](https://github.com/LibertasIoT/libertas-rs) rustdoc owns exact
runtime APIs. The [Rust manual](../reference/languages/rust.md) owns full language
behavior; use the [annotation inventory](schema_annotations.md) for every
schema macro.

## What to load

- The [Rust App rules](rust_app.md) card: every project, discovery, export,
  type-mapping, rustdoc, attribute-spelling, protocol, SDK runtime, persistent
  data, string resource, Tokio and native build rule generated from the manual.
- The [code organization](code_organization.md) card for module boundaries,
  explicit type exports and thin entry functions, with the manual's
  [Rust layout](../reference/code-organization.md#rust-layout).
- The shared cards for the meaning of what the attributes declare:
  [App design](app_design.md), [documentation](documentation_invariants.md),
  [protocols and runtime](protocols_runtime.md) and its feature cards,
  [persistent data](data_api.md), [calculations](calculations.md),
  [validation rules](validation_rules.md), [presentation](presentation.md),
  [messages (LMF1)](messages_lmf1.md), [AI Import](ai_import.md),
  [data-type evolution](data_type_design.md) and [Matter](matter.md).

## Where the manual helps most

- [Optional types](../reference/languages/rust.md#optional-types) and
  [labeled decisions](../reference/languages/rust.md#labeled-decisions-and-internal-absence):
  worked examples for `String` versus `Option<T>`, Unions and Enumerations.
- [General field attributes](../reference/languages/rust.md#general-field-attributes):
  the Rust spelling of the AI Import example; the
  [AI Import card](ai_import.md) owns the blank rules.
- [Formatted runtime text](../reference/languages/rust.md#formatted-runtime-text):
  `libertas_formatted_text` with typed arguments under the
  [messages card](messages_lmf1.md).
- [Matter SDK dependency](../reference/languages/rust.md#matter-sdk-dependency)
  and [native build packages](../reference/languages/rust.md#declaring-native-build-packages)
  for `Cargo.toml`.
