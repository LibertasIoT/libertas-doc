# App recipe index

Use the canonical [Libertas App recipes](../examples/app-recipes.md) when a task
needs a declaration starting shape. Endpoint, permission and persistence
recipes have empty runtime functions: implement the required behavior using
the owning contract and installed SDK before delivering an App. The
[runnable configuration example](../guides/developing-apps.md#complete-copyable-source)
is complete for its stated form-demonstration purpose. The language guides and the
[annotation inventory](schema_annotations.md) remain authoritative for syntax.

Choose the smallest matching recipe:

- [TypeScript Endpoint server](../examples/app-recipes.md#typescript-endpoint-server)
- [Rust Endpoint server](../examples/app-recipes.md#rust-endpoint-server)
- [TypesOnly shared package](../examples/app-recipes.md#typesonly-shared-package)
- [Function permission and reason](../examples/app-recipes.md#function-permission-and-reason)
- [Persistent data](../examples/app-recipes.md#persistent-data)

Never copy a schematic marked non-copyable (`evolution.schematics.not-source`).
Do not invent annotations, omit schema-visible documentation, hard-code
generated `AppDependencies` (`schema.dependencies.exact-version`), or change
an existing Union's name, variant order, discriminator, or payload meaning
([data-type evolution card](data_type_design.md)).
