# Reusable GitHub Actions

The current workflow contract is available from `main`. Consumers that have
not migrated to trusted publishing should remain on the `1.1.2` tag. The
`1.x` tags retain the previous API-key publishing behavior.

Repository for reusable GitHub actions.

## Automatic semantic version

We utilize the GitHub action [semantic-version](https://github.com/PaulHatch/semantic-version) to automatically raise package versions when creating releases. In short to the following:

+ if introducing new features, append `(MINOR)` to the commit message,
+ if introducing breaking changes, append `(MAJOR)` to the commit message,
+ otherwise do nothing.

The action will calculate the version number change like that:

+ any number of `(MAJOR)` since the last release > raise package version to new major (e.g. 5.0.0) and ignore other labels,
+ any number of `(MINOR)` but no `(MAJOR)` since last release > raise package feature version by one (e.g. 5.0.1 becomes 5.1.0),
+ no `(MAJOR)` or `(MINOR)` since last release > raise only patch version by one (e.g. 5.0.1 becomes 5.0.2)

## Action `build-and-pack.yml`

Builds, tests and packs the source code as NuGet package. It's assumed that all source files reside in a directory `src` inside the root of the repository.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`configuration`|Define whether this is a debug or a release build.|yes|`string`|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|
|`is_prerelease`|Define if this is a prerelease.|yes|`boolean`|
|`suffix`|Define the prerelease suffix, e.g. alpha.|yes|`string`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|10.0.x|

## Action `create-release.yml`

`create-release.yml` uses [release-action by ncipollo](https://github.com/ncipollo/release-action) to automatically generate release notes from the commits since the latest release and new contributors if there are any.

> Keep in mind that creating a release will create a support branch at the position of the last version tag, if you introduced any breaking changes since then. Therefore it is recommended to run `create-release.yml` if you want a release that contains all your features and bug fixes for the last major version., before merging a PR with breaking changes.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`generate_release_notes`|Generates release notes from the commits since the last release and new contributors and adds them to the GitHub page.|no|`boolean`|`false`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|10.0.x|

## Action `develop.yml`

Workflow to build, test, and package the develop branch.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|10.0.x|

## Action `feature-branch.yml`

Workflow to build, test, and package feature/fix branches.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|10.0.x|

## Action `gitversion.yml`

This workflow uses [semantic-version by paulhatch](https://github.com/PaulHatch/semantic-version) to automatically raise major, minor and fix version numbers on respective changes. To use this functionality include `(MAJOR)` or `(MINOR)` in your commit message if you introduce breaking changes (`(MAJOR)`) or new features (`(MINOR)`). The fix version will be raised automatically if there are any changes between releases.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`is_prerelease`|Define if this is a prerelease.|yes|`boolean`|none|
|`suffix`|Define the prerelease suffix, e.g. alpha.|yes|`string`|`""`|

### Outputs

|Output name|Description|Type|
|-----------|-----------|----|
|`package_version`|Computed version for the package (SemVer).|`string`|
|`semver_raise_type`|The type of version raise (major,minor,patch).|`string`|

## Action `deploy-gh-pages.yml`

Build GitHub Pages content using Jekyll and deploy the artifactes to the GitHub Pages of your repository.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`source`|The directory to build GitHub Pages from.|no|`string`|`./`|
|`destination`|The directory to build GitHub Pages to.|no|`string`|`./_site`|

## Secrets

NuGet publication is not performed by the reusable workflows. They build and
upload the `packages` artifact. Consumers should publish from a workflow job
defined in their own repository using NuGet trusted publishing:

* `NUGET_USER` is the NuGet profile name used by `NuGet/login@v1`; it is not an
  API key or email address.
* The caller's publish job must grant `id-token: write`.
* The caller-owned job should run `NuGet/login@v1` and `dotnet nuget push`
  directly. NuGet trusted-publishing policy matching currently does not work
  reliably when `NuGet/login@v1` runs inside a reusable workflow; see
  [NuGet/login issue #6](https://github.com/NuGet/login/issues/6).

`create-release.yml` and `create-support-release.yml` create commits, merge
branches, create tags, and push to the target repository. They require a
GitHub personal access token named `PUSH_TO_GITHUB_REPO_PAT` with the
permissions required by the consuming repository. The calling job must pass
this secret to the reusable workflow, as shown in the
[release](Examples/create-release.yml) and
[support release](Examples/create-support-release.yml) examples.
