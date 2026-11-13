# TypeScript App rules

**TypeScript is under development and not release ready** (`ts.status.*`).
Load this card when writing or changing a TypeScript App: its project
configuration and entry module, schema types and exports, JSDoc decorator
spelling and placement, SDK runtime calls, persistent data and string
resources. Its rules are generated from the
[TypeScript manual](../reference/languages/typescript.md), which owns them;
the [TypeScript quick guide](typescript.md) gives the framing and the
[annotation inventory](schema_annotations.md) the complete spelling table.
Rules shared by every language stay in their owning cards and are linked from
the manual: the semantics of each decorator, calculations, validation rules,
presentation, documentation and protocols.

## Order of work

1. Confirm the SDK and project baseline and the discovery roots
   (`ts.status.*`, `ts.sdk.*`, `ts.project.*`, `ts.dependencies.*`,
   `ts.discovery.*`, `ts.types-only.*`, `ts.packages.*`).
2. Shape the schema types and exports (`ts.types.*`, `ts.enum.*`,
   `ts.union.*`, `ts.exports.*`, `ts.scope.*`, `ts.foreign-type.*`).
3. Document every schema-visible declaration in JSDoc (`ts.doc.*`).
4. Spell each decorator exactly (`ts.attr.*`, `ts.number.*`, `ts.size.*`,
   `ts.pattern.*`, `ts.bitflags.*`, `ts.array.*`, `ts.enum-source.*`,
   `ts.enum-attr.*`, `ts.llm-doc.*`, `ts.matter.*`, `ts.presentation.*`,
   `ts.formatted-text.*`, `ts.chart.*`, `ts.chart3d.*`, `ts.ai.*`).
5. Declare the endpoint protocol (`ts.endpoint.*`, `ts.protocol.*`,
   `ts.conformance.*`, `ts.calc.*`, `ts.workflow.*`, `ts.link.*`,
   `ts.choices.*`, `ts.append.*`, `ts.validation.*`).
6. Implement the task: functions, permissions, hosts, string resources and
   data (`ts.singleton.*`, `ts.permissions.*`, `ts.access-host.*`,
   `ts.string-resources.*`, `ts.data.*`).
7. Test (`ts.testing.*`), then compile with Studio and fix every reported
   rule.

<!-- generated:codebot-rules task=typescript-app -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Development status

Owner: [typescript.md#development-status](../reference/languages/typescript.md#development-status)

- `ts.status.under-development`: **TypeScript support is under development and is not release ready.** This manual and its examples describe the development contract; they do not certify a released TypeScript SDK or a production-ready App path. A matching SDK is still required for compilation and the target Hub's runtime ABI.

### Source code is the ground truth

Owner: [typescript.md#source-code-is-the-ground-truth](../reference/languages/typescript.md#source-code-is-the-ground-truth)

- `ts.discovery.roots`: In an ordinary package, exported non-ignored function declarations in that entry module are the roots and only the minimum transitive type closure from their arguments and reachable schema-reference decorators is published. In a `TypesOnly` package, every public package-owned named type exported by the entry module is automatically a root unless ignored.
- `ts.dependencies.no-manual-pins`: The parser records external package-qualified references and emits an empty intermediate `AppDependencies` list. Studio assigns required exact `Version` pins during the [dependency-first deployment workflow](../reference/deployment.md). Application source must never predict or manually maintain these pins.
- `ts.sdk.declarations-normative`: For TypeScript SDK behavior, the public comments in [`libertas-ts/index.d.ts`](https://www.npmjs.com/package/@libertas-ai/types) are the sole ground truth. This manual is a non-normative usage summary. If it, an example, a test, or another document disagrees with those SDK comments, follow the SDK comments.
- `ts.sdk.bigint-identities`: The 64-bit SDK (0.2.0) represents device, endpoint, LAN-device, user, and peer identities as `bigint` aliases in the signed 64-bit range. Keep those values and collection keys as bigint; converting through `number` loses precision above `2^53`. Preserve negative runtime identities, including opaque peer handles; their lifetime follows the [system identity rules](../reference/schema.md#24-runtime-types). Use `0n` only where an API defines a system context. Interaction transaction IDs remain numbers. Rebuild Apps against this SDK for the matching Hub runtime.
- `ts.sdk.matching-sdk-required`: The older 0.1.x SDK uses the earlier identity ABI and is not a substitute for the matching 64-bit SDK on a modern Hub. Confirm that the required SDK is available and installed before implementing runtime calls. If its public distribution is unavailable, report that prerequisite; do not use a private platform checkout, invent declarations, or silently downgrade the SDK.

### Project configuration baseline

Owner: [typescript.md#project-configuration-baseline](../reference/languages/typescript.md#project-configuration-baseline)

- `ts.project.strict-baseline`: Preserve an existing project's SDK and compiler pins. For a new ESM App, this is the supported strict baseline; the `types` entry makes the SDK's ambient Libertas declarations available without invented imports:

  ```json
  {
    "compilerOptions": {
      "module": "nodenext",
      "target": "esnext",
      "lib": ["esnext"],
      "types": ["@libertas-ai/types"],
      "strict": true,
      "verbatimModuleSyntax": true,
      "isolatedModules": true,
      "skipLibCheck": true
    },
    "include": ["src/**/*.ts"]
  }
  ```

- `ts.project.typecheck-first`: Run the project's build/typecheck or at least `tsc --noEmit` before schema generation.

### Documents and comments

Owner: [typescript.md#documents-and-comments](../reference/languages/typescript.md#documents-and-comments)

- `ts.doc.package-comments`: The triple-slash `///` comments define the **Package-level** documentation.
- `ts.doc.package-comments-placement`: They should only be placed at the very top of your entry file (e.g., `index.ts`).
- `ts.doc.jsdoc-blocks`: Standard JSDoc block comments (`/** ... */` or `/* ... */`) are used for documenting types, interfaces, classes, enums, and properties.
- `ts.doc.short-document`: **Short Document (First Line)**: The very first line of the comment is extracted as the **Short Document**, which is used as the Short Name or Display Name in the UI.
- `ts.doc.long-document`: **Long Document (Remaining Lines)**: Any remaining lines after the first line are extracted as the **Long Document**, which serves as the detailed Description field in the UI or LLM tool call description. Long descriptions use HTML-style flow: one logical CR/LF break becomes exactly one space, while two or more logical breaks become one LF paragraph break. CRLF counts as one break, so leave a blank JSDoc line only when a paragraph break is intended.
- `ts.doc.enum-and-variant-text`: **Enums & Union Variants**: Documenting an underlying type or enum member will correctly attach that documentation to the variant within the discriminated union.
- `ts.doc.text-before-tags`: Put every ordinary label/description line and extension block before the first `@Libertas...` tag. Document extraction stops at that tag.
- `ts.doc.code-comments-ignored`: Standard double-slash `//` comments are **completely ignored** by the schema generator. Use them freely for internal code documentation.
- `ts.doc.replace-example-identity` (judgment): Replace the example Plaza identity before publication.
- `ts.doc.param-text-identical`: TypeScript parameter documentation may come from `@param` text or the parameter declaration; keep them semantically identical when both exist.
- `ts.doc.variant-text-consistent`: Union variant text can come from a discriminator member, branch, or payload type and must never conflict across those sources.
- `ts.attr.no-inference-from-schema`: Do not infer an authoring tag from a generated schema attribute or a parser-internal alias.
- `ts.doc.excerpts-need-package-text` (judgment): Unless a block begins with package-level `///` text, its declarations are an excerpt to place inside an entry module that already has the required package label and description. Such declaration excerpts remain copyable when their referenced types are declared and documented elsewhere. A deliberately incomplete grammar or precedence illustration is explicitly labeled **non-copyable**.

### Core Development Concepts

Owner: [typescript.md#core-development-concepts](../reference/languages/typescript.md#core-development-concepts)

- `ts.project.global-types`: **Global Types**: All TypeScript projects must reference `@libertas-ai/types` as global types.
- `ts.source.compile-before-schema`: **Standard TypeScript**: Write standard TypeScript. Your code must compile successfully (`tsc --noEmit` should pass) before the schema can be generated.
- `ts.project.entry-point`: **Entry Point**: The schema generator always starts scanning at `index.ts`.

### Native Global Types

Owner: [typescript.md#native-global-types](../reference/languages/typescript.md#native-global-types)

- `ts.types.native`: When defining properties and function parameters, use the native Libertas types provided by the `@libertas-ai/types` package to ensure your schemas map correctly to the platform's features:
- `ts.types.endpoint`: **`LibertasEndpoint`**: Represents a protocol endpoint for interactions with humans, AI, and other software.
- `ts.types.datetime`: **`LibertasDateTime`**: A timestamp (Unix epoch/seconds). In the UI, this automatically displays in the user's local timezone and parsed back to UTC on save.
- `ts.types.time-only`: **`LibertasTimeOnly`**: Provides a 24-hour time selector in the UI.
- `ts.types.bytes`: **`Uint8Array`**: Represents binary data/bytes, mapping directly to the platform built-in type `"Bytes"`.
- `ts.types.user`: **`LibertasUser`**: Presents a system user or user group picker. Use explicit [User and group defaults](../reference/schema.md#user-and-group-defaults) for initial suggestions; the picker does not infer the current user.
- `ts.types.device`: **`LibertasDevice`**: Presents a physical or logical device picker.
- `ts.types.virtual-device`: **`LibertasVirtualDevice`**: Presents a software-emulated virtual device picker.
- `ts.types.lan-device`: **`LibertasLanDevice`**: Presents a WiFi or Ethernet network device picker.
- `ts.types.mime-types`: For a MIME-aware `Uint8Array` field, write `/** @LibertasMimeTypes("image/png,image/jpeg") */` above the declaration. The parser emits `MimeTypes`
- `ts.types.mime-bytes-value`: A TypeScript App emits the LMB1 byte sequence as its ordinary `Uint8Array` value; an empty `Uint8Array` is the valid media-undefined value. Keep unannotated `Uint8Array` fields for raw binary protocols.

### Matter Status Values

Owner: [typescript.md#matter-status-values](../reference/languages/typescript.md#matter-status-values)

- `ts.matter.status-packing`: The TypeScript SDK represents a Libertas Matter client status as an unsigned 32-bit number:
- `ts.matter.status-layout`: Bits 0-7 contain the Matter interaction-model status, bits 8-15 contain the optional cluster-specific status, and bits 16-31 contain a Libertas Hub status. A zero cluster byte means that no cluster-specific status is present. `LibertasMatterHubStatus.Superseded` is `0x00010000`. It rejects a queued command replaced by a later command, even when their payloads are identical; the retained caller is notified only after the surviving interaction completes. Only the surviving command receives its own success or error result. For writes, it reports a requested attribute value differing from the value finally written by the pooled Matter interaction. See the [Matter status contract](../reference/matter/status.md#command-supersession). This encoding is used by `LibertasIdStatus` and numeric `LibertasCommandRsp` and `LibertasDeviceRsp` values received by a Matter client.
- `ts.matter.virtual-device-status`: A Matter virtual device may return only the standard low 16 bits. Therefore a plain `Matter.Status` value can be passed directly when no cluster status is needed; a virtual device must never return `LibertasMatterHubStatus` values. The TypeScript runtime rejects a virtual-device status with nonzero upper bits.
- `ts.matter.status-decode`: Decode a returned status with `packedStatus & 0xff` for the interaction-model status and `(packedStatus >>> 8) & 0xff` for the cluster status. Decode the Hub field with `packedStatus >>> 16`. See the [Libertas Matter client status contract](../reference/matter/status.md) for supersession, completion timing, and error-override details.

### Interface & Class Composition

Owner: [typescript.md#interface--class-composition](../reference/languages/typescript.md#interface--class-composition)

- `ts.types.inheritance`: You can build reusable models using standard TypeScript `extends` clauses for classes and interfaces. The platform will deeply explore all inheritance chains and automatically resolve all properties from the parent base classes into your component's definition. Properties, attributes, and comments from base interfaces are automatically inherited and displayed as part of the child object in the UI. The child can selectively override any inherited property definition or comment by re-declaring it.

### Enumerations

Owner: [typescript.md#enumerations](../reference/languages/typescript.md#enumerations)

- `ts.enum.const-object-type-alias`: **Const Object Enum**: Use a read-only `as const` object. Because TypeScript separates its value and type spaces, you must also export a corresponding type alias to use the object's values as a type annotation.
- `ts.enum.const-enum-file`: **Const Enum File**: Place a `@LibertasConstEnum` JSDoc tag at the very top of a file. Every exported numeric `const` variable declaration within that file will automatically be parsed as an independent choice for that enumeration. This is particularly useful for aggregating sets of disparate constants into a single logical Enum type.
- `ts.enum.protocol-variant`: An enum member can carry `/** @LibertasProtocolVariant("EditWages") */` to declare an [enum/protocol association](../reference/schema.md#enumeration-protocol-associations). Put the comment on the standard enum member, const-object property, or exported constant statement, respectively.

### Discriminated Unions (Union Types)

Owner: [typescript.md#discriminated-unions-union-types](../reference/languages/typescript.md#discriminated-unions-union-types)

- `ts.union.inline-variants`: When modeling discriminated unions for protocol or data schemas, you can define variants as either separate named interfaces or anonymous inline type literals. The UI handles anonymous inline literals `{ type, payload }` seamlessly, providing the exact same input form as if you had used a named interface, recursively flattening fields directly under the variant.
- `ts.union.discriminator-undocumented`: The parser intentionally suppresses document text for the TypeScript discriminator field itself. This compiler-oriented field is the sole ordinary exception to the all-node documentation invariant. Document the observable variant through its enum member, Union branch, or payload type instead.

### Optional Types

Owner: [typescript.md#optional-types](../reference/languages/typescript.md#optional-types)

- `ts.types.nullable-choice`: Optional modifiers (`?`) and nullable unions (`| null` or `| undefined`) declare meaningful absence. An editor presents an editable one as a two-answer choice labelled by `[NullText]` and `[SetText]`; give it no Default when the person must answer (see [Nullable choices](../reference/app-contract.md#nullable-choices)). For ordinary optional user text, prefer a `string` with an empty Schema Default and size bounds permitting zero, under the shared [type-selection guidance](../reference/app-contract.md#user-facing-and-non-user-facing-types):

  ```typescript
  /**
   * Student details
   * Describes the personal details recorded for one student.
   */
  export interface Student {
      /**
       * Middle name
       * Enter the student's middle name. Leave it empty if absent.
       * @LibertasDefault("")
       * @LibertasSize(min=0, max=120)
       */
      middleName: string;
  }
  ```

- `ts.types.nullable-forms`: When absence must remain distinct from a present value, use `amount?: number` or `amount: number | null` or `amount: number | undefined`, for example to distinguish an unknown amount from known zero. Both forms emit the platform schema's `Nullable` attribute; `[NullText]` states what null means. A Nullable no person answers starts null; one the person answers starts unanswered unless its Default is `$null`.
- `ts.types.size-min-not-default`: A size minimum constrains a supplied String, not null. A non-Nullable String without a Fixed or applicable default starts unanswered, even when its size minimum is zero.
- `ts.types.nullable-alias-reference`: **Special Case**: If you define a property referencing a Union Type Alias with `| undefined`, the parser correctly resolves it as a reference to the type alias with the `Nullable` attribute, rather than flattening the union inline.

### Supported JSDoc Decorators

Owner: [typescript.md#supported-jsdoc-decorators](../reference/languages/typescript.md#supported-jsdoc-decorators)

- `ts.attr.exact-spelling`: Codebots must generate the exact positional or named-property spelling shown in the exhaustive [schema annotation inventory](../reference/annotations.md). Some older named-argument variations remain parser compatibility syntax, but they are not additional public authoring forms. `@LibertasFormattedText` and `@LibertasSingleton` are no-argument markers.

### General Decorators

Owner: [typescript.md#general-decorators](../reference/languages/typescript.md#general-decorators)

- `ts.attr.export-helper`: **`@LibertasExport`**: Parser/compiler helper that does not emit a schema attribute. It does not create a standalone root in an ordinary package and is not needed to root public types in a `TypesOnly` package.
- `ts.attr.default-locale`: **`@LibertasDefaultLocale(locale_name)`**: Package-level decorator (must be in `///` comments) setting the default locale for your package. Defaults to `"en"`.
- `ts.string-resources.package-level`: **`@LibertasStringResources(CONST_NAME, …)`**: Package-level decorator in the leading `///` comments. It resolves one or more non-empty `const` object-literal catalogs, which may sit in modules no runtime module imports, emits the sorted keys of their union on the schema package, and writes its LMF1 templates to the document's top-level `StringResources` object.
- `ts.types-only.marker`: **`@LibertasTypesOnly`**: No-argument package-level marker in the leading `///` comments.
- `ts.attr.default`: **`@LibertasDefault(value)`**: Sets a default value for an input field. Enumeration member names or numeric defaults follow the [shared enumeration contract](../reference/schema.md#53-enumerations). User and User-array defaults use the [identity-selector contract](../reference/schema.md#user-and-group-defaults), for example `@LibertasDefault("current-user")`.
- `ts.attr.fixed`: **`@LibertasFixed(value)`**: Emits one fixed encoded value. On an Endpoint client the value instead identifies the peer task process as `developer-package::function`; the Endpoint selector remains editable, the compatible server set is constrained to that process, and its package is a mandatory dependency. Enum member Fixed values and Union member Discriminator values come from their native declarations; these attributes have independent meanings.
- `ts.presentation.suppress-label-and-display-json`: **`@LibertasSuppressLabel`**
- `ts.presentation.include-description`: **`@LibertasUIIncludeDescription`**: No-argument client presentation marker for a schema data field or type. The UI node displays its localized long description inline together with its short display name. It does not change the typed value or wire encoding and is compiled into the [Display Template](../reference/presentation.md#display-template) as its `include_description` hint.
- `ts.attr.read-only`: **`@LibertasReadOnly`**: Marks the field as read-only in the UI.
- `ts.attr.output-only`: **`@LibertasOutputOnly`**: No-argument marker emitting `OutputOnly=true`
- `ts.attr.hidden`: **`@LibertasHidden`**: Hides the field from the UI entirely.
- `ts.attr.llm-tool-call`: **`@LibertasLLMToolCall(false)` / `@LibertasLLMToolCall(true)`**
- `ts.llm-doc.trans-and-field`: **`@LibertasLLMDocTrans(instruction="...", validation="@.facts", doc_type="image/*", accept_multiple_inputs=true)`**
- `ts.attr.exclude-ui`: **`@LibertasExcludeUI`**
- `ts.number.physical-unit`: **`@LibertasPhysicalUnit("millimeter")`**: Declares a canonical CLDR core unit identifier
- `ts.number.money`: `@LibertasMoney("USD", 2)`
- `ts.number.format`: **`@LibertasFormat("pattern")`**: Display-only `Format`, compiled into the [Display Template](../reference/presentation.md#display-template)
- `ts.foreign-type.decorator`: **`@LibertasForeignType("developer-package::Type")`**: Declares the direct official type
- `ts.exports.external-ownership`: A resolved type outside the current package is automatically foreign and uses its native owning App FQN.

### Cross-language interoperable packages

Owner: [typescript.md#cross-language-interoperable-packages](../reference/languages/typescript.md#cross-language-interoperable-packages)

- `ts.packages.own-developer-name`: Build every package under your own developer name; never use another developer's name as your package namespace. As a best practice, publish public data schemas in a `TypesOnly` package and make its source available.

### Numeric Modifiers

Owner: [typescript.md#numeric-modifiers](../reference/languages/typescript.md#numeric-modifiers)

- `ts.validation.range-attributes`: **`@LibertasRangeMin("expression")` / `@LibertasRangeMax("expression")`**: Each carries one Data Validation bound expression compatible with the host type on a Number or `LibertasDateTime` field/type. See [dynamic numeric ranges](../reference/validation-rules.md#31-dynamic-numeric-ranges) for result typing, compatible dates, and GUI/final enforcement. These expressions have no message key, semicolon pair, or `FinalOnly` directive.
- `ts.number.time-interval`: **`@LibertasTimeInterval`**: In the Task Editor, this triggers a specialized `HH:MM` time interval input for users.
- `ts.number.date-only`: **`@LibertasDateOnly`**: No-argument marker
- `ts.bitflags.forms`: **`@LibertasBitflags` / `@LibertasBitflags(TypeName)`**: The no-argument form projects a TypeScript enum into a localized Enumeration catalog.

### Bitflag catalogs and integer fields

Owner: [typescript.md#bitflag-catalogs-and-integer-fields](../reference/languages/typescript.md#bitflag-catalogs-and-integer-fields)

- `ts.bitflags.declaration`: Use a TypeScript `enum` as the language-equivalent bitflag declaration. Put the no-argument marker on the enum and name it from the integer field marker. The enum may remain private because the parser fabricates schema metadata from it; the enum itself is not the runtime or wire value.

### Strings and Arrays

Owner: [typescript.md#strings-and-arrays](../reference/languages/typescript.md#strings-and-arrays)

- `ts.enum-attr.compatibility-only`: **`@LibertasEnum(A, B, C)`**: Parser-supported compatibility syntax that fabricates positional string choices. Those fabricated members have no source location where a codebot can provide each member's required natural label and description. Codebots must therefore prefer a documented native numeric enum and must not generate this positional form until Libertas defines a member-document syntax for it. Existing hand-authored uses remain parseable.
- `ts.size.decorator`: **`@LibertasSize(min?, max?)`**
- `ts.array.unordered`: **`@LibertasUnordered`**: Marks an array as an unordered set, affecting how arrays are merged or displayed.
- `ts.array.unique`: **`@LibertasUnique(level?: number)`**
- `ts.array.key`: **`@LibertasKey`**: No-argument marker emitting `Key=true`

### Union Types & Discriminators

Owner: [typescript.md#union-types--discriminators](../reference/languages/typescript.md#union-types--discriminators)

- `ts.union.discriminators`: The parser emits numeric `Discriminator` on each Union member, sorted by that value.

### Union Variant Documentation

Owner: [typescript.md#union-variant-documentation](../reference/languages/typescript.md#union-variant-documentation)

- `ts.doc.variant-precedence`: When you represent complex states, commands, or data variants using TypeScript discriminated unions, Libertas aggregates your documentation to provide a clean user experience using a strict 3-tier precedence hierarchy.

### 1. Document Precedence

Owner: [typescript.md#1-document-precedence](../reference/languages/typescript.md#1-document-precedence)

- `ts.doc.precedence-discriminator`:

  | Priority | Source Layer | Description |
  | :--- | :--- | :--- |
  | **Priority 1 (Highest)** | **Corresponding Discriminator Value** | JSDoc comments preceding the resolved `enum` member or `const` property declaration corresponding to the variant's discriminator value. |

- `ts.doc.precedence-variant-node`:

  | Priority | Source Layer | Description |
  | :--- | :--- | :--- |
  | **Priority 2 (Middle)** | **Union Variant Node** | JSDoc comments preceding the variant type reference directly within the union type definition. |

- `ts.doc.precedence-variant-type`:

  | Priority | Source Layer | Description |
  | :--- | :--- | :--- |
  | **Priority 3 (Lowest)** | **Variant Class/Interface** | JSDoc comments preceding the base interface, class, or type declaration of the variant itself. |

### Non-copyable precedence schematic

Owner: [typescript.md#non-copyable-precedence-schematic](../reference/languages/typescript.md#non-copyable-precedence-schematic)

- `ts.examples.schematic-not-copyable`: The following fragment deliberately references omitted declarations and uses precedence-layer labels as placeholder document text. It exists only to show which of three comments wins. Do not copy it into an App; real declarations must use one consistent, natural label and description for every shown node.

### 2. Attribute and Decorator Merging

Owner: [typescript.md#2-attribute-and-decorator-merging](../reference/languages/typescript.md#2-attribute-and-decorator-merging)

- `ts.doc.attribute-merging`: Unlike the Short Name and Description (which follow strict overriding precedence), **custom JSDoc decorators and attributes are merged cumulatively** across all three layers. All resolved attributes are combined, eliminating duplicates, to form the final variant metadata schema.

### Data-Backed Enum Indexes

Owner: [typescript.md#data-backed-enum-indexes](../reference/languages/typescript.md#data-backed-enum-indexes)

- `ts.enum-source.decorator`: Use `@LibertasEnumSource("path")`

### Formatted Runtime Text

Owner: [typescript.md#formatted-runtime-text](../reference/languages/typescript.md#formatted-runtime-text)

- `ts.formatted-text.marker`: Use the no-argument `@LibertasFormattedText` marker
- `ts.formatted-text.bytes`: The byte blob is the canonical resource key plus ordered, typed `LibertasMessageArgument` values, including an empty argument list. Localized templates use zero-based LMF1, and runtime payloads never contain resource templates.
- `ts.formatted-text.no-base64-string`: Do not use a `string` host containing Base64. Rendering would only decode that String back to the same byte tuple, adding a pointless encode/decode cycle and providing no benefit. Keeping formatted values as Bytes creates a sharp type distinction from literal text, making developer mistakes less likely. App code keeps the value as `Uint8Array`; if a generic JSON transport represents Bytes as Base64, that is only the transport codec and never the field's semantic type.
- `ts.formatted-text.sdk-function`: The public SDK function `libertasFormattedText(resourceName, arguments)` returns the canonical `Uint8Array`. App code ordinarily does not decode it; clients resolve and render the LMF1 template at the presentation boundary.

### Protocol chart presentation

Owner: [typescript.md#protocol-chart-presentation](../reference/languages/typescript.md#protocol-chart-presentation)

- `ts.chart.decorators`: Chart metadata is part of the ordinary endpoint protocol schema. It adds no payload bytes or chart-specific message. Apply `@LibertasChart(mark)`

### 3D chart source attributes

Owner: [typescript.md#3d-chart-source-attributes](../reference/languages/typescript.md#3d-chart-source-attributes)

- `ts.chart3d.decorators`: Three-dimensional presentation uses five separate decorators: `@LibertasChart3D`, `@LibertasChart3DChannel`, `@LibertasChart3DScale`, `@LibertasChart3DGuide`, and `@LibertasChart3DView`. They add only schema metadata and do not alter the typed response payload or the existing 2D decorators.
- `ts.chart3d.family-tokens`: Chart-family names are not extra tokens: scatter and point clouds use `point`, bubbles add `size`, traces and waterfalls use `line`, columns use `bar`, height maps and sampled mathematical grids use `surface`, contours layer `line` over `surface`, and impulses use `stem`. Use `detail` for homogeneous multiple series and `layer` for heterogeneous series.

### Endpoint Protocols

Owner: [typescript.md#endpoint-protocols](../reference/languages/typescript.md#endpoint-protocols)

- `ts.endpoint.schema`: **`@LibertasEndpointSchema(MySchemaUnion)`**
- `ts.endpoint.server`: **`@LibertasEndpointServer`**: Marks the endpoint as a server.
- `ts.endpoint.default-process`: **`@LibertasDefault("developer-package::function")`** on an Endpoint client: when no schema-compatible server Endpoint exists, recommends provisioning the named task process. It does not filter existing compatible choices. If this Endpoint is absolutely required by its ancestor path, the client blocks task creation and requires that process to be installed when no compatible server exists.
- `ts.endpoint.fixed-process`: **`@LibertasFixed("developer-package::function")`** on an Endpoint client:
- `ts.conformance.decorator`: **`@LibertasProtocolConformance("VariantA & (VariantB | !VariantC)")`**
- `ts.endpoint.base-objects`: **`@LibertasEndpointBaseObjects("$.device,$.room")`**: Binds the Endpoint instance to the configuration objects named by one positional quoted comma-delimited path string; the Hub derives the Endpoint's displayed name and icon from them. Every path must resolve to set values, and task-value validation rejects two Endpoint instances in one task that resolve to exactly the same base-object set; see [path-based object binding](../reference/schema.md#7-path-based-object-and-choice-binding).

### Protocol Roles & Recursive Variant Validation

Owner: [typescript.md#protocol-roles--recursive-variant-validation](../reference/languages/typescript.md#protocol-roles--recursive-variant-validation)

- `ts.protocol.error-marker.authoring`: Declare `@LibertasError` beside `@LibertasResponse` for a typed error result; [out-of-band error handling](../reference/protocols.md#protocol-roles) preserves the current interaction.
- `ts.protocol.error-field-path`: the no-argument `@LibertasErrorFieldPath` to emit `ErrorFieldPath=true`.
- `ts.protocol.reveal-path`: the no-argument `@LibertasRevealPath` to emit `RevealPath=true`, and a Request or Response payload may mark one `@LibertasHidden` `string` field with the no-argument `@LibertasLandingPath` to emit `LandingPath=true`.
- `ts.protocol.access-privilege`: Use `@LibertasAccessPrivilege("Read")`, `@LibertasAccessPrivilege("Write")`, or `@LibertasAccessPrivilege("Read,Write")`
- `ts.workflow.request-and-prev`: `@LibertasWorkflowRequest("EditProfile")` declares the [WorkflowRequest semi-role](../reference/protocols.md#workflowrequest-semi-role).
- `ts.link.decorator`: `@LibertasLink("OpenTopic")`
- `ts.link.link-only`: `@LibertasLinkOnly`, with no argument,
- `ts.protocol.request-confirmation`: `@LibertasRequestConfirmation`, with no argument,

### Advanced Chaining & Caching

Owner: [typescript.md#advanced-chaining--caching](../reference/languages/typescript.md#advanced-chaining--caching)

- `ts.protocol.next-request-response`: **`@LibertasNextRequest("...")`** / **`@LibertasNextResponse("...")`**

### State Management

Owner: [typescript.md#state-management](../reference/languages/typescript.md#state-management)

- `ts.calc.calculation-request`: **`@LibertasCalculationRequest("CalculatePrice,CalculateShipping")`**
- `ts.calc.calculation-root`: **`@LibertasCalculationRoot("CalculatePrice")`**
- `ts.calc.copy-to`: **`@LibertasCopyTo("@.orderTotal")`**
- `ts.calc.copy-from`: **`@LibertasCopyFrom("$.first,$.fallback")`**

### Semantic validation rules

Owner: [typescript.md#semantic-validation-rules](../reference/languages/typescript.md#semantic-validation-rules)

- `ts.validation.rules-decorator`: Use `@LibertasValidationRules("...")`

### Testing validation rules

Owner: [typescript.md#testing-validation-rules](../reference/languages/typescript.md#testing-validation-rules)

- `ts.testing.test-rules` (judgment): Keep TypeScript compilation and application tests alongside schema validation. For client-rule fixtures in CI, use Studio's installed schema compiler launcher with `test-rules`, the same command used by the Rust SDK's `examples/validation-rules` sample. Fixtures use the schema's JSON value form regardless of source language. See [Debugging rules](../reference/validation-rules.md#debugging-rules) for the fixture format, exact expectations, report, exit status and launcher command. It shares the compile path and evaluator with `evaluate_validation_rules`.

### Endpoint Signaling and Liveness (non-normative summary)

Owner: [typescript.md#endpoint-signaling-and-liveness-non-normative-summary](../reference/languages/typescript.md#endpoint-signaling-and-liveness-non-normative-summary)

- `ts.endpoint.signaling`: Use the [endpoint signaling orientation](../reference/endpoint-lifecycle.md) and the installed `@libertas-ai/types` endpoint declarations before implementing callbacks. The public SDK owns the complete contract. Its TypeScript liveness entry point is `libertasEndpointPeerAlive(endpoint, peer?)`.

### Standard Data Schemas

Owner: [typescript.md#standard-data-schemas](../reference/languages/typescript.md#standard-data-schemas)

- `ts.data.context`: `libertasData<Keys, Values>()` supplies the synchronous schema-bound context; `data.async()` supplies the Promise API. Both accept exact sorted-key targets and prefixes. The [transactional API](../reference/protocols.md#transactional-key-value-api) owns the primitive inventory, ownership rules, scan continuations, and limits.
- `ts.data.error-status-decorator`: Place `@LibertasDataErrorStatus` in the public task function's JSDoc
- `ts.data.checked-calls`: They return `DataResult<T>` with a numeric `DataStatus`; only `Success` carries `value`. Ordinary Data methods retain default failure behavior, even on opted-in tasks.
- `ts.data.schema-decorator`: `@LibertasDataSchema(key=DataKey,value=StoredData)`.
- `ts.data.key-sizes`: Key String and Uint8Array fields reuse `@LibertasSize(min=...,max=...)` with byte bounds. A maximum alone selects a bounded payload; explicit equal bounds select an exact-length payload. Numeric fields reuse `@LibertasNumber(type=...)`. Optional fields use the existing `Nullable` mapping. See the [shared storage schema rules](../reference/protocols.md#storage-schema-declarations).

### Singleton tasks

Owner: [typescript.md#singleton-tasks](../reference/languages/typescript.md#singleton-tasks)

- `ts.singleton.decorator`: Use `@LibertasSingleton` on an exported function
- `ts.singleton.identity`: The singleton identity is the App plus the function's native name; it is not a package-wide restriction and it does not reset between App versions. Studio disables a singleton function when another matching task is already present, and the Hub authoritatively checks every create, retarget, and task-version change. A task may update itself. If two selected versions disagree about the marker, the function remains singleton when either version declares it.

### Permissions

Owner: [typescript.md#permissions](../reference/languages/typescript.md#permissions)

- `ts.permissions.endpoint-server`: The same decorator may be placed on a built-in Endpoint property marked `@LibertasEndpointServer`.

### Network host access

Owner: [typescript.md#network-host-access](../reference/languages/typescript.md#network-host-access)

- `ts.access-host.function`: Use `@LibertasAccessHost("host1,host2")` on a function to declare the DNS hostnames its task process needs to contact.

### String Resources

Owner: [typescript.md#string-resources](../reference/languages/typescript.md#string-resources)

- `ts.string-resources.one-catalog`: Declare one localized catalog for the whole package. Notifications, permission and host-access reasons, data names, `FormattedText`, and formatted chart labels from every function use the same stable package-local keys. The task already identifies the App package, so encoded tuples do not carry a function/type owner or FQN.

### Folding and inline choices

Owner: [typescript.md#folding-and-inline-choices](../reference/languages/typescript.md#folding-and-inline-choices)

- `ts.presentation.foldable`: `@LibertasFoldable` permits manual folding on a struct, union, or list type or field; `@LibertasFoldable(initially_open=false)` starts it collapsed.
- `ts.presentation.inline-choices`: `@LibertasInlineChoices` on an enum or union declaration, or on a field of either type,

### Backend-constrained choices

Owner: [typescript.md#backend-constrained-choices](../reference/languages/typescript.md#backend-constrained-choices)

- `ts.choices.constrained-by`: `@LibertasConstrainedBy("$.allowed_choices")`
- `ts.protocol.default-request`: Default endpoint entry uses `@LibertasDefaultRequest`
- `ts.protocol.action-control`: For additional response controls, put `@LibertasActionControl("sections")` and `@LibertasHidden`
- `ts.workflow.cache-identity`: For large workflow request payloads, `@LibertasCacheIdentity`

### AI Search guidance

Owner: [typescript.md#ai-search-guidance](../reference/languages/typescript.md#ai-search-guidance)

- `ts.ai.navigation-prompt`: Use `@LibertasAINavigationPrompt("NAVIGATION_PROMPT")`

### ValidationRules literal layer

Owner: [validation-rules.md#validationrules-literal-layer](../reference/validation-rules.md#validationrules-literal-layer)

- `validation.source.ts-annotation-escaping`: For the old TypeScript annotation, an apostrophe inside a double-quoted rule String must also survive the single-quoted annotation envelope. For example:

  ```typescript
  /** @LibertasValidationRules('matches([], "[\"\']+") : "QUOTES";') */
  ```

- `validation.source.annotation-versus-program-string`: The outer annotation preserves both backslash/quote sequences; the rule lexer then decodes them to regex `["']+`. Ordinary TypeScript program constants instead use native TypeScript string decoding. Do not confuse a JSDoc annotation envelope with a program String literal.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Code examples

Owner: [typescript.md#code-examples](../reference/languages/typescript.md#code-examples)

- `ts.enum.const-enum-file-layout`: A const-enum file declares one exported numeric member per statement; the entry module imports it as a namespace and exports a documented `typeof` alias

### Discriminated Unions (Union Types)

Owner: [typescript.md#discriminated-unions-union-types](../reference/languages/typescript.md#discriminated-unions-union-types)

- `ts.union.numeric-first-discriminator`: The discriminator is the variant's first field and numeric; a string discriminator is rejected

### General Decorators

Owner: [typescript.md#general-decorators](../reference/languages/typescript.md#general-decorators)

- `ts.attr.ignored`: `@LibertasIgnored` on an exported function or named type omits it as a root; a reachable ignored type fails
- `ts.exports.exact-name`: Every published named type is exported from the root `index.ts` under its exact declaration name; renamed exports are rejected
- `ts.scope.native-resolution`: Type names resolve only in their declaration's TypeScript scope; an unbound name is a parser error

### Numeric Modifiers

Owner: [typescript.md#numeric-modifiers](../reference/languages/typescript.md#numeric-modifiers)

- `ts.number.attribute`: TypeScript Number syntax requires explicit type=, fixed step and numeric or quoted expression bounds
- `ts.number.removed-spellings`: The former Integer/Float source spellings are removed

### Bitflag catalogs and integer fields

Owner: [typescript.md#bitflag-catalogs-and-integer-fields](../reference/languages/typescript.md#bitflag-catalogs-and-integer-fields)

- `ts.bitflags.external-catalog`: An external bitflag catalog may stay private but is declared under its exact name at the owning App's root `index.ts`

### Strings and Arrays

Owner: [typescript.md#strings-and-arrays](../reference/languages/typescript.md#strings-and-arrays)

- `ts.pattern.decorator`: TypeScript Pattern syntax takes one JSON String or a resolvable native String constant

### Permissions

Owner: [typescript.md#permissions](../reference/languages/typescript.md#permissions)

- `ts.permissions.function`: TypeScript Permissions syntax resolves one const string-array literal

### String Resources

Owner: [typescript.md#string-resources](../reference/languages/typescript.md#string-resources)

- `ts.string-resources.types-only`: A `TypesOnly` package's catalog is one non-empty `const` object literal with unique non-empty keys and string values

<!-- /generated -->
