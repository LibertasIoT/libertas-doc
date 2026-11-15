# Organizing App code

Load this card when a task creates an App, adds or moves modules, changes
exports, or reorganizes source files, in Rust or TypeScript. Its rules are
generated from the [App code organization practice](../reference/code-organization.md),
which owns them; the language rules themselves are in the
[Rust App rules](rust_app.md) and [TypeScript App rules](typescript_app.md)
cards, and the platform requirements in the [App contract](app_contract.md).

## Order of work

1. Keep the responsibilities in the fewest modules the App needs and split
   only by responsibility (`org.modules.*`, `org.operations.*`,
   `org.contracts.*`).
2. Keep dependencies and types easy to follow (`org.dependencies.*`,
   `org.types.*`, `org.rules.*`, `org.runtime.*`).
3. Apply the chosen language's layout: entry file, exports, imports and tests
   (`org.rust.*` or `org.ts.*`).
4. Make the change through its owners and, for a file-only reorganization,
   keep every published identity unchanged (`org.change.*`).
5. Review with the checklist before handing off (`org.review.*`).

<!-- generated:codebot-rules task=code-organization -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### App code organization

Owner: [code-organization.md#app-code-organization](../reference/code-organization.md#app-code-organization)

- `org.modules.fewest` (judgment): Preserve working conventions and start with the fewest modules the App needs.
- `org.operations.one-implementation` (judgment): Keep one implementation of each operation and let the platform derive its interfaces under the [shared interaction model](../reference/app-contract.md#application-model).

### Choose boundaries by responsibility

Owner: [code-organization.md#choose-boundaries-by-responsibility](../reference/code-organization.md#choose-boundaries-by-responsibility)

- `org.modules.split-by-responsibility` (judgment): Split a module when its responsibility becomes difficult to navigate or can be understood and tested independently; file length alone does not require another layer.
- `org.contracts.union-in-one-place` (judgment):

  | Owner | Contains | Boundary |
  | --- | --- | --- |
  | Protocol and stored contracts | Each Endpoint protocol Union and persistent-data Union, referring to feature payload types | Keep each published Union's declaration and append-only order in one obvious place. |

- `org.modules.shared-helper-named` (judgment): Move a helper into a specifically named shared module when multiple features need the same meaning; a general `utils` module obscures ownership.
- `org.modules.adapter-for-real-boundary` (judgment): Add an interface or adapter for a real implementation boundary or test substitute. Splitting files alone does not justify repository, service, controller or mapping abstractions around an existing SDK API.

### Keep dependencies easy to follow

Owner: [code-organization.md#keep-dependencies-easy-to-follow](../reference/code-organization.md#keep-dependencies-easy-to-follow)

- `org.dependencies.import-from-definer` (judgment): Internal modules import from the defining module instead of routing their dependencies through the App's public entry file.
- `org.types.reuse-by-meaning` (judgment): Reuse a type when its meaning and constraints are the same. Use distinct input, output or stored types when their actual semantics differ; follow [type selection by occurrence](../reference/app-contract.md#user-facing-and-non-user-facing-types).
- `org.types.no-copy-for-file-boundary` (judgment): A file boundary is not a reason to copy the same record into another shape.
- `org.runtime.owns-state` (judgment): The task runtime owns shared state and identifies per-user or per-peer state explicitly. Handlers receive the state and identity needed for their work.
- `org.rules.with-owning-operation` (judgment): Keep workflow decisions and domain checks with the operation that owns them; all callers reach that same behavior.
- `org.runtime.sdk-lifetimes` (judgment): SDK callbacks, subscriptions and timers retain their documented ownership and lifetime. Use normal startup and the [Data API responsibility boundary](../reference/protocols.md#persistence-failure-and-the-no-status-api) for persistence failures.

### Rust layout

Owner: [code-organization.md#rust-layout](../reference/code-organization.md#rust-layout)

- `org.rust.delegate-runtime` (judgment): Delegate runtime work as it grows, keeping implementation functions private or `pub(crate)` as appropriate.
- `org.rust.no-reexport-declarations`: Function re-exports do not replace entry-file task declarations.
- `org.rust.explicit-pub-use` (judgment): Use explicit `pub use` declarations so a change cannot silently enlarge the published surface.
- `org.rust.test-placement` (judgment): Place focused unit tests beside the rules they test under `#[cfg(test)]`; use `tests/` for tests through the library's public API.
- `org.rust.module-file-style` (judgment): Preserve an existing choice of module files or `mod.rs`; either follows native Rust resolution.
- `org.rust.separate-crate` (judgment): Create a separate crate or workspace member when it is a real reusable package, not merely to shorten a file. A new Libertas package introduces ownership and deployment dependencies under the [App dependency contract](../reference/app-contract.md#external-types-and-app-dependencies).

### TypeScript layout

Owner: [code-organization.md#typescript-layout](../reference/code-organization.md#typescript-layout)

- `org.ts.esm-project`: Organize it as an ESM project using the selected Studio entry module and the [strict configuration baseline](../reference/languages/typescript.md#project-configuration-baseline). For that NodeNext baseline, set `"type": "module"` in `package.json`.
- `org.ts.entry-functions`: Declare documented exported task functions in the selected entry module, with configuration arguments and function annotations. Delegate runtime work as it grows.
- `org.ts.helpers-from-definer` (judgment): Export helpers from their defining module when other implementation files need them, keeping them out of the App entry's exports.
- `org.ts.reexport-not-task-declaration`: Function re-exports do not replace entry-file task declarations.
- `org.ts.type-only-exports` (judgment): Prefer explicit type-only imports/exports for interfaces and type aliases; export an enum or other value normally when runtime code needs its value. These distinctions follow the [TypeScript module reference](https://www.typescriptlang.org/docs/handbook/modules/reference.html#type-only-imports-and-exports).
- `org.ts.direct-imports` (judgment): Use direct imports between implementation modules and avoid chains of `export *` files.
- `org.ts.module-isolation` (judgment): A source file needing only module isolation can use `export {}`; keep ordinary App implementation out of the global namespace.
- `org.ts.test-placement` (judgment): Tests may sit beside the implementation or in `tests/`, provided they are not App entry exports or runtime dependencies.
- `org.ts.project-references` (judgment): Introduce [TypeScript project references](https://www.typescriptlang.org/docs/handbook/project-references.html) only when multiple real projects need separate builds or dependency boundaries; ordinary modules are enough for one App.
- `org.ts.native-structure-not-ownership` (judgment): Native project structure does not replace Libertas package ownership or Studio's dependency resolution.

### Make a change through its owners

Owner: [code-organization.md#make-a-change-through-its-owners](../reference/code-organization.md#make-a-change-through-its-owners)

- `org.change.domain-rule-and-handler` (judgment): Change the feature's domain rule and handler. Keep persistence or device effects in the module that performs them, and use that behavior for human, AI and software invocation.
- `org.change.single-union`: Update the single protocol/data Union when a new variant is required. Preserve its name, existing cases and order under [data-type evolution](../reference/data-type-evolution.md).
- `org.change.exports-and-docs-together` (judgment): Update exact-name root exports and localized source documentation together.
- `org.change.presentation-only-when-needed`: Update presentation source only when the same data needs a different arrangement. Generated schema, localized documents, compiled templates and deployment bindings remain generated under the [authoring contract](../reference/app-contract.md).
- `org.change.meaningful-examples` (judgment): Check the affected behavior with meaningful examples. Cover relevant state transitions, invalid input, identity/access, conflicting updates and external effects; use the App's requirements to select the cases.
- `org.change.file-only-reorganization` (judgment): For a file-only reorganization, keep task entry names, published type identities, Union order, field shapes, annotations and localized meaning unchanged. Run the native checker and Studio validation, and review the generated schema, document and Display Template for unintended changes. Follow the [validation and handoff workflow](delivery.md).

### Review checklist

Owner: [code-organization.md#review-checklist](../reference/code-organization.md#review-checklist)

- `org.review.findable` (judgment): Can a CodeBot find the entry function, each contract and the owning handler without tracing generic utility or export chains?
- `org.review.one-owner` (judgment): Does each domain rule and effect have one owner, shared by all participants?
- `org.review.intentional-exports` (judgment): Are root exports intentional and all schema-visible declarations documented?
- `org.review.union-order` (judgment): Are protocol and stored Union identities and order preserved?
- `org.review.minimal-dependencies` (judgment): Does each module depend only on what its responsibility needs?
- `org.review.justified-structure` (judgment): Are extra folders, wrappers and packages justified by real behavior?
- `org.review.checks-pass` (judgment): Do native checks, Studio validation and relevant behavior tests pass?

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### TypeScript layout

Owner: [code-organization.md#typescript-layout](../reference/code-organization.md#typescript-layout)

- `org.ts.resolvable-imports`: Imports resolve under the project's ESM configuration; NodeNext relative specifiers use the emitted `.js` paths

<!-- /generated -->
