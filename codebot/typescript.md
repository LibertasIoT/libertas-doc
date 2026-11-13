# TypeScript App quick guide

**TypeScript is under development and not release ready.** Follow the manual's
[development status](../reference/languages/typescript.md#development-status)
and SDK prerequisites before starting or changing a TypeScript App.

The installed
[`@libertas-ai/types`](https://www.npmjs.com/package/@libertas-ai/types)
`index.d.ts` owns exact runtime APIs. The
[TypeScript manual](../reference/languages/typescript.md) owns full language behavior; use
the [annotation inventory](schema_annotations.md) for every JSDoc tag.

## What to load

- The [TypeScript App rules](typescript_app.md) card: every SDK, project,
  discovery, export, type-mapping, JSDoc, decorator-spelling, protocol,
  persistent data and string resource rule generated from the manual.
- The [code organization](code_organization.md) card for module boundaries,
  explicit type exports and thin entry functions, with the manual's
  [TypeScript layout](../reference/code-organization.md#typescript-layout).
- The shared cards for the meaning of what the decorators declare:
  [App design](app_design.md), [documentation](documentation_invariants.md),
  [protocols and runtime](protocols_runtime.md) and its feature cards,
  [persistent data](data_api.md), [calculations](calculations.md),
  [validation rules](validation_rules.md), [presentation](presentation.md),
  [messages (LMF1)](messages_lmf1.md), [AI Import](ai_import.md),
  [data-type evolution](data_type_design.md) and [Matter](matter.md).

## Where the manual helps most

- [Project configuration baseline](../reference/languages/typescript.md#project-configuration-baseline):
  the strict `tsconfig.json` for a new ESM App.
- [Enumerations](../reference/languages/typescript.md#enumerations) and
  [discriminated unions](../reference/languages/typescript.md#discriminated-unions-union-types):
  the three enumeration forms and numeric first-field discriminators.
- [Optional types](../reference/languages/typescript.md#optional-types): `string`
  with an empty Default versus `?`, `| null` and `| undefined`.
- [Formatted runtime text](../reference/languages/typescript.md#formatted-runtime-text):
  `libertasFormattedText` with typed argument objects under the
  [messages card](messages_lmf1.md).
