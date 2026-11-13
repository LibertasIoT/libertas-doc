# Translating a schema document

Load this card when a task translates a package's localized document into
another locale. A translation task writes only its designated handoff output.
Its rules are generated from the
[translation document contract](../reference/translation.md), which owns
them. Translated prose follows the
[user-language invariant](../reference/documentation.md#universal-user-language-invariant),
quoted in the [source documentation card](source_documentation.md#universal-user-language-invariant).
For workspace preparation, locale selection, import and removal, follow the
[translation guide](../guides/translation.md).

## Order of work

1. Read the exported default-locale document and keep its structure
   (`translate.output.*`, `translate.locale.*`, `translate.structure.*`,
   `translate.nodes.*`).
2. Translate every user-text value and nothing structural
   (`translate.values.*`, `translate.keys.*`), keeping
   paragraph boundaries and intentionally empty strings
   (`translate.description.*`, `translate.empty.*`).
3. Translate only the literal text of each LMF1 message
   (`translate.lmf1.*`).
4. Write the output for import, where Studio applies the migration guarantees
   (`translate.migration.*`).

<!-- generated:codebot-rules task=translation -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Translation workflow

Owner: [presentation.md#translation-workflow](../reference/presentation.md#translation-workflow)

- `translation.output.generated-structure`: Use Studio's locale chooser and translation export/import workflow. Do not manually invent a locale entry or restructure a generated `.doc.json` file.

### Translation document contract

Owner: [translation.md#translation-document-contract](../reference/translation.md#translation-document-contract)

- `translate.output.json-only`: Read and write UTF-8 JSON without Markdown fences or explanatory output.
- `translate.locale.filename`: Set `Locale` to the locale tag used in the output filename.
- `translate.structure.preserve`: Preserve `DocumentVersion`, object structure, arrays, ordering, numeric values, and structural identifiers.
- `translate.keys.untranslated`: Do not translate structural keys, native type/function names, string-resource keys, or schema IDs.
- `translate.values.scope`: Translate package, function, and type `DisplayName` and `Description`; the `Name` and `Description` of every field, List container and item, Enumeration member, Union variant, and inline field; package `StringResources` values; and custom document-property string values. This includes `[DefaultText]` on fields and functions, Boolean `[TrueText]`/`[FalseText]`, and nullable-field `[NullText]` and `[SetText]`. Preserve their keys and translate their values as non-empty single lines.
- `translate.description.paragraphs`: Input `Description` values already have canonical HTML-style flow: source wrapping has become spaces, and paragraph boundaries are single LF characters. Preserve those paragraph boundaries in the translation and keep each paragraph flowing without line wrapping. Do not apply source-line normalization again to exported descriptions. This rule does not apply to custom-property or `StringResources` values.
- `translate.empty.preserve`: Preserve intentionally empty localized strings.
- `translate.lmf1.preserve`: Package `StringResources` values use Libertas Message Format 1 (LMF1), based on ICU MessageFormat 1 with Libertas types. Argument identifiers are zero-based positions, never printf placeholders or names. Preserve every position, type, style, selector key, plural offset, brace, and apostrophe escape exactly, including `LibertasDevice` and the Enumeration type-name style of `LibertasEnum`. Translate only literal message text, including literal text inside variants.
- `translate.nodes.unchanged`: Do not add, remove, reorder, or duplicate schema nodes.

### Migration guarantees

Owner: [translation.md#migration-guarantees](../reference/translation.md#migration-guarantees)

- `translate.migration.signature-match.authoring`: Review translations after schema changes under the [migration guarantees](../reference/translation.md#migration-guarantees); only compatible localized entries are retained automatically.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Translation document contract

Owner: [translation.md#translation-document-contract](../reference/translation.md#translation-document-contract)

- `translate.locale.syntax`: Locale tags are 1–128 ASCII alphanumerics in segments separated by one `-` or `_`

<!-- /generated -->
