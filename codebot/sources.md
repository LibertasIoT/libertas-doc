# Source policy

## Choose the authority

| Question | Owner |
| --- | --- |
| Exact runtime APIs, types, callbacks and lifetimes | The public SDK version selected by the project. |
| Native syntax, resolution and extraction | The source-language parser; the language reference documents its contract. |
| Portable schema semantics | Studio's shared validator. |
| Full authoring behavior and explanation | The [shared public references](../reference/index.md). |

The [App contract](../reference/app-contract.md) defines these responsibilities.
CodeBot cards select compact task context; examples do not add APIs or override
the owners. Use only public SDK declarations and listed source annotations,
never spellings inferred from generated output or private implementation.

## Handle a missing or conflicting contract

Report the exact symbol, package/SDK version, conflicting sections and desired
behavior to the requester. Use the project's published issue/support channel
when available and the task authorizes reporting. Do not invent a private
contact or inspect private implementation to fill an App-contract gap.

## Canonical public documentation location

The reusable CodeBot URL is
`https://github.com/LibertasIoT/libertas-doc/blob/main/CODEBOT.md`.
Use that [CodeBot entry](../CODEBOT.md) in prompts and tools; a one-line prompt
can use its raw form,
`https://raw.githubusercontent.com/LibertasIoT/libertas-doc/main/CODEBOT.md`,
which AI assistants fetch more reliably. The canonical
[libertas-doc repository](https://github.com/LibertasIoT/libertas-doc) publishes
all linked contracts and guides; its [homepage](../index.md) provides human
orientation.

Clone the repository for offline reading. `main` is the current documentation; the
repository carries that edition only. Runtime
APIs still follow the SDK version selected by the project. SDK entries link
here and contain no second documentation bundle.

For an AI answering questions rather than editing App source, use the
[developer assistant](../DEVELOPER-ASSISTANT.md) or
[user assistant](../USER-ASSISTANT.md) entry.
