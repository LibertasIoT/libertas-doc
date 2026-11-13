# Libertas App contract

This card is the entry to the [App authoring contract](../reference/app-contract.md).
The contract's rules are generated into the task cards below; load each one
the task needs, and read the linked owner before implementing its feature.
Never infer schema behavior from a summary.

| The task | Card |
| --- | --- |
| Any App design or change: the design principle, asking once, the data-first model, types and Nullable choices, discovery and dependencies, permissions, the task lifecycle, documentation, wire compatibility and the compile loop | [Designing an App](app_design.md) |
| A page whose values come from a document, or a new field on an importing page | [AI Import](ai_import.md) |
| `StringResources` templates, notifications, `FormattedText` values and data names | [Messages (LMF1)](messages_lmf1.md) |
| A function whose task process uses TCP or UDP (`AccessHost`) | [Network access](network_access.md) |
| Deployment to a Hub or Plaza publication | [Deploying and publishing](deployment_publication.md) |
| An Endpoint, its protocol Union, roles, access (`AccessPrivilege`, `Write`) or a client Endpoint's `Default` or `Fixed` selector | [Endpoint protocol](endpoint_protocol.md), with [Endpoint callback code](endpoint_runtime.md) for the App code that serves or calls it |
| A page searchable by AI Search | [AI Search](ai_search.md) |
| A storage schema or persistent records | [Persistent data](data_api.md) |
| Types, fields, attributes, Unions, keys, defaults, Schema Paths and `CopyFrom`/`CopyTo` | [Data model](data_model.md) |

Related owners outside these cards:

- [Data-type evolution](data_type_design.md) before changing a published
  protocol or stored representation.
- The [source documentation card](source_documentation.md) for every
  schema-visible node and public function.
- The [task pre-creation dependency check](../reference/schema.md#62-task-pre-creation-static-dependency-check)
  for a client Endpoint with `Default` or `Fixed`.

## Completion checklist

- Every required behavior has one owner and each fact is asked once
  (`app.design.*`, `app.ask.*`).
- Every document-sourced page declares import, each added field is classified,
  and replacement preserves person-supplied records (`app.import.*`).
- Every AI Search destination is bound and documented with its actual meaning,
  identifiers and common names ([AI Search card](ai_search.md)).
- Exports, schema-visible documentation, permission and host reasons and wire
  changes are intentional and complete (`app.discovery.*`, `app.docs.*`,
  `app.function.*`, `net.*`, `app.wire.*`, `app.checklist.*`).

Then [validate and report the result](delivery.md). Keep generated schema,
documents, dependency lists, manifests and release versions generated.
