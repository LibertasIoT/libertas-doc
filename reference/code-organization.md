# App code organization

This Rust and TypeScript practice supports Libertas's
[collaboration goal](../principles.md#collaboration-goal). Adapt the example
directory names to the project. <!-- rule: org.modules.fewest class=judgment enforced=no tasks=code-organization --> Preserve working conventions and start with
the fewest modules the App needs. <!-- /rule --> The [App contract](app-contract.md) and
language manuals own the platform requirements.

**Organize around data interaction.** Domain data, operations and behavior form
one App model for people, AI and other Apps. Its data also supplies the GUI. <!-- rule: org.operations.one-implementation class=judgment enforced=no tasks=code-organization -->
Keep one implementation of each operation and let the platform derive its
interfaces under the [shared interaction model](app-contract.md#application-model).

## Choose boundaries by responsibility

Keep code that changes for the same reason together. A small App may put all of
these responsibilities in its entry file. <!-- rule: org.modules.split-by-responsibility class=judgment enforced=no tasks=code-organization --> Split a module when its responsibility
becomes difficult to navigate or can be understood and tested independently;
file length alone does not require another layer.

| Owner | Contains | Boundary |
| --- | --- | --- |
| Package entry | Package documentation/directives, explicit public type exports, documented task entry functions | Make the published surface easy to review; delegate runtime work as it grows. |
| Domain feature | Its values, rules, calculations and interaction handlers | Name it for the domain, such as catalog or scheduling. Keep feature-specific helpers here. |
| <!-- rule: org.contracts.union-in-one-place class=judgment enforced=no tasks=code-organization --> Protocol and stored contracts | Each Endpoint protocol Union and persistent-data Union, referring to feature payload types | Keep each published Union's declaration and append-only order in one obvious place. |
| Task runtime | Startup, task state, SDK callbacks, dispatch and shutdown | Own runtime lifetimes and pass the context handlers need. |
| SDK access | The actual persistence, device or external-service calls and App-specific consistency work | Extract a small module when it helps; use SDK types directly. |
| Presentation source | Source metadata beside the value it describes; optional project-root `display.json` | Arrange the same data; reuse generated controls and navigation. |
| Tests | Domain examples and interaction outcomes | Exercise behavior and contracts, using synthetic fixtures. |

A feature can grow from one file into `types`, `rules` and `interaction` modules. <!-- rule: org.modules.shared-helper-named class=judgment enforced=no tasks=code-organization -->
Move a helper into a specifically named shared module when multiple features
need the same meaning; a general `utils` module obscures ownership. <!-- rule: org.modules.adapter-for-real-boundary class=judgment enforced=no tasks=code-organization --> Add an
interface or adapter for a real implementation boundary or test substitute.
Splitting files alone does not justify repository, service, controller or mapping
abstractions around an existing SDK API.

## Keep dependencies easy to follow

When these responsibilities have separate modules, the entry composes the
runtime and the runtime dispatches to feature handlers.
Handlers use domain rules and the SDK operations they need. Rules work on typed
values and remain independent of callbacks, storage handles and presentation
where their behavior permits it. Protocol and stored contracts refer to the
domain payload types. <!-- rule: org.dependencies.import-from-definer class=judgment enforced=no tasks=code-organization --> Internal modules import from the defining module instead
of routing their dependencies through the App's public entry file.

<!-- rule: org.types.reuse-by-meaning class=judgment enforced=no tasks=code-organization -->
Reuse a type when its meaning and constraints are the same. Use distinct input,
output or stored types when their actual semantics differ; follow
[type selection by occurrence](app-contract.md#user-facing-and-non-user-facing-types). <!-- rule: org.types.no-copy-for-file-boundary class=judgment enforced=no tasks=code-organization -->
A file boundary is not a reason to copy the same record into another shape.

<!-- rule: org.runtime.owns-state class=judgment enforced=no tasks=code-organization -->
The task runtime owns shared state and identifies per-user or per-peer state
explicitly. Handlers receive the state and identity needed for their work. <!-- rule: org.rules.with-owning-operation class=judgment enforced=no tasks=code-organization -->
Keep workflow decisions and domain checks with the operation that owns them;
all callers reach that same behavior. <!-- rule: org.runtime.sdk-lifetimes class=judgment enforced=no tasks=code-organization --> SDK callbacks, subscriptions and timers
retain their documented ownership and lifetime. Use normal startup and the
[Data API responsibility boundary](protocols.md#persistence-failure-and-the-no-status-api)
for persistence failures.

## Rust layout

Rust's ordinary module files and explicit re-exports let implementation files
move while the library's public API stays clear. The following adapts the
[Rust module conventions](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html)
and [Cargo package layout](https://doc.rust-lang.org/cargo/guide/project-layout.html)
to a Libertas library:

```text
Cargo.toml
display.json                 optional presentation source
src/
  lib.rs                     package text, public exports, task entry functions
  configuration.rs           task arguments and configuration types
  protocol.rs                stable protocol Unions
  data.rs                    stable stored-data Unions and records
  runtime.rs                 task lifecycle and dispatch
  catalog.rs                 feature module declarations and local exports
  catalog/
    types.rs                 feature values and protocol payloads
    rules.rs                 domain checks and calculations
    interaction.rs           operation handlers
  storage.rs                 App's Data API calls, when useful
tests/
  interactions.rs            public behavior scenarios
  fixtures/                  synthetic test data
```

Keep the library and documented public task functions in `src/lib.rs`, with
configuration arguments and [function attributes](app-contract.md#project-and-discovery-rules). <!-- rule: org.rust.delegate-runtime class=judgment enforced=no tasks=code-organization --> Delegate runtime work as it
grows, keeping implementation functions private or `pub(crate)` as appropriate. <!-- rule: org.rust.no-reexport-declarations class=hard enforced=no tasks=code-organization -->
Function re-exports do not replace entry-file task declarations.

Every published named schema type must remain an
[exact-name root export](languages/rust.md#schema-discovery-and-shape-rules),
including nested payload types reached through the protocol or data schema. <!-- rule: org.rust.explicit-pub-use class=judgment enforced=no tasks=code-organization -->
Use explicit `pub use` declarations so a change cannot silently enlarge the
published surface. <!-- /rule --> [`TypesOnly` packages](app-contract.md#project-and-discovery-rules) require explicit type exports and have
no executable task entries. <!-- /rule --> Follow the [Rust discovery and scope rules](languages/rust.md#project-contract-and-library-requirements).

An entry-file excerpt showing module wiring only:

```rust
mod configuration;
mod protocol;
mod data;
mod runtime;
mod catalog;
mod storage;

pub use configuration::Configuration;
pub use protocol::Protocol;
pub use data::StoredData;
pub use catalog::{Item, SaveItem};
```

The named declarations must exist and be [documented](documentation.md) in their owning modules;
`catalog.rs` explicitly re-exports its public types. Add every other reachable
named type to the [root exports](languages/rust.md#schema-discovery-and-shape-rules). <!-- /rule --> This wiring excerpt omits package text and the
task function; it is not a complete App.

<!-- rule: org.rust.test-placement class=judgment enforced=no tasks=code-organization -->
Place focused unit tests beside the rules they test under `#[cfg(test)]`;
use `tests/` for tests through the library's public API. <!-- /rule --> This follows
[Rust test organization](https://doc.rust-lang.org/book/ch11-03-test-organization.html). <!-- rule: org.rust.module-file-style class=judgment enforced=no tasks=code-organization -->
Preserve an existing choice of module files or `mod.rs`; either follows native
Rust resolution. <!-- rule: org.rust.separate-crate class=judgment enforced=no tasks=code-organization --> Create a separate crate or workspace member when it is a real
reusable package, not merely to shorten a file. A new Libertas package introduces
ownership and deployment dependencies under the [App dependency contract](app-contract.md#external-types-and-app-dependencies).

## TypeScript layout

**TypeScript is under development and not release ready.** Check the
[development status and matching SDK prerequisite](languages/typescript.md#development-status)
before choosing this path. <!-- rule: org.ts.esm-project class=hard enforced=no tasks=code-organization --> Organize it as an ESM project using the selected
Studio entry module and the [strict configuration baseline](languages/typescript.md#project-configuration-baseline).
For that NodeNext baseline, set `"type": "module"` in `package.json`.

```text
package.json
tsconfig.json
display.json                 optional presentation source
src/
  index.ts                   package text, public exports, task entry functions
  configuration.ts           task arguments and configuration types
  protocol.ts                stable protocol Unions
  data.ts                    stable stored-data Unions and records
  runtime.ts                 task lifecycle and dispatch
  catalog/
    types.ts                 feature values and protocol payloads
    rules.ts                 domain checks and calculations
    interaction.ts           operation handlers
  storage.ts                 App's Data API calls, when useful
tests/
  interactions.test.ts       behavior scenarios using the project's runner
  fixtures/                  synthetic test data
```

<!-- rule: org.ts.entry-functions class=hard enforced=no tasks=code-organization -->
Declare documented exported task functions in the selected entry module, with
configuration arguments and function annotations. Delegate runtime work as it
grows. <!-- rule: org.ts.helpers-from-definer class=judgment enforced=no tasks=code-organization --> Export helpers from their defining module when other implementation
files need them, keeping them out of the App entry's exports. <!-- /rule --> <!-- rule: org.ts.reexport-not-task-declaration class=hard enforced=no tasks=code-organization --> Function re-exports
do not replace entry-file task declarations.

Use [named, exact-name exports](languages/typescript.md#general-decorators) for every published schema type. <!-- rule: org.ts.type-only-exports class=judgment enforced=no tasks=code-organization --> Prefer explicit
type-only imports/exports for interfaces and type aliases; export an enum or
other value normally when runtime code needs its value. These distinctions
follow the [TypeScript module reference](https://www.typescriptlang.org/docs/handbook/modules/reference.html#type-only-imports-and-exports). <!-- rule: org.ts.resolvable-imports class=hard enforced=yes tasks=code-organization title="Imports resolve under the project's ESM configuration; NodeNext relative specifiers use the emitted `.js` paths" -->
Keep imports resolvable under the project's ESM configuration. For the NodeNext
baseline, relative specifiers use the emitted `.js` paths that TypeScript
resolves to source `.ts` files.

An entry-file excerpt showing type exports only:

```typescript
export type { Configuration } from "./configuration.js";
export type { Protocol } from "./protocol.js";
export type { StoredData } from "./data.js";
export type { Item, SaveItem } from "./catalog/types.js";
```

The excerpt assumes the named declarations are interfaces or type aliases and
are documented in their defining files. It omits the package text, task function
and runtime imports. Add every other reachable named type to the entry exports. <!-- rule: org.ts.direct-imports class=judgment enforced=no tasks=code-organization -->
Use direct imports between implementation modules and avoid chains of
`export *` files. <!-- rule: org.ts.module-isolation class=judgment enforced=no tasks=code-organization --> A source file needing only module isolation can use `export {}`;
keep ordinary App implementation out of the global namespace.

Use the project's existing test runner. <!-- rule: org.ts.test-placement class=judgment enforced=no tasks=code-organization --> Tests may sit beside the implementation
or in `tests/`, provided they are not App entry exports or runtime dependencies. <!-- rule: org.ts.project-references class=judgment enforced=no tasks=code-organization -->
Introduce [TypeScript project references](https://www.typescriptlang.org/docs/handbook/project-references.html)
only when multiple real projects need separate builds or dependency boundaries;
ordinary modules are enough for one App. <!-- rule: org.ts.native-structure-not-ownership class=judgment enforced=no tasks=code-organization --> Native project structure does not
replace Libertas package ownership or Studio's dependency resolution.

## Make a change through its owners

1. Locate the published Request, Response or stored variant and its feature
   payload type. Read the source documentation and applicable contract.
2. <!-- rule: org.change.domain-rule-and-handler class=judgment enforced=no tasks=code-organization --> Change the feature's domain rule and handler. Keep persistence or device
   effects in the module that performs them, and use that behavior for human,
   AI and software invocation.
3. <!-- rule: org.change.single-union class=hard enforced=no tasks=code-organization --> Update the single protocol/data Union when a new variant is required. Preserve
   its name, existing cases and order under [data-type evolution](data-type-evolution.md). <!-- rule: org.change.exports-and-docs-together class=judgment enforced=no tasks=code-organization -->
   Update exact-name root exports and localized source documentation together.
4. <!-- rule: org.change.presentation-only-when-needed class=hard enforced=no tasks=code-organization --> Update presentation source only when the same data needs a different
   arrangement. Generated schema, localized documents, compiled templates and
   deployment bindings remain generated under the [authoring contract](app-contract.md).
5. <!-- rule: org.change.meaningful-examples class=judgment enforced=no tasks=code-organization --> Check the affected behavior with meaningful examples. Cover relevant state
   transitions, invalid input, identity/access, conflicting updates and external
   effects; use the App's requirements to select the cases.

<!-- rule: org.change.file-only-reorganization class=judgment enforced=no tasks=code-organization -->
For a file-only reorganization, keep task entry names, published type identities,
Union order, field shapes, annotations and localized meaning unchanged. Run the
native checker and Studio validation, and review the generated schema, document
and Display Template for unintended changes. Follow the
[validation and handoff workflow](../codebot/delivery.md).

## Review checklist

- <!-- rule: org.review.findable class=judgment enforced=no tasks=code-organization --> Can a CodeBot find the entry function, each contract and the owning handler
  without tracing generic utility or export chains?
- <!-- rule: org.review.one-owner class=judgment enforced=no tasks=code-organization --> Does each domain rule and effect have one owner, shared by all participants?
- <!-- rule: org.review.intentional-exports class=judgment enforced=no tasks=code-organization --> Are root exports intentional and all schema-visible declarations documented?
- <!-- rule: org.review.union-order class=judgment enforced=no tasks=code-organization --> Are protocol and stored Union identities and order preserved?
- <!-- rule: org.review.minimal-dependencies class=judgment enforced=no tasks=code-organization --> Does each module depend only on what its responsibility needs?
- <!-- rule: org.review.justified-structure class=judgment enforced=no tasks=code-organization --> Are extra folders, wrappers and packages justified by real behavior?
- <!-- rule: org.review.checks-pass class=judgment enforced=no tasks=code-organization --> Do native checks, Studio validation and relevant behavior tests pass?
