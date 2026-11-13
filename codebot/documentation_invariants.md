# Schema documentation quick reference

Lookup for the [source documentation card](source_documentation.md), which
quotes every rule from the
[schema documentation contract](../reference/documentation.md). Function
comments follow the same card and the
[function-comment template](function_comment_invariants.md).

## Where each kind of text goes

| Source | Holds |
| --- | --- |
| TypeScript `///` in the selected entry module, or Rust `//!` in `src/lib.rs` | Package label and description |
| JSDoc `/** ... */` or Rust `///` on a declaration or typed parameter | That node's label (first ordinary line) and description |
| Ordinary `//` comments | Engineering notes; never localized |
| Package `StringResources` | Localized messages, permission reasons and other App text |

Rules: [source and output](source_documentation.md#source-and-output-invariants),
[user language](source_documentation.md#universal-user-language-invariant),
[exact source syntax](source_documentation.md#exact-source-document-syntax),
[StringResources prose](source_documentation.md#stringresources-prose).

## Property syntax

A custom property marker stands alone on its line after the ordinary text:

```text
[PropertyName]
Value through the line before the next marker or section end.
```

Rules: [custom document properties](source_documentation.md#custom-document-properties).

### Standard properties

<!-- LIBERTAS_DOCUMENT_EXTENSION_INVENTORY_START -->
- `[DefaultText]`: initial task name on a function, or initial text on an
  effective `String` or `EndpointServer` field.
- `[TrueText]` and `[FalseText]`: labels of a Boolean field's `true` and
  `false` choices.
- `[NullText]`: caption of a nullable field's explicit null.
- `[SetText]`: label of a nullable field's value branch, the lead-in to the
  value.
<!-- LIBERTAS_DOCUMENT_EXTENSION_INVENTORY_END -->

Rules: [standard properties](source_documentation.md#standard-properties).
