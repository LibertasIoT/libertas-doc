# Libertas CodeBot entry

Work only from this documentation and the project's installed SDK. Fetch each
linked file from `https://raw.githubusercontent.com/LibertasIoT/libertas-doc/main/`,
resolving a relative link from the file that contains it. If you cannot open
the documentation, say so; do not write App source from memory.

Start with the [App interaction model](reference/app-contract.md#application-model):
**App is data interaction. Data is the GUI. Humans, AI and other Apps use the
same interaction contract.**

See [Hub architecture](concepts/hub.md) for platform responsibilities and the
roles of human users, AI and applications.

Use the [CodeBot workflow](codebot/index.md) to write or change a Libertas App.
It gives the required reading, selects contracts for the task, and ends with
validation and handoff.

Use the [common code organization practice](reference/code-organization.md) for
Rust and TypeScript project structure.

Follow the [collaboration goal](principles.md#collaboration-goal) and the
[smallest logically complete design requirement](reference/app-contract.md#required-design-principle).
Use the installed public SDK for exact runtime APIs and Studio's
[schema compiler workflow](reference/app-contract.md#schema-compilation-with-codex)
for saved source.

[TypeScript is under development and not release ready](reference/languages/typescript.md#development-status).

This entry and its linked documentation are published only in
[libertas-doc](https://github.com/LibertasIoT/libertas-doc).
