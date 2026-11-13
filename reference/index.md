# Shared detailed reference

These pages own the precise contracts used by human developers and CodeBots.
Human guides explain the workflow; CodeBot cards select the minimum complete
context. Both link to these same owners. Source-language examples belong in the
language pages, while portable semantics belong in the relevant topic below.

- [App contract](app-contract.md)
- [App code organization for Rust and TypeScript](code-organization.md)
- [User-facing and non-user-facing type design](app-contract.md#user-facing-and-non-user-facing-types)
- [TypeScript — under development, not release ready](languages/typescript.md#development-status)
- [Rust](languages/rust.md)
- [Source annotations](annotations.md)
- [Schema artifacts](schema.md)
- [Libertas Schema Path](schema.md#libertas-schema-path)
- [Documentation](documentation.md)
- [Function comments](function-comments.md)
- [Translation](translation.md)
- [Data-type evolution](data-type-evolution.md)
- [Validation rules](validation-rules.md)
- [String formats, Pattern and matches](validation-rules.md#132-full-string-patterns)
- [Endpoint protocols](protocols.md)
- [Storage schemas and key encoding](protocols.md#storage-schema-declarations), [transactional KV API](protocols.md#transactional-key-value-api)
- [LLM document transformation](llm-document-transformation.md)
- [Endpoint lifecycle](endpoint-lifecycle.md)
- [Endpoint presentation](endpoint-presentation.md)
- [Presentation and localization](presentation.md)
- [Message format](message-format.md)
- [2D charts](charts/2d.md)
- [3D charts](charts/3d.md)
- [Matter descriptors](matter/descriptors.md)
- [Matter status](matter/status.md)
- [Network access](network-access.md)
- [Deployment dependencies](deployment.md)
- [Plaza publication](publication.md)

<!-- rule: sdk.owns-runtime-apis class=hard enforced=no tasks=app-design -->
The installed public SDK owns exact runtime APIs:
[`@libertas-ai/types`](https://www.npmjs.com/package/@libertas-ai/types) for
TypeScript and [`libertas-rs`](https://github.com/LibertasIoT/libertas-rs) for
Rust. Use the version selected by the project. Chart references define their
versioned contracts and fallback behavior; they do not certify every older
client build.

[Human guides](../guides/index.md) · [CodeBot instructions](../codebot/index.md)
· [Documentation home](../index.md)
