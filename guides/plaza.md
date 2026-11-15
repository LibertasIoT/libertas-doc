# Finding, installing, and publishing Apps

Plaza helps people find functions and lets developers publish verified App
releases. Installing makes a package available on a Hub; creating a task
configures one function for a particular use. Publishing shares a developer's
release. Those operations have separate prerequisites and permissions.

## Browse installed Apps and search Plaza

Open Plaza for the intended Hub through its Admin entry, then choose **Installed**
or **Search Plaza**. [Plaza access](../reference/deployment.md#plaza-installation-access)
states the administrative and App permissions needed for Hub-assisted operations.
Installed packages are sorted by developer. A package with one public function
appears as that function; a package with multiple functions has a package root
and function children. TypesOnly packages remain visible in the App catalog.
The new-task chooser uses the same rows but omits packages without functions.

Each row shows the display name and the first two lines of the description.
Tap the row to open the full description and its available actions. Search
results also offer a direct button: **Install** for an absent package,
**Upgrade** when the installed version is older, or **Select** when the result's
version or a newer one is installed. Returning from details preserves Search
Plaza and the query. Selecting a function from the new-task chooser continues
to the task editor after any required installation/upgrade completes.

Filter Installed by display name, description, function, developer, package,
or ID. Search Plaza describes the capability you need and uses the selected
Hub's target compatibility. It requires a Hub supporting Plaza; Installed
remains available on Hubs without Plaza support. Studio still requires its
[supported Hub version](studio.md); legacy-Hub operation is mobile-only. Read the full function description before
installation to understand its prerequisites, access requests, and limits.

For a watering example, installing the App makes its watering function
available; **New Task** supplies the controller and schedule for one garden.
A TypesOnly dependency can support that function without offering its own task.
[Administration](administration.md#3-tasks-tab) explains task access checks.

## Developer account and repository

Run **Libertas: Open Libertas Plaza**. Studio opens a persistent center-editor
page at the fixed Plaza service, `https://plaza.smartonlabs.com`; there is no
custom service-URL setting. First-time setup has three steps:

1. **Plaza identity**: choose a permanent public lowercase ASCII `snake_case`
   Plaza ID, at most 32 bytes, and authorize GitHub. This ID may differ from your
   GitHub name or repository owner. Studio obtains the Plaza token automatically.
2. **App repository**: authorize GitHub repository access and explicitly select
   a public repository on which that identity has write access. You do not paste
   a token or type/guess a repository. Selection is required even with one result.
3. **Ready**: review the remembered Plaza ID, GitHub ID, and App repository.

Existing developers use **Sign in with GitHub**; sign-in never silently creates
an account and resumes at the first incomplete step. The public display name
starts from GitHub; the optional public company name is editable on the profile.
The verified primary email comes from GitHub and refreshes at authentication;
Studio does not let you type or override it.

Studio keeps credentials in VS Code Secret Storage. GitHub authorization covers
public repositories; Studio changes release assets only in the eligible
repository you select. It remembers the selection across reloads, cancellations,
failures, and successful publication until you change it or sign out.

After setup you can publish, change or refresh the repository, reauthenticate,
manage/revoke API tokens, or sign out. **Refresh GitHub authorization** updates
choices after repository creation, rename, transfer, or permission changes.
Private or read-only repositories are excluded. Initialize an empty repository
with at least one commit, such as a README, before publishing a release.
For token scope, expiry, descendant revocation, and the limits of repository
permission refresh, see
[account and publication authorization](../reference/publication.md#account-and-publication-authorization).

If GitHub rejects the saved publishing authorization with HTTP 401, Studio opens
repository authorization and retries the rejected request once. Identity sign-in
alone does not renew repository access. Publication stops if authorization fails,
the account or selected repository changes, or GitHub rejects the renewed access.
Your repository selection is retained; earlier successful publication steps remain.

## Publish a deployed App

Deploy the same source to every Hub whose platform you want to offer first.
Run **Libertas: Publish to Plaza**, or choose **Publish an App** on the ready
account page. The dedicated center document keeps Hub and App choice,
progress, errors, and completion in one place. Publishing does not compile,
build, deploy, or open a local project, and uploads no source: it publishes
what the chosen Hubs already hold.

If your App recommends another service App, publish that service separately
before sharing the caller. For example, Smart Sprinkler's TypesOnly Weather
dependency supplies the protocol types, while the Local Weather service App
supplies the function that creates a weather Endpoint. The service needs its
own deployed target and Plaza publication. Publish each App dependency before
the Apps that pin it. See
[package and target identity](../reference/publication.md#package-and-target-identity)
for how service recommendations differ from package dependencies.

1. **Hubs**: choose one or more registered Hubs. Each Hub contributes the
   deployed target of its own platform, for example a Raspberry Pi Hub for
   `arm64` and a cloud Hub for `x86_64`; choose one Hub per distribution and
   architecture. With only one registered Hub, Studio selects it. Complete any
   missing account or repository prerequisite in the page. Starting from the
   ready account page passes that exact selected repository through without
   asking again. The wizard shows it as the App storage repository, with
   **Change repository** on the Hub and App steps. The choice applies to this
   publication and all future ones; your Hub selection is kept, and **Keep
   current repository** changes nothing.
2. **App**: choose one of your Apps that is current on every chosen Hub. Only
   Apps whose developer is your Plaza ID are listed, with each Hub's App
   version. Choosing **Publish** starts the publication at once.
3. **Publish**: Studio reads each chosen Hub's deployment, Plaza history and the
   App's current GitHub release, then checks them before any change. Every
   chosen Hub must hold the identical App schema; if not, Studio names each
   Hub's schema digest and stops. It also stops on a duplicate platform, an
   exact dependency that is not on Plaza for a chosen platform, or a version
   conflict with earlier releases, and explains the fix. It then publishes every
   Hub's binary, one per platform, and for each locale the document with the
   highest version any chosen Hub holds, and likewise the newest Display
   Template. Completion shows the Plaza version and the published platforms.

The [publication reference](../reference/publication.md) owns exact version
identity, document selection, retained assets, and recovery. Publishing writes
nothing to your Hubs, and a release carries no translation companion. Studio opens GitHub repository authorization before publishing so
Plaza can verify recent authority for the calling token and selected
repository. Complete that handoff; signing into the website does not renew the
publishing token. If the authorization window expires during a long
operation, refresh authorization and retry the publication.

## Withdraw a platform target

Run **Libertas: Withdraw Plaza Target** in the local project and choose an
active target. It becomes unavailable through current discovery; retained
history and GitHub/Hub assets remain. To restore the same binding, publish it
again. Changed content or App version follows the
[immutable version rules](../reference/publication.md#app-publication-and-version-identity).
