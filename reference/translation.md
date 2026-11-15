# Translation document contract

<!-- rule: translate.output.json-only class=hard enforced=no tasks=translation -->
- Read and write UTF-8 JSON without Markdown fences or explanatory output. <!-- rule: translate.locale.filename class=hard enforced=no tasks=translation title="The translation document's `Locale` matches the locale tag in its output filename" -->
- Set `Locale` to the locale tag used in the output filename. <!-- rule: translate.locale.syntax class=hard enforced=yes tasks=translation title="Locale tags are 1–128 ASCII alphanumerics in segments separated by one `-` or `_`" -->
- Locale tags are 1–128 ASCII alphanumeric characters grouped into segments
  separated by one `-` or `_`; leading, trailing, or repeated separators are
  invalid. <!-- rule: translate.structure.preserve class=hard enforced=no tasks=translation -->
- Preserve `DocumentVersion`, object structure, arrays, ordering, numeric values,
  and structural identifiers. <!-- rule: translate.keys.untranslated class=hard enforced=no tasks=translation -->
- Do not translate structural keys, native type/function names, string-resource
  keys, or schema IDs. <!-- rule: translate.values.scope class=hard enforced=no tasks=translation -->
- Translate package, function, and type `DisplayName` and `Description`; the
  `Name` and `Description` of every field, List container and item,
  Enumeration member, Union variant, and inline field; package
  `StringResources` values; and custom document-property string values. This
  includes `[DefaultText]` on fields and
  functions, Boolean `[TrueText]`/`[FalseText]`, and nullable-field `[NullText]`
  and `[SetText]`.
  Preserve their keys and translate their values as non-empty single lines.
- Apply the [user-language invariant](documentation.md#universal-user-language-invariant)
  to translated prose: use natural, idiomatic, technology-neutral wording while
  preserving every material meaning. <!-- rule: translate.description.paragraphs class=hard enforced=no tasks=translation -->
- Input `Description` values already have canonical HTML-style flow: source
  wrapping has become spaces, and paragraph boundaries are single LF characters.
  Preserve those paragraph boundaries in the translation and keep each paragraph
  flowing without line wrapping. Do not apply source-line normalization again
  to exported descriptions. This rule does not apply to custom-property or
  `StringResources` values. <!-- rule: translate.empty.preserve class=hard enforced=no tasks=translation -->
- Preserve intentionally empty localized strings. <!-- rule: translate.lmf1.preserve class=hard enforced=no tasks=translation -->
- Package `StringResources` values use Libertas Message Format 1 (LMF1), based on ICU
  MessageFormat 1 with Libertas types. Argument identifiers are zero-based
  positions, never printf placeholders or names. Preserve every position, type,
  style, selector key, plural offset, brace, and apostrophe escape exactly,
  including `LibertasDevice` and the Enumeration type-name style of
  `LibertasEnum`. Translate only literal message text, including literal text
  inside variants. <!-- rule: translate.nodes.unchanged class=hard enforced=no tasks=translation -->
- Do not add, remove, reorder, or duplicate schema nodes.

## Migration guarantees

<!-- rule: translate.migration.signature-match.authoring class=hard enforced=no tasks=translation -->
Review translations after schema changes under the [migration guarantees](#migration-guarantees); only compatible localized entries are retained automatically.

<!-- /rule -->

<!-- rule: translate.migration.signature-match class=hard enforced=no tasks=client-runtime -->
When both signatures are known, functions match by stable native name and types
with the same native name are disambiguated by namespace identity. Fields and
enum entries match by their complete native declaration path. New localized values
receive default-locale text with an explicit review warning; obsolete entries
are removed; translated custom attributes and string resources are retained;
and regenerated numeric IDs are installed.

<!-- rule: translate.migration.shape-fallback class=hard enforced=no tasks=client-runtime -->
When an old signature is unavailable, Studio preserves field-level translation
only if the complete numeric shape is identical. If the shape changed, it copies
the new default fields instead of risking a translation being attached to the
wrong field.


For workspace preparation, locale selection, import, and removal, follow the
[translation guide](../guides/translation.md).

## Translation development files

<!-- rule: translate.project.git-inventory class=hard enforced=no tasks=deployment-publication -->
`translations/project.json` declares the complete managed locale inventory for
one package. Saving translations in Studio creates this inventory. Commit it,
the declared `translations/<package>.<locale>.doc.json` files, and the whole
`translations/state/` directory together. Generated prompts, worksets, results,
and the older `manifest.json` export are not required for deployment from a fresh
checkout; a managed project's `manifest.json` carries no per-locale translation
baselines, because its state supersedes them. A package without `project.json`
keeps the existing Hub-document retention behavior until its translations are
saved into a managed project.

The inventory has `FormatVersion: 2`, `PackageName`, `SourceSha256` (the digest
of `state/source.json`, present when a locale is declared), and `Locales`, keyed
by locale. Each locale records `DocumentSha256` and `StateSha256`, the digest of
its state index. It may also have `RequireReviewed`, an array of declared
locales subject to the project review policy. There may be at most 255 target
locales, plus the default locale. Studio still reads a format-1 project, whose
`translations/state/<locale>.json` files each carry a whole locale state, and its
first save converts it to format 2 without changing any value, check, or document.

<!-- rule: translate.state.native-identity class=hard enforced=no tasks=client-runtime -->
One source snapshot serves every locale. `translations/state/source.json` has
`FormatVersion: 2`, `PackageName`, `SourceLocale`, `SourceSchema`, and
`SourceDocument`. `SourceDocument` contains the exact original-locale strings;
the schema snapshot excludes transient source locations, generated revision
metadata, and deployment-assigned dependency locks. Runtime schema/dependency
identity retains its full lock under the [deployment contract](deployment.md).
Each value's original text and deterministic declaration context are rebuilt
from this snapshot, so no locale stores them.

<!-- rule: translate.state.locale-chunks class=hard enforced=no tasks=client-runtime -->
Each locale's state is the directory `translations/state/<locale>/` of at most
256 bucket files, `00.json` to `ff.json`, and `index.json`. A value belongs to
the bucket named by the first byte, in lowercase hex, of the SHA-256 of its
declaration's identity (the package, one function, or one type), so a
declaration's values stay together; an empty bucket has no file. A bucket file
holds `Entries`, `Translated`, and `Reviewed` for its values. `index.json` has
`FormatVersion: 2`, `PackageName`, `Locale`, and `Chunks`, mapping each bucket
name to the digest of its file. Each entry is keyed by the stable native declaration and
localized property or resource key. Numeric schema IDs and array positions
locate values but do not identify translation work. Every value of the snapshot
has exactly one entry, which contains:

- `Current`: the digest of its current exact source, context, and target pair:
  a `Revision` object with the original `Source` text and deterministic
  `Context` string rebuilt from the snapshot, and the translated `Target` text.
- `Target`: the translated text.
- Optional `Pending`: `new` or `source-changed`.

A save rewrites only the buckets whose values or checks changed, their index,
and changed documents; a source change also writes the snapshot once.

<!-- rule: translate.state.exact-pair-checks class=hard enforced=no tasks=client-runtime -->
`Translated` and `Reviewed` are separate identity-keyed claim maps. Each claim
contains the current `Revision` digest, ISO timestamp `At`, and attribution `By`.
Only the current pair and current claims are stored; previous text, withdrawn
claims, and removed entries are not retained in the files. Git owns
translation history. Reviewed requires current completion for the same exact
pair. Review is a recorded human
assertion, not an authenticated professional certification. Unknown imported
wording establishes no review. Completed translation output establishes
Translated for its selected entries. Editing an already completed current target
preserves completion on a new text revision and clears review. Changing its
source or relevant declaration context clears both current checks. Unchanged
pairs retain both claims, including after schema IDs are regenerated.

<!-- rule: translate.state.source-locale-migration class=hard enforced=no tasks=client-runtime -->
Changing the default/source locale requires an explicit migration; Studio does
not reinterpret retained original text under a different language tag.

<!-- rule: translate.state.preserve-working-text class=hard enforced=no tasks=client-runtime -->
New source entries receive original-locale fallback text and start with both
checks unchecked. Existing unknown text is retained; missing provenance does
not trigger a whole-locale replacement. The explicit **Fresh translation**
action may replace the selected locale's working text. Previous versions are
recoverable from Git when committed. Removing a value removes its entry and
claims; removing a locale removes its inventory entry, document, and state directory.
Reintroduced values start unchecked and re-adoption records only current data.

<!-- rule: translate.project.integrity-reconciliation class=hard enforced=yes tasks=deployment-publication title="Translation text, state, and inventory must be reconciled before deployment" -->
Digests use `sha256:` plus lowercase hex over canonical UTF-8 JSON: object keys
sorted by UTF-8 byte order, arrays unchanged, two-space indentation, and a final
LF. Studio saves documents, buckets, indexes, and the snapshot before the
inventory digests that bind them. Missing files, mismatched snapshot, index, or
bucket digests, or a partial Git merge stop deployment before Hub mutation. For
externally edited text with intact state, reopening the editor
offers **Reconcile changed translations**; it keeps the current working wording,
invalidates affected approvals, and saves the matching files together. A
corrupted or half-merged state must first be repaired in Git.

## Completion and review in Studio

<!-- rule: translate.editor.schema-match class=hard enforced=no tasks=client-runtime -->
App Translation works on one schema: the App's local source, compiled now, which
must equal the schema installed on the selected Hub. When App Translation opens,
and before every automatic translation, import, save, or locale removal, Studio
compiles the local source and compares its `Revision.ContentSha256` with the
installed schema's. If they differ, Studio reports the mismatch, naming the Hub
and its installed App version, and does nothing else: no translation, no file
write, no upload. Deploy first; a deployment carries the local translations with
its schema. Translation inside a deployment's schema review works on the schema
being deployed and is part of that deployment.

<!-- rule: translate.editor.aggregate-checks class=hard enforced=no tasks=client-runtime -->
Every package/type/function node, field row, and individual localized value has
separate Translated and Reviewed checks. Parent checks aggregate their entire
scope and show mixed state. Filters and collapsed rows do not narrow a bulk
check. Bulk review requires every value in that scope to be currently translated.
The selected-node review table shows current original and target text, pending
reasons, counts, filters, and navigation to
untranslated or unreviewed text. Changes are saved with **OK** or **Submit**;
closing without saving discards unsaved check changes.

<!-- rule: translate.project.optional-review-policy class=hard enforced=yes tasks=deployment-publication title="A required-review locale stays a draft until all current values are reviewed" -->
The default workflow has no required-review gate: completed translations still
upload immediately. An optional inventory `RequireReviewed` policy prevents
Studio uploads, deployments, and publication of a required locale with pending
review. Translation generation may save drafts and continue the locale queue;
those drafts are not uploaded. This policy is enforced by Studio's authoring
paths, not a Hub-wide admission rule or a new App annotation.

## Translation model context

<!-- rule: translate.prompt.context-preserved class=hard enforced=no tasks=client-runtime -->
Every automatic translation batch receives the App's source name and description,
each selected entry's role and native declaration context, and relevant surrounding
source text. Current reviewed source/target pairs supply terminology examples;
unknown wording and stale approvals are not presented as approved terminology.
Context is read-only and has a separate bounded reference budget. Batch splitting
retains context for every batch. Only the selected paths may appear in the result,
and Studio validates that exact scope before merging. Regional locale wording,
concise labels, precise explanations, paragraph boundaries, and message-format
syntax remain governed by the rules above. No unchanged reviewed entry becomes
translation work merely because it supplies context.

## Deployment of translations

<!-- rule: translate.deploy.local-authority class=hard enforced=no tasks=deployment-publication -->
Normal Studio and MCP deployment reads the local managed inventory automatically,
including for each local dependency. It projects new original-locale fallbacks
and stale-check invalidation without rewriting tracked files. The declared locale
set wins over the Hub's locale set; omitted locales are removed from the current
set, with historical artifacts retained. Changing only translation text
synchronizes metadata without rebuilding the App or changing its App version,
code, schema, dependency pins, or original build provenance. Changing only
checks changes nothing on the Hub. Repeating the same bundle is idempotent. A
changed project bundle after planning requires a new plan.

<!-- rule: translate.deploy.project-files-only class=hard enforced=no tasks=client-runtime -->
Translation and review state lives only in the
[translation development files](#translation-development-files); Git owns it
and its history. A deployment or metadata update sends the Hub only the locale
documents, which the Hub stores exactly as supplied; a translation save is a
[documents-only update](publication.md#translation-review-state). The Hub keeps no
translation authoring companion and no review state:
[publication](publication.md#translation-review-state) owns that rule.
Development files stay readable canonical JSON.
Runtime `.doc.json` remains version `0.0.2`.
