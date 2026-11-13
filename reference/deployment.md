# Deployment dependency contract

<!-- rule: deploy.dependencies.just-in-time class=hard enforced=no tasks=client-runtime -->
The language parsers cannot assign dependency versions. In particular, the new
version of a local dependency in the same deployment batch is unknown until that
dependency has successfully deployed. Studio therefore constructs
`AppDependencies` just in time for each package:

<!-- rule: deploy.dependencies.discovery class=hard enforced=no tasks=deployment-publication -->
1. It discovers direct package references from every parser FQN in a reference
   `Type`, heritage, `Bitflags`, `ForeignTypeSchema`, `FunctionDataSchema`, or
   `ProtocolSchema`, plus the package component of every Endpoint-client
   `Fixed=developer-package::function` constraint. Endpoint `Default` does not
   create a dependency. <!-- rule: deploy.dependencies.source-cache class=hard enforced=yes tasks=deployment-publication title="A direct package needs its source in the package manager's local cache; a binary-only cache stops deployment" --> During queue planning—before any deployment begins—it
   recursively resolves local packages. For each remaining direct package,
   Studio locates the same package-manager source used by the language parser:
   TypeScript source or declarations in the nearest `node_modules`, or Rust
   source in Cargo's local cache resolved with offline metadata. It generates
   that package's schema as comparison evidence. A binary-only or incomplete
   cache is not sufficient and stops deployment. The deployment plan labels
   these as **cache dependencies**, distinctly from deployable **local
   dependencies** and exact **Plaza transitive dependencies**. <!-- rule: deploy.dependencies.plaza-schema-match class=hard enforced=yes tasks=deployment-publication title="A direct package's source-generated schema must equal its Plaza schema; a mismatch stops before Hub mutation" -->
2. Studio reads the selected Hub's exact Plaza distribution/architecture keys,
   searches Plaza for each direct package's latest target, downloads and verifies
   its schema, selected localized document, and Display Template, and compares the source-generated
   schema with the Plaza schema after removing deployment-owned dependency locks
   and revision metadata. The programming-language dependency version is never
   interpreted as a Plaza pin. A mismatch stops before Hub mutation. <!-- rule: deploy.dependencies.plaza-closure class=hard enforced=yes tasks=deployment-publication title="Missing or withdrawn Plaza targets, conflicting pins, invalid assets or cycles stop the entire preflight" -->
3. Studio follows the selected Plaza schema's exact `AppDependencies` locks and
   recursively resolves every historical target in their transitive closure.
   Direct packages select latest; transitive packages select the exact recorded
   App version. Missing or withdrawn targets, conflicting pins, invalid assets,
   or cycles stop the entire preflight. Studio installs the resulting closure on
   the Hub in dependency-first order and verifies each exact schema/version
   read-back before submitting any local package. An already installed exact
   target is verified and left unchanged.
4. For a queued local dependency, it first deploys that package, reads its schema
   and newly assigned App version back from the Hub, and verifies that the schema
   digest is the one just submitted.
5. <!-- rule: deploy.dependencies.verified-version-only class=hard enforced=no tasks=client-runtime --> It validates the dependent against the complete exact resource set. The new
   schema records the version actually verified for each package; it never
   predicts a version, substitutes Plaza latest for an exact transitive pin, or
   copies an unverified language-package version.
6. It validates and sorts the complete list, then computes the dependent schema's
   revision and submits the dependent package.

<!-- rule: deploy.dependencies.stop-on-failure class=hard enforced=no tasks=client-runtime title="A failed deployment or read-back stops dependents; never predict the Hub version or use 0.0.0.0 as a dependency" -->
If any deployment or read-back fails, Studio stops before submitting its
dependents. It never predicts the Hub version, and `0.0.0.0` is not a valid
dependency entry.

## Plaza installation access

<!-- rule: deploy.plaza.admin-connection class=hard enforced=no tasks=deployment-publication -->
Hub-assisted Plaza search, installation-state queries, and App installation or
upgrade require an authenticated administrative connection. Control connections
cannot perform these operations, even when the account has creation or
configuration permission. Administrative access grants admission only: reading
an installed App's installation state also requires read or configuration
permission on that App; installing a new App requires permission to add Apps;
and upgrading an installed App requires configuration permission on that App.

## Display Template preflight and synchronization

Every package deploys with its compiled
[Display Template](presentation.md#display-template). Preflight compiles it
from source and compares its digest with the Hub's current template for that
package, as it compares the schema and the selected localized document. A
template difference alone is a document-only change: the Hub stores the new
template, advances `targetVersion`, and preserves the App version, schema, and
artifact, with no rebuild. A schema or code change deploys the template with
the new App version. Synchronization converges the template with the
documents under the [document rules](publication.md#display-template-identity),
and an unchanged template writes nothing.

## Local translation preflight and synchronization

Local translation files participate in ordinary Studio and MCP deployment for
the requested package and local dependencies. The
[translation contract](translation.md#deployment-of-translations) owns
inventory authority, completion and review state, fallback, read-only checkout
behavior, project policy, and planning conflicts.

## MCP deployment

Studio's MCP server exposes `list_deployment_hubs`, `deploy_app`, and
`deployment_status` to Codex and Claude Code. <!-- rule: deploy.mcp.prerequisites class=hard enforced=no tasks=deployment-publication title="MCP deployment needs an explicit user request, a saved project directory and an exact registered Hub ID" --> Deployment requires an explicit
request from the user, a saved local Rust or TypeScript App project directory,
and an exact registered Hub ID returned by `list_deployment_hubs`. VS Code and
Libertas Studio must remain open. Hub credentials remain in Studio and are not
returned by the tools.

`deploy_app` returns a job ID immediately. Studio parses and validates the
schema, confirms developer mode on the selected Hub, plans the complete
dependency closure, prepares localized documents and the Display Template, and compiles an executable
TypeScript App before submission. If preflight fails, the job reports a
failure without starting the deployment queue. <!-- rule: deploy.mcp.automatic class=hard enforced=no tasks=deployment-publication --> After preflight, Studio deploys
automatically; there is no schema review or Submit step. Poll
`deployment_status` for the current phase, package audit, progress, and final
success or error. A failed package stops later packages. Rust source is built
by the Hub during deployment. <!-- rule: deploy.mcp.native-packages class=hard enforced=no tasks=deployment-publication --> If it needs native build packages that require
approval, the MCP job fails and reports the plan; use Studio's interactive
deployment to review and approve that installation.

`compile_schema` remains a separate, Hub-independent validation tool. Its
success alone does not establish exact Hub dependency pins or deploy the App.

| Rule debugging tool | Input and result |
| --- | --- |
| <!-- rule: deploy.mcp.rule-debugging-input class=hard enforced=no tasks=app-design --> `evaluate_validation_rules` | `project` and optional `entry` identify saved App source as for `compile_schema`; `type` names a Request variant or data type. Supply either `value` in schema JSON value form or `avroPath`, a saved Avro request file. Optional `phase` is `final` (default) or `incremental`. Returns structured rule traces and a short text summary, with compile diagnostics. |

<!-- rule: deploy.rule-debugger.no-hub-or-submit class=hard enforced=no tasks=app-design -->
Rule debugging compiles the same saved source and materialized dependencies
as `compile_schema`, then uses Studio's ordinary rule evaluator. It needs no
Hub connection and never submits the supplied request. <!-- rule: deploy.mcp.rule-debugging-values class=hard enforced=no tasks=app-design --> JSON supplies the named
Request's payload; omitted members remain unanswered and explicit null remains
absence. A saved Avro Request includes its protocol Union discriminant.
`avroPath` is absolute or relative to `project`. See
[Debugging rules](validation-rules.md#debugging-rules) for outcomes, lazy reads,
developer presentation, and CI fixtures.

<!-- rule: deploy.mcp.preview-display class=hard enforced=no tasks=app-design -->
`preview_display` shows one screen of the saved workspace App in Studio's
Display Template preview, Hub-independent like `compile_schema`. It takes the
absolute project directory, the protocol variant (a Union member such as a
Response; `Union.Member` names it exactly when two Unions have a member of
that name, and the result's `ambiguousVariant` lists the candidates when an
unqualified name matched several) or function name to show, a device `form` (`phone`, `tablet`,
`desktop`, or `tv`), optional `width`, `height`, `orientation`, `pointer`, and
`theme` size-class/condition overrides, and an optional Libertas Schema Path
`select` naming a node to inspect. It compiles the project, then opens a
preview if none is open for it or refocuses the open one and switches it to
this screen and device; later saves re-render the same preview live. Its JSON
result reports `preview` (`opened`, `refocused`, or `headless` with a
`reason`), the resolved device conditions and window, the root node's
resolved layout, style, and hints, and, when `select` is given, the matched
rules in cascade order with their winning members and each rule's source (a
`display.json` line or a source attribute's file and line). Compile
diagnostics are included, and `templateCurrent` is `false` when a failed
compilation leaves the last good template on screen. It requires VS Code and
Libertas Studio to be running and fails explicitly otherwise.
For a local Request sample, optionally supply `value` (the variant's JSON
payload, without its outer variant key) or `avroPath` (a saved raw protocol
Avro Request including its Union discriminant, absolute or relative to the
project), never both: the sample overrides open-task values and defaults,
preserves unanswered JSON fields without initialization, and stays selected
on later source saves; malformed input or an Avro variant mismatch fails
explicitly. The result identifies `displayed` as `sample-value` or
`sample-file` and `sample` names the input source, qualified variant and file
path when applicable. Samples use local codecs without contacting a Hub or
submitting a request; function and non-Request screens accept no sample.

## Schema availability and task requirements

<!-- rule: deploy.dependencies.pinned-history class=hard enforced=no tasks=client-runtime -->
At runtime, `AppDependencies` is both the client's complete package-fetch list and
the Hub's preloaded authorization and exact-version lookup table. The Hub reads
the historical schema, documents, manifest, and code belonging to the pinned
version; it never substitutes the package's current version. Clients verify that
the returned App version equals the declared pin before assembling the graph. <!-- rule: deploy.version.exact-identity class=hard enforced=no tasks=client-runtime -->
Every modern producer and consumer uses the required case-sensitive `Version`
field. It is an exact identity, never a minimum or latest-version request.

<!-- rule: deploy.dependencies.schema-before-editing class=hard enforced=no tasks=client-runtime -->
This closure covers the complete published schema, including optional fields
and unselected Union alternatives. Its schemas and localized documents must
already be available in the Hub's type system so editors can resolve and display
every supported choice without interrupting editing for on-demand package
installation. <!-- rule: deploy.dependencies.no-service-task class=hard enforced=no tasks=deployment-publication --> Listing an App dependency does not by itself require a configured
service task. Required Endpoint services are determined separately by the
[task pre-creation gate](schema.md#62-task-pre-creation-static-dependency-check).


For the human workflow, see [deploying with Studio](../guides/studio.md#deploying-to-a-hub).
For publication identities, see [Plaza publication](publication.md).
