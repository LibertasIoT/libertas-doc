# Deploying and publishing

Load this card when a task deploys an App to a Hub or publishes it to Plaza.
Deploy or publish only when the user requests it. Its rules are generated from
the [deployment dependency contract](../reference/deployment.md), the
[Plaza publication contract](../reference/publication.md) and the
[App contract](../reference/app-contract.md#wire-and-compatibility-discipline),
which own them. The [Studio guide](../guides/studio.md) and
[Plaza guide](../guides/plaza.md) own the interactive workflow.

## Order of work

1. Compile first under the [App design card](app_design.md); deployment
   resolves the dependencies itself (`deploy.dependencies.*`,
   `deploy.version.*`).
2. Deploy through the MCP tools with an exact Hub ID (`deploy.mcp.*`); Plaza
   installation needs an administrative connection (`deploy.plaza.*`).
3. To publish, deploy the same source to every target Hub first, and publish a
   recommended service App and each dependency separately
   (`publish.package.*`, `publish.auth.*`).
4. Check which version and release each change creates (`publish.preflight.*`,
   `publish.schema.*`, `publish.version.*`, `app.wire.plaza-release-immutable`), and leave the
   Studio-owned assets generated (`publish.assets.*`, `publish.documents.*`,
   `publish.install.*`).

<!-- generated:codebot-rules task=deployment-publication -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### Wire and compatibility discipline

Owner: [app-contract.md#wire-and-compatibility-discipline](../reference/app-contract.md#wire-and-compatibility-discipline)

- `app.wire.plaza-release-immutable`: The common schema/dependency lock and each existing target's exact App version, binary, language, TypesOnly state, and distribution dependencies are immutable within one Plaza release. Changing any of them creates a new content-derived release; adding or removing targets and changing documents or allowed manifest metadata synchronizes the current release. App source never chooses the release tag or public version. See the [Plaza publication contract](../reference/publication.md#app-publication-and-version-identity).

### Deployment dependency contract

Owner: [deployment.md#deployment-dependency-contract](../reference/deployment.md#deployment-dependency-contract)

- `deploy.dependencies.discovery`: It discovers direct package references from every parser FQN in a reference `Type`, heritage, `Bitflags`, `ForeignTypeSchema`, `FunctionDataSchema`, or `ProtocolSchema`, plus the package component of every Endpoint-client `Fixed=developer-package::function` constraint. Endpoint `Default` does not create a dependency.

### Plaza installation access

Owner: [deployment.md#plaza-installation-access](../reference/deployment.md#plaza-installation-access)

- `deploy.plaza.admin-connection`: Hub-assisted Plaza search, installation-state queries, and App installation or upgrade require an authenticated administrative connection. Control connections cannot perform these operations, even when the account has creation or configuration permission. Administrative access grants admission only: reading an installed App's installation state also requires read or configuration permission on that App; installing a new App requires permission to add Apps; and upgrading an installed App requires configuration permission on that App.

### MCP deployment

Owner: [deployment.md#mcp-deployment](../reference/deployment.md#mcp-deployment)

- `deploy.mcp.prerequisites`: Deployment requires an explicit request from the user, a saved local Rust or TypeScript App project directory, and an exact registered Hub ID returned by `list_deployment_hubs`. VS Code and Libertas Studio must remain open. Hub credentials remain in Studio and are not returned by the tools.
- `deploy.mcp.automatic`: After preflight, Studio deploys automatically; there is no schema review or Submit step. Poll `deployment_status` for the current phase, package audit, progress, and final success or error. A failed package stops later packages. Rust source is built by the Hub during deployment.
- `deploy.mcp.native-packages`: If it needs native build packages that require approval, the MCP job fails and reports the plan; use Studio's interactive deployment to review and approve that installation.

### Schema availability and task requirements

Owner: [deployment.md#schema-availability-and-task-requirements](../reference/deployment.md#schema-availability-and-task-requirements)

- `deploy.dependencies.no-service-task`: Listing an App dependency does not by itself require a configured service task. Required Endpoint services are determined separately by the [task pre-creation gate](../reference/schema.md#62-task-pre-creation-static-dependency-check).

### Account and publication authorization

Owner: [publication.md#account-and-publication-authorization](../reference/publication.md#account-and-publication-authorization)

- `publish.auth.scoped-account-access`: Private account reads require `account:read`; account changes, token creation, revocation, and repository authorization require `account:write`.
- `publish.auth.token-scope`: Publication and withdrawal require `store:publish` on an active Plaza API token. Browser sessions cannot publish or withdraw, and account disabling prevents access.
- `publish.auth.delegation-bounded`: A token created using another token has only scopes held by its parent and cannot expire later than its parent. Omitting an expiry still inherits a parent's limit. Revoking or expiring any ancestor invalidates its descendants; delegation supports at most eight generations. New delegated tokens require their own publishing reauthentication.
- `publish.auth.github-recent`: Publication and withdrawal require GitHub authentication for the exact calling Plaza token within the preceding 15 minutes. Signing into the website or using a different Plaza token does not renew that authority. Publication additionally requires public-repository write permission verified for that same token and repository within 15 minutes. Repository authorization must use the GitHub identity belonging to the signed-in Plaza account. Revocation or expiry of the initiating Plaza token, or disabling its account, invalidates an unfinished repository authorization and its pending credential handoff.
- `publish.auth.github-revocation-separate`: Studio refreshes GitHub repository authorization before publication and withdrawal, preserving the explicit repository selection. Plaza rechecks a publishing repository's stable identity and public visibility when verifying release assets. External write-permission removal takes effect at the next authorization refresh or when the existing 15-minute proof expires; it is not an instantaneous GitHub revocation notification. Revoking a Plaza token stops its Plaza authority; GitHub authorization is a separate credential and must be revoked at GitHub if that access should also end.

### Package and target identity

Owner: [publication.md#package-and-target-identity](../reference/publication.md#package-and-target-identity)

- `publish.package.deployed-first`: A publication comes only from chosen Hubs. Choose one or more registered Hubs, then one App that is current on every chosen Hub, including a TypesOnly package; the chooser lists only Apps whose developer is your Plaza ID, matched by package identity and language. Each chosen Hub contributes its own platform's deployed target. No local project or workspace file is read, and nothing from a Hub you did not choose. Deploy the App to each intended Hub first.
- `publish.package.service-separately`: Deploy and publish the recommended service App separately, with a target for each supported Hub platform.
- `publish.package.types-only-no-service`: Publishing a TypesOnly protocol package makes its types available; it does not publish a service that implements the protocol.
- `publish.package.service-task`: A configured service Task must then create a compatible Endpoint before the calling Task can use it; publishing or installing an App does not create that Task.

### Preflight and latest documents

Owner: [publication.md#preflight-and-latest-documents](../reference/publication.md#preflight-and-latest-documents)

- `publish.preflight.stops`: Studio reads the chosen Hubs, Plaza history, and the package's current GitHub Release before changing GitHub or Plaza. A foreign developer, duplicate targets, differing [stored schemas](../reference/publication.md#preflight-and-latest-documents), an exact dependency unavailable on Plaza for a chosen platform, a binary failing its recorded SHA-256, or a version or binding conflict with retained history stops preflight. Targets may have different App versions while sharing the same schema and exact dependency lock. Publish each dependency package before the Apps that pin it.

### Hub target manifest

Owner: [publication.md#hub-target-manifest](../reference/publication.md#hub-target-manifest)

- `publish.version.new-source-revision`: A deployment with a changed source revision receives a new App version, even when the compiled artifact is identical or the package is TypesOnly. Installing an existing Plaza release preserves its published build time.
- `publish.version.dependency-pin-upgrade`: Consequently changing any dependency pin changes schema identity, requires a new App version for every affected target carrying that schema, and is an upgrade even when the target artifact bytes happen to be identical. It is never classified as document-only synchronization.

### App publication and version identity

Owner: [publication.md#app-publication-and-version-identity](../reference/publication.md#app-publication-and-version-identity)

- `publish.documents.default-locale-remains`: At least the declared default locale must remain.

### Display Template identity

Owner: [publication.md#display-template-identity](../reference/publication.md#display-template-identity)

- `publish.template.document-only-change.authoring`: Use Studio publication for generated assets and identities; [template-only changes preserve the App version and need no rebuild](../reference/publication.md#display-template-identity).

### Public release assets

Owner: [publication.md#public-release-assets](../reference/publication.md#public-release-assets)

- `publish.assets.studio-owned`: Studio owns these files; App authors must not hand-edit them.

### Publication completion and recovery

Owner: [publication.md#publication-completion-and-recovery](../reference/publication.md#publication-completion-and-recovery)

- `publish.install.exact-dependency`: Installation cannot substitute a different target or latest version for an unavailable exact dependency. Re-adding the same retained binding restores availability; changed runtime content follows the version rules above.

### Translation development files

Owner: [translation.md#translation-development-files](../reference/translation.md#translation-development-files)

- `translate.project.git-inventory`: `translations/project.json` declares the complete managed locale inventory for one package. Saving translations in Studio creates this inventory. Commit it, the declared `translations/<package>.<locale>.doc.json` files, and the whole `translations/state/` directory together. Generated prompts, worksets, results, and the older `manifest.json` export are not required for deployment from a fresh checkout; a managed project's `manifest.json` carries no per-locale translation baselines, because its state supersedes them. A package without `project.json` keeps the existing Hub-document retention behavior until its translations are saved into a managed project.

### Deployment of translations

Owner: [translation.md#deployment-of-translations](../reference/translation.md#deployment-of-translations)

- `translate.deploy.local-authority`: Normal Studio and MCP deployment reads the local managed inventory automatically, including for each local dependency. It projects new original-locale fallbacks and stale-check invalidation without rewriting tracked files. The declared locale set wins over the Hub's locale set; omitted locales are removed from the current set, with historical artifacts retained. Changing only translation text synchronizes metadata without rebuilding the App or changing its App version, code, schema, dependency pins, or original build provenance. Changing only checks changes nothing on the Hub. Repeating the same bundle is idempotent. A changed project bundle after planning requires a new plan.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Deployment dependency contract

Owner: [deployment.md#deployment-dependency-contract](../reference/deployment.md#deployment-dependency-contract)

- `deploy.dependencies.source-cache`: A direct package needs its source in the package manager's local cache; a binary-only cache stops deployment
- `deploy.dependencies.plaza-schema-match`: A direct package's source-generated schema must equal its Plaza schema; a mismatch stops before Hub mutation
- `deploy.dependencies.plaza-closure`: Missing or withdrawn Plaza targets, conflicting pins, invalid assets or cycles stop the entire preflight

### Package and target identity

Owner: [publication.md#package-and-target-identity](../reference/publication.md#package-and-target-identity)

- `publish.package.own-developer-only`: A publication's developer must be the signed-in account's own Plaza ID

### Preflight and latest documents

Owner: [publication.md#preflight-and-latest-documents](../reference/publication.md#preflight-and-latest-documents)

- `publish.schema.hubs-agree`: Every Hub chosen for one publication holds the identical stored App schema

### App publication and version identity

Owner: [publication.md#app-publication-and-version-identity](../reference/publication.md#app-publication-and-version-identity)

- `publish.version.must-exceed-maximum`: A changed target needs an exact version above the package's greatest published public version

### Translation development files

Owner: [translation.md#translation-development-files](../reference/translation.md#translation-development-files)

- `translate.project.integrity-reconciliation`: Translation text, state, and inventory must be reconciled before deployment

### Completion and review in Studio

Owner: [translation.md#completion-and-review-in-studio](../reference/translation.md#completion-and-review-in-studio)

- `translate.project.optional-review-policy`: A required-review locale stays a draft until all current values are reviewed

<!-- /generated -->
