# Deployment targets and Plaza publication

Deploying installs an App on one Hub. Publishing makes verified deployed
artifacts publicly discoverable and downloadable for selected targets. Studio
publishes the existing artifacts of the chosen Hubs; it does not compile, build,
redeploy, read a local project, upload source, or invent an App version. Follow
the [Plaza guide](../guides/plaza.md) for the wizard and account controls. This
page owns the release and artifact contract.

## Account and publication authorization

These rules apply when the updated Plaza service and Studio builds are deployed
together; they do not establish the version running at a hosted service.

This security upgrade preserves existing accounts and publication history but
revokes older Plaza API tokens. Sign in again in Studio, refresh GitHub
repository authorization, and recreate any additional API tokens. Browser
sessions remain valid.

<!-- rule: publish.auth.scoped-account-access class=hard enforced=no tasks=deployment-publication -->
Private account reads require `account:read`; account changes, token creation,
revocation, and repository authorization require `account:write`. <!-- rule: publish.auth.token-scope class=hard enforced=no tasks=deployment-publication --> Publication
and withdrawal require `store:publish` on an active Plaza API token. Browser
sessions cannot publish or withdraw, and account disabling prevents access.

<!-- rule: publish.auth.delegation-bounded class=hard enforced=no tasks=deployment-publication -->
A token created using another token has only scopes held by its parent and
cannot expire later than its parent. Omitting an expiry still inherits a
parent's limit. Revoking or expiring any ancestor invalidates its descendants;
delegation supports at most eight generations. New delegated tokens require
their own publishing reauthentication.

<!-- rule: publish.auth.github-recent class=hard enforced=no tasks=deployment-publication -->
Publication and withdrawal require GitHub authentication for the exact calling
Plaza token within the preceding 15 minutes. Signing into the website or using
a different Plaza token does not renew that authority. Publication additionally
requires public-repository write permission verified for that same token and
repository within 15 minutes. Repository authorization must use the GitHub
identity belonging to the signed-in Plaza account. Revocation or expiry of the
initiating Plaza token, or disabling its account, invalidates an unfinished
repository authorization and its pending credential handoff.

<!-- rule: publish.auth.github-revocation-separate class=hard enforced=no tasks=deployment-publication -->
Studio refreshes GitHub repository authorization before publication and
withdrawal, preserving the explicit repository selection. Plaza rechecks a
publishing repository's stable identity and public visibility when verifying
release assets. External write-permission removal takes effect at the next
authorization refresh or when the existing 15-minute proof expires; it is not
an instantaneous GitHub revocation notification. Revoking a Plaza token stops
its Plaza authority; GitHub authorization is a separate credential and must be
revoked at GitHub if that access should also end.

## Package and target identity

A package is identified by `developer-package`. A target is one exact
`distribution`/`architecture` pair. One publication can contain several targets,
each retaining its own Hub-assigned exact `appVersion`. `TypesOnly=true`
publishes as `types_only`, with no executable; other packages publish as `app`.
There is no separate agent/automaton publication choice.

<!-- rule: publish.package.deployed-first class=hard enforced=no tasks=deployment-publication -->
A publication comes only from chosen Hubs. Choose one or more registered Hubs,
then one App that is current on every chosen Hub, including a TypesOnly
package; the chooser lists only Apps whose developer is your Plaza ID, matched
by package identity and language. Each chosen Hub contributes its own
platform's deployed target. No local project or workspace file is read, and
nothing from a Hub you did not choose. Deploy the App to each intended Hub
first.

<!-- rule: publish.package.own-developer-only class=hard enforced=yes tasks=deployment-publication title="A publication's developer must be the signed-in account's own Plaza ID" -->
A publication's developer must be the signed-in account's own Plaza ID. Studio
stops before any GitHub or Plaza change otherwise. Plaza takes the developer
name only from the account behind the publishing token, never from anything
Studio sends: it catalogs the package as `{account}-{package}` and rejects a
publication whose release marker or target manifests name another developer or
package, with an error naming both. Every Hub installing from Plaza checks that
the downloaded target manifest names the package it requested from the
catalog. Together these checks mean nobody can publish an installable App under
another developer's name, even with a modified Studio; the Studio-only token
requirement is not part of this guarantee.

Logical App dependencies belong to the schema's exact
[`AppDependencies` lock](deployment.md). Native OS packages belong to the target
manifest. A language package constraint is neither of those identities.

An Endpoint client's `Default` names a recommended service function. It does
not add that service App to `AppDependencies` or to the caller's publication
batch. <!-- rule: publish.package.service-separately class=hard enforced=no tasks=deployment-publication --> Deploy and publish the recommended service App separately, with a target
for each supported Hub platform. <!-- rule: publish.package.types-only-no-service class=hard enforced=no tasks=deployment-publication --> Publishing a TypesOnly protocol package makes
its types available; it does not publish a service that implements the protocol. <!-- /rule -->
An Endpoint client's `Fixed` restricts the owning function and includes its
package in the dependency graph.

Installing the service App makes its functions available. <!-- rule: publish.package.service-task class=hard enforced=no tasks=deployment-publication --> A configured service
Task must then create a compatible Endpoint before the calling Task can use it;
publishing or installing an App does not create that Task.

## Preflight and latest documents

<!-- rule: publish.preflight.stops class=hard enforced=no tasks=deployment-publication title="Publication preflight stops on a foreign package, duplicate targets, a schema mismatch, an unavailable dependency or a history conflict" -->
Studio reads the chosen Hubs, Plaza history, and the package's current GitHub
Release before changing GitHub or Plaza. A foreign developer, duplicate
targets, differing [stored schemas](#preflight-and-latest-documents), an exact
dependency unavailable on Plaza for a chosen platform, a binary failing its
recorded SHA-256, or a version or binding conflict with retained history stops
preflight. Targets may have different App versions while sharing the same
schema and exact dependency lock. Publish each dependency package before the
Apps that pin it.

<!-- rule: publish.schema.hubs-agree class=hard enforced=yes tasks=deployment-publication title="Every Hub chosen for one publication holds the identical stored App schema" -->
The chosen Hubs' stored schemas for the App, including the exact dependency
lock, must be byte-identical, compared by SHA-256. A difference stops
publication before any GitHub or Plaza change and names each Hub's digest;
deploy the same source to every chosen Hub and retry.

<!-- rule: publish.documents.latest-version class=hard enforced=no tasks=client-runtime -->
For each locale present on any chosen Hub, Studio publishes the document with
the highest version the Hubs report for that locale: the four-part content
revision the Hub assigns when it stores the document, compared numerically.
Equal versions are the same revision, so either Hub's copy is taken; a document
without a version ranks lowest. The [Display Template](#display-template-identity)
is chosen the same way by its highest content revision. `DocumentVersion` and
`DisplayVersion` are JSON format versions and never choose a winner.
Publication writes nothing to any Hub.

## Hub target manifest


<!-- rule: publish.artifacts.exact-installed-only class=hard enforced=no tasks=client-runtime -->
After every successful deploy, or a document-metadata synchronization that
actually changes document metadata, including the Display Template, the Hub writes a new immutable,
separately versioned target manifest and atomically advances the current pointer
for that language. An idempotent metadata request writes nothing. A
document-only revision
advances `targetVersion` but preserves `appVersion`, schema, artifact, language,
TypesOnly state, and distribution dependencies. The Hub retains historical
manifests, schemas, App manifests, localized
document mappings and files, and executable artifacts by exact App version. A
Hub stores only its own platform target. Studio retrieves an exact manifest and
installed artifact; publishing never rebuilds an App or allocates a replacement
version.

The JSON contract is:

```json
{
  "formatVersion": 1,
  "developerPackage": "developer-package",
  "developer": "developer",
  "package": "package",
  "language": "rs",
  "sourceRevision": "sha256:...",
  "targetVersion": "20260820.143025.417.0",
  "appVersion": "20260820.142959.003.0",
  "target": {
    "distribution": "resolute",
    "architecture": "arm64"
  },
  "typesOnly": false,
  "artifact": {
    "name": "libdeveloper__package.20260820.142959.003.0.so",
    "size": 123456,
    "sha256": "sha256:..."
  },
  "DistroDependencies": {
    "manager": "apt",
    "packages": ["libc6", "libgcc-s1"]
  },
  "schema": { "size": 1234, "sha256": "sha256:..." },
  "documents": { "count": 2, "sha256": "sha256:..." },
  "builtAtUnixMilliseconds": 1787241025417
}
```

<!-- rule: publish.artifacts.published-target-manifest class=hard enforced=no tasks=client-runtime -->
Studio publishes each chosen Hub's current target manifest with two changes:
`documents` describes the published [latest documents](#preflight-and-latest-documents),
and `translation` is removed. Every other member is the Hub's. The artifact is
the Hub's stored binary, byte for byte.

<!-- rule: publish.artifacts.compression-not-content class=hard enforced=no tasks=client-runtime -->
`artifact` is `null` for TypesOnly. `schema.sha256` and `schema.size` describe
the decoded canonical schema. `documents.sha256` identifies the canonical raw
format-2 documents inventory, and `documents.count` is its locale count. For a
format-3 gzip inventory, this identity is computed by replacing each compressed
asset descriptor with its decoded `contentSha256` and `contentSize` and its raw
`document.<locale>.<contentSha256>.json` or `display.<contentSha256>.json` name,
keeping sorted locales and the default locale, and serializing the resulting
format-2 JSON with sorted object keys and no whitespace. The downloaded
format-3 manifest has its own asset digest. Compression never changes the
Hub's schema or document content identity. Because that manifest names the
[Display Template](#display-template-identity), its digest covers the template
too; the target manifest keeps this shape and the template is not a locale. The target manifest's APT dependency
names are deduplicated and sorted. They are per-target OS installation data and never
appear in the Libertas schema. This is distinct from the schema's logical
`AppDependencies` array of Libertas App packages. For native Rust, the compiling
Hub resolves linked shared libraries to the owning APT packages after its local
build.

<!-- rule: publish.artifacts.no-overwrite class=hard enforced=no tasks=client-runtime -->
Studio publishes each target-manifest revision under a content-addressed name,
`{distribution}-{architecture}.target.{sha256}.json`. It never overwrites the
prior asset. For one platform and App version, the greatest numeric
`targetVersion` is current; older manifest revisions remain available for
in-flight downloads and exact history.

<!-- rule: publish.artifacts.target-source-revision class=hard enforced=no tasks=client-runtime -->
`sourceRevision` in every target-manifest revision is immutable Hub deployment
provenance for that exact App/runtime binding. Document synchronization copies
it from the prior target manifest. It must not replace it with Studio's current
project or publication source revision. `builtAtUnixMilliseconds` is likewise
the original target build time. Document revisions and redeployments that reuse
the same source revision, schema, artifact, and OS dependencies preserve it. <!-- rule: publish.version.new-source-revision class=hard enforced=no tasks=deployment-publication -->
A deployment with a changed source revision receives a new App version, even
when the compiled artifact is identical or the package is TypesOnly. Installing
an existing Plaza release preserves its published build time.

The manifest's exact `appVersion` is its install identity. Its schema digest
covers the canonical schema including the complete exact
`AppDependencies` lock. <!-- rule: publish.version.dependency-pin-upgrade class=hard enforced=no tasks=deployment-publication --> Consequently changing any dependency pin changes schema
identity, requires a new App version for every affected target carrying that
schema, and is an upgrade even when the target artifact bytes happen to be
identical. It is never classified as document-only synchronization.

### Translation review state

<!-- rule: publish.translations.hub-keeps-none class=hard enforced=no tasks=client-runtime -->
The Hub stores and serves no translation authoring companion, and target
manifests carry no `translation` descriptor. Translation and review state lives
only in the App's [translation project files](translation.md#translation-development-files)
under Git. Studio is the authority for translation content: the Hub receives
only the locale documents and stores them exactly as Studio supplies them,
without validating them, so a save completes in the time it takes to store the
bytes. Runtime clients use the
same locale documents.

<!-- rule: publish.translations.metadata-revision class=hard enforced=no tasks=client-runtime -->
The Hub keeps one metadata revision per App and language: the current target
manifest's `targetVersion`, empty before the first deployment. Studio reads it
when a deployment or metadata update starts and sends it with the save. The Hub
refuses the save when another save changed the revision in between; retry the
save. Saves are otherwise last-writer-wins.

<!-- rule: publish.translations.documents-only-update class=hard enforced=no tasks=client-runtime -->
A translation save is a documents-only update. Studio sends the complete current
locale document set and the metadata revision, with no schema and no Display
Template. The Hub checks only the caller's authority and the metadata revision,
then replaces the current locale documents. It does not read, parse, compare, or
validate the schema or the template. A deployment between Studio's
[schema check](translation.md#completion-and-review-in-studio) and the save
changes the metadata revision, so the Hub refuses that save.
A save whose complete locale document set equals the Hub's current one, as
confirmed by its unchanged metadata revision, sends no documents.

<!-- rule: publish.metadata.no-schema-change class=hard enforced=no tasks=client-runtime -->
A metadata update never changes an App's schema, App version, code, or
dependency pins; only a deployment does. The Hub refuses a metadata update that
supplies a schema different from the installed one.

<!-- rule: publish.translations.not-published class=hard enforced=no tasks=client-runtime -->
Plaza releases carry no translation companion. Studio removes `translation`
from each published target manifest, and Plaza refuses a target manifest that
still names one. Companion files in existing releases remain untouched history.

## App publication and version identity

<!-- rule: publish.identity.manifest-content-binding class=hard enforced=no tasks=client-runtime -->
A runtime binding consists of the exact App version, canonical schema digest,
target pair, language, TypesOnly state, artifact digest, and normalized runtime
OS dependencies. Target version, source revision, build time, and documents do
not replace that identity. Each admitted binding is immutable; availability
and current metadata pointers may change.

<!-- rule: publish.identity.high-water-never-decreases class=hard enforced=no tasks=client-runtime -->
The one public Plaza version starts as the numeric, component-wise maximum exact
`appVersion` in the reconciled creation target set. Thereafter it is the
greatest exact target App version ever admitted to that publication and never
decreases when a target is withdrawn. It is not an independently allocated
timestamp and it does not replace any target's exact version. For example,
creation bindings `20260820.142959.003.0` and `20260821.090000.010.0` produce
public version `20260821.090000.010.0`, while each installation retains its own
value. If a Hub later deploys content-equivalent bytes under a different
`appVersion`, that exact version remains a new immutable binding and follows the
normal new-publication and maximum-version rules.

Public versions are unique within a package and Plaza retains every version. <!-- rule: publish.version.must-exceed-maximum class=hard enforced=yes tasks=deployment-publication title="A changed target needs an exact version above the package's greatest published public version" -->
If the immutable schema or runtime content of an existing target changes but
the numeric target maximum does not advance beyond that package's greatest
published public version, Studio fails preflight before any Hub, GitHub, or
Plaza publication mutation. The affected target must be redeployed with an
exact version that makes the new maximum greater; other unchanged targets do
not need to be redeployed. A changed exact dependency lock is an immutable
schema change and follows this forced-upgrade rule.

<!-- rule: publish.version.absent-target-joins class=hard enforced=no tasks=client-runtime -->
A previously absent target whose exact App version is less than or equal to the
publication's greatest-ever version may be appended to that same publication as
a new immutable binding without changing the public version, marker, or tag. A
new target above that version creates a new publication whose public version is
the new maximum. <!-- rule: publish.version.withdrawal class=hard enforced=no tasks=client-runtime --> Withdrawing any target, including the one that established the
maximum, changes only current availability and never lowers or reassigns the
public version. Re-adding a withdrawn target reuses its retained exact binding;
different content or version is not an in-place re-addition.

<!-- rule: publish.legacy.preserve-exact-target-bindings class=hard enforced=no tasks=client-runtime -->
Before asking the developer to confirm, Studio reads the matching GitHub Release
with the newest publication time and Plaza history, then downloads and verifies
its marker, schema, current document manifest, locale assets, and Display Template, and complete immutable
target binding map for a current-format release. A recognized legacy release
(a format-1 marker or the older asset layout without a marker) is historical
input: Studio skips its obsolete asset verification and publishes a new
current-format release. The selected targets must still produce a public
version greater than every retained Plaza version, and existing exact target
bindings must remain unchanged. Studio preserves the legacy release and its
assets; it cannot synchronize documents into that release. With retained Plaza
history, a missing GitHub release still stops publication. Unrecognized or
malformed markers, failed current-format integrity checks, and GitHub access
errors also stop publication.

<!-- rule: publish.legacy.conversion-greater-version class=hard enforced=no tasks=client-runtime -->
A release with a valid format-2 marker and the older raw `schema.json` layout
is also classified as obsolete without downloading its old metadata bodies.
If the compiled publication still has the same immutable creation identity,
Studio appends gzip assets to that same tag and preserves the marker and old
assets. A changed creation identity requires a greater public version. The
schema gzip asset is uploaded after the other required assets, so
an interrupted conversion remains recognizable and can be retried normally.

Reconciliation has these outcomes:

- identical immutable schema/runtime content reuses the publication, exact
  target bindings, public version, marker, and tag;
- <!-- rule: publish.identity.no-target-rebinding class=hard enforced=no tasks=client-runtime --> a selected target with a different Hub `appVersion` never reuses or is
  rewritten to an older published binding, even when other content is equal;
- any schema change, including any exact `AppDependencies` pin change, or any
  real code/runtime change to an existing target creates a new publication
  after the strict public-version check above;
- a previously absent lower/equal-version target appends one immutable binding
  to the same publication, while a higher-version target creates the next
  publication; withdrawal changes latest availability but retains its binding,
  assets, and greatest-ever public version; <!-- rule: publish.version.document-only class=hard enforced=no tasks=client-runtime -->
- localized document, source-comment, function-comment, and Display Template
  changes update only
  the current content-addressed document locator, search presentation, and one
  new content-addressed target-manifest revision per affected target for the
  same publication; each `targetVersion` advances, while no target App version,
  public version, tag, marker, schema, runtime binding, or artifact changes; and
- an identical complete state is idempotent.

The content-addressed documents manifest is the canonical current locale
inventory. Removing a bad translation means omitting its entry from a newly
addressed manifest; it does not delete or overwrite target assets. <!-- rule: publish.documents.default-locale-remains class=hard enforced=no tasks=deployment-publication --> At least the
declared default locale must remain. <!-- /rule --> <!-- rule: publish.identity.immutable-asset-authority class=hard enforced=no tasks=client-runtime --> GitHub is the immutable asset authority;
Plaza independently rejects an in-place schema, dependency-lock, target
runtime binding, existing target-manifest asset, artifact, or public-version
change; advancing to a new content-addressed target-manifest revision is the
allowed document-only operation.


### Display Template identity

<!-- rule: publish.template.not-a-locale class=hard enforced=no tasks=client-runtime -->
Every package has one compiled [Display Template](presentation.md#display-template),
which is not part of the schema. It is a locale-independent member of the
content-addressed documents manifest with its own four-part content revision
and a SHA-256 digest of its canonical JSON; the empty template has both. It is
never counted as a locale, and removing a locale never removes it.

<!-- rule: publish.template.document-only-change.authoring class=hard enforced=no tasks=deployment-publication -->
Use Studio publication for generated assets and identities; [template-only changes preserve the App version and need no rebuild](#display-template-identity).

<!-- /rule -->

- <!-- rule: publish.template.document-only-change class=hard enforced=no tasks=client-runtime --> **Document-only change.** A template that differs from the Hub's is a
  document-only change, never a schema change. It advances `targetVersion`
  and preserves `appVersion`, schema, artifact, language, TypesOnly state, and
  distribution dependencies, with no rebuild. Publication follows the
  document-only reconciliation outcome above: no new App version, public
  version, tag, or marker.
- <!-- rule: publish.template.format-not-winner class=hard enforced=no tasks=client-runtime --> **Selection.** Publication takes the template with the highest content
  revision among the chosen Hubs, as for
  [documents](#preflight-and-latest-documents), and writes nothing to a Hub.
  `DisplayVersion` is the JSON format version and never chooses a winner.
- <!-- rule: publish.template.hub-storage-immutable class=hard enforced=no tasks=client-runtime --> **Hub storage.** The Hub retains templates by digest with the target
  manifests it already keeps and rejects a template that Studio did not compile
  from source. It never compiles or edits one.
- <!-- rule: publish.template.schema-delivery-pair class=hard enforced=no tasks=client-runtime --> **Delivery.** The Hub sends a package's template with its schema through the
  existing conditional delivery of cached resources, keyed by digest: an
  unchanged template costs nothing, and a changed one arrives without
  reinstalling the App. A client never receives a schema without its template,
  including for every exact dependency it fetches.

## Public release assets

The following hash and JSON examples describe generated artifacts. Placeholder
digests and sample sizes are schematic, not installable assets. <!-- rule: publish.assets.studio-owned class=hard enforced=no tasks=deployment-publication --> Studio owns
these files; App authors must not hand-edit them.

At creation Studio hashes the canonical target-key-sorted list of
`{target,appVersion,binarySha256}` as `targetCodeSha256`; a TypesOnly target uses
the SHA-256 of its empty code payload. It deliberately excludes target version,
source revision, build time, and documents. It then hashes
`{publicationVersion,schemaSha256,targetCodeSha256}` as
`releaseIdentitySha256`. The tag is
`libertas-{developer}-{package}-{release-identity-sha256}`. Source provenance,
function comments, and localized documents are absent from all identity hashes.
The GitHub Release title is `{developer}-{package} · {publicationVersion}` so
the public App version is visible in GitHub's release list. Studio also updates
the title when synchronizing an existing release; its immutable tag stays the same.

<!-- rule: publish.assets.revisions-immutable class=hard enforced=no tasks=client-runtime -->
Each current GitHub Release contains `libertas.release.json`, immutable `schema.json.gz`,
every immutable content-addressed target-manifest revision and executable
artifact, the current content-addressed documents manifest, and its
content-addressed locale assets. Published runtime bindings, schema, dependency
pins, artifacts, and individual target-manifest revisions are never overwritten
or removed by document synchronization; current manifest pointers may advance
to a newly published revision.

<!-- rule: publish.assets.metadata-only-manifest class=hard enforced=no tasks=client-runtime -->
The documents manifest contains metadata only, never embedded locale document
or template bodies. Its asset name includes its own lowercase SHA-256 as
`documents.<sha256>.json`; locale assets follow
`document.<locale>.<sha256>.json.gz` and the compiled Display Template follows
`display.<sha256>.json.gz`. Schema, locale documents, and the template are
precompressed with gzip level 9. The marker and target/document manifests stay
JSON. Every compressed descriptor's `sha256` and `size` describe the downloaded
bytes; `contentSha256` and `contentSize` describe the exact decoded UTF-8 JSON.
The release marker's `schemaSha256` continues to identify the decoded schema:


```json
{
  "defaultLocale": "en-US",
  "display": {
    "name": "display.<sha256>.json.gz",
    "sha256": "<64 lowercase hex digits>",
    "size": 64,
    "contentSha256": "<64 lowercase hex digits>",
    "contentSize": 48
  },
  "documents": [
    {
      "locale": "en-US",
      "name": "document.en-US.<sha256>.json.gz",
      "sha256": "<64 lowercase hex digits>",
      "size": 456,
      "contentSha256": "<64 lowercase hex digits>",
      "contentSize": 1234
    }
  ],
  "formatVersion": 3
}
```

<!-- rule: publish.assets.format-three-display-required class=hard enforced=no tasks=client-runtime -->
Format 3 requires the `display` member, including for the empty template.
Entries are sorted by locale and own the exact GitHub asset name, byte size,
and digest needed for a consumer to choose before downloading. Plaza atomically
advances a publication's document-manifest locator after all newly referenced
assets exist. Each target-manifest revision records the logical raw inventory
digest described above and its locale count. Old content-addressed documents and target manifests remain
and are never replaced in place, so a failed upload or retry cannot create a
missing-asset window.

```json
{
  "formatVersion": 2,
  "developerPackage": "developer-package",
  "publicationVersion": "20260820.143025.417.0",
  "schemaSha256": "sha256:<64 lowercase hex digits>",
  "targetCodeSha256": "sha256:<64 lowercase hex digits>"
}
```

The marker is immutable creation identity, including the public version and
creation target set. It is not a mutable inventory: later lower/equal-version
target additions are discovered from their retained immutable binding assets,
and withdrawals change only current availability. The tag and marker remain
stable during those operations and document-only synchronization, including
new `targetVersion` values and content-addressed target-manifest revisions.
Older marker formats are historical read-only input; new publication requests
use this format-2 contract.


## Publication completion and recovery

The Plaza service's database upgrade discards obsolete catalog packages,
publications, target bindings, functions, and search entries from older asset
formats while preserving accounts, repository authorizations, and unrelated
settings. New publication and catalog downloads use the gzip format only;
older document manifests are rejected. This upgrade does not delete GitHub
release assets. Studio can append current gzip assets to an existing release
with the same immutable creation identity without inventing a new App version.
The Hub preserves installed App data and automatically prepares missing gzip
artifacts from its retained raw metadata before serving clients.

Consumers verify compressed size and digest before decoding, enforce bounds
on both compressed and decoded content, then verify decoded content identity
and JSON before installation or presentation. A malformed gzip body fails;
there is no raw fallback. App schema, locale document, and template bodies are
limited to 16 MiB each in Plaza downloads. Their metadata manifest is limited
to 256 KiB. Installation uses the existing exact dependency graph and ordinary
Hub deployment path after decoding.

Plaza exposes a complete target set only after referenced public assets verify.
It has no user-visible staged or partially admitted target publication. GitHub
asset upload and Plaza admission are separate operations: a failed admission
can leave already uploaded public assets. Retry reuses the same content-addressed
snapshot; an upload failure does not change the Plaza catalog. Older referenced
assets remain available during metadata updates.

Withdrawing a target changes current availability, not retained identity or
assets. Latest discovery omits it; an exact historical lookup can still identify
it as unavailable. <!-- rule: publish.install.exact-dependency class=hard enforced=no tasks=deployment-publication --> Installation cannot substitute a different target or latest
version for an unavailable exact dependency. Re-adding the same retained binding
restores availability; changed runtime content follows the version rules above.
