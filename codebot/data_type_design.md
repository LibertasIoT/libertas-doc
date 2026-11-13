# Data-type evolution

Load this card when a task changes a protocol or persistent-data Union, a
stored representation, or a persistent key of an App that has been published.
Choose initial representations under the shared
[user-facing and non-user-facing type guidance](../reference/app-contract.md#user-facing-and-non-user-facing-types).
Its rules are generated from the
[Libertas App data-type design guidelines](../reference/data-type-evolution.md)
and the [data model reference](../reference/schema.md), which own them; the
guidelines also hold the rationale, the language schematics and the review
checklist.

Nested Tables and Unions must evolve compatibly as well. Peers may
group a capability at different Union depths; only the
[capability prefix](../reference/protocols.md#capability-prefixes) is
translated, so nested optional variants inside a payload still only extend the
tail. Installing a new version never changes the version a task runs, so
older and newer versions coexist; which one reads data the other wrote
follows the linked directional rules
([task version selection](../reference/app-contract.md#task-version-selection)).

## Order of work

1. Compare each protocol and persistent-data Union with the last published
   version (`evolution.review-before-publish`, `evolution.union.*`).
2. Append, never insert or change (`evolution.append.*`,
   `evolution.preserve.*`, `evolution.declaration-order`).
3. Check persistent keys and stored values (`evolution.keys.*`,
   `enumsource.key-no-migration`, `schema.digest.key-layout`).
4. Check the readers of older data and peers (`evolution.data.*`,
   `evolution.directional`) and localized text (`docs.patch.*`).

<!-- generated:codebot-rules task=data-evolution -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Libertas App data-type design guidelines

Owner: [data-type-evolution.md#libertas-app-data-type-design-guidelines](../reference/data-type-evolution.md#libertas-app-data-type-design-guidelines)

- `evolution.unpublished-free` (judgment): An unpublished design may replace experimental types, encodings and APIs without retaining predecessor variants or implementing a data converter.
- `evolution.backward-compatible`: Libertas protocol schemas and persistent-data schemas are published contracts. Every published version of an App must remain backward compatible with the earlier versions of those contracts.

### Stable Union identity

Owner: [data-type-evolution.md#stable-union-identity](../reference/data-type-evolution.md#stable-union-identity)

- `evolution.union.stable-name`: Use one invariant Union type for each protocol or persistent-data family for the entire lifetime of the App. The Union's native type name is part of its schema identity and must not contain an App, schema, or release version.
- `evolution.union.no-versioned-container`: Do not publish container names such as `WeatherProtocolV1`, `WeatherProtocolV2`, `StoredDataV3`, or `Protocol2026`. A later App version must continue to reference the original Union; it must not replace that Union with a newly versioned container.
- `evolution.union.variant-names-free` (judgment): This restriction applies to the Union container. A newly appended payload type or variant may have a version-specific name when that is the clearest way to distinguish a new wire shape, although a semantic name is normally easier to maintain.

### Append-only evolution

Owner: [data-type-evolution.md#append-only-evolution](../reference/data-type-evolution.md#append-only-evolution)

- `evolution.append.at-end`: Evolve a published protocol or persistent-data Union only by appending new variants at the end of the existing declaration.
- `evolution.preserve.position`: preserve its declaration position and `Discriminator` (a variant's `Fixed` constrains its payload and never identifies the variant);
- `evolution.preserve.name`: preserve its native variant name;
- `evolution.preserve.payload`: preserve its payload's wire shape and meaning; and
- `evolution.preserve.roles`: preserve its protocol roles and other compatibility-relevant contract.
- `evolution.append.never-insert`: Never insert a new variant before an existing one, remove or reorder a variant, reuse an existing discriminator, or silently change an existing payload type.
- `evolution.append.new-variant-for-incompatible`: If a payload needs an incompatible shape or meaning, define a new payload/variant and append it.
- `evolution.preserve-container`: Do not evolve it by replacing the container:

  ```text
  WeatherProtocolV1 -> WeatherProtocolV2    invalid design
  ```

### Persistent keys

Owner: [data-type-evolution.md#persistent-keys](../reference/data-type-evolution.md#persistent-keys)

- `evolution.keys.append-only`: Each task binds one stable key union and one stable value union under the [storage schema contract](../reference/protocols.md#storage-schema-declarations). The append-only rule applies to both roots after publication. Existing key variants keep their tags, field order, numeric widths, byte bounds and optional slots.
- `evolution.keys.embedded-width`: An embedded union reserves its maximum payload width. Appending a variant to it must preserve that width and every existing encoding. If the new variant needs a larger slot, define a new layout and append a new root key variant; never enlarge the embedded slot of an existing published key variant. Reference changes or aliases must not hide an encoding change. The [key codec](../reference/protocols.md#key-encoding-and-ordering) defines byte equality.

### Why both rules are required

Owner: [data-type-evolution.md#why-both-rules-are-required](../reference/data-type-evolution.md#why-both-rules-are-required)

- `evolution.data.read-older`: For persistent data, a newer App version must continue to recognize values written under every older version of the same data schema. Appending a new payload case preserves the older cases while allowing the newer version to write a new data shape.
- `evolution.directional`: Backward compatibility is directional: the newer App must understand all older published variants. An older App is not expected to understand a variant first published later. Protocol code therefore treats later optional variants as capabilities and checks support before sending them.

### Declaration-order schematics

Owner: [data-type-evolution.md#declaration-order-schematics](../reference/data-type-evolution.md#declaration-order-schematics)

- `evolution.schematics.not-source`: The following blocks are deliberately non-copyable schematics: they omit payload declarations, required user documentation, schema roles, discriminators, derives, and export annotations so that only stable identity and declaration order are visible. Do not paste them into App source.
- `evolution.declaration-order`: The TypeScript variants must retain their stable numeric discriminator values. The Rust declaration order supplies the corresponding variant indices.

### Review checklist

Owner: [data-type-evolution.md#review-checklist](../reference/data-type-evolution.md#review-checklist)

- `evolution.review-before-publish` (judgment): Before publishing a new App version, compare each protocol and persistent-data Union with the last published version:

### 7. Path-based object and choice binding

Owner: [schema.md#7-path-based-object-and-choice-binding](../reference/schema.md#7-path-based-object-and-choice-binding)

- `enumsource.key-no-migration`: A List without a key keeps index storage. Adding or removing a key never migrates or remaps stored host values; the App owns rewriting any it keeps.

### 11.3 Revision digest

Owner: [schema.md#113-revision-digest](../reference/schema.md#113-revision-digest)

- `schema.digest.key-layout`: Indentation, CRLF/LF choice, and original object-key order therefore do not affect the digest. String contents, attributes, IDs, and semantically ordered arrays do. Key discriminator, field-order, numeric-width, byte-capacity and nullability changes affect this digest. Document translations are separate artifacts and do not affect it. Structural equality is not a substitute for checking whether an already published key layout preserves every encoded byte.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

<!-- /generated -->
