# Reusable GitHub Actions

Repository for reusable GitHub actions.

## Action `build-and-pack.yml`

Builds, tests and packs the source code as NuGet package. It's assumed that all source files reside in a directory `src` inside the root of the repository.

### Inputs

|Input name|Description|Required|Type|
|----------|-----------|--------|----|
|`configuration`|Define whether this is a debug or a release build.|yes|`string`|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|
|`is_prerelease`|Define if this is a prerelease.|yes|`boolean`|
|`suffix`|Define the prerelease suffix, e.g. alpha.|yes|`string`|
|`publish_target`|Define the publish target (None/nuget.org).|yes|`string`|

## Action `create-release.yml`

`create-release.yml` uses [release-action by ncipollo](https://github.com/ncipollo/release-action) to automatically generate release notes from the commits since the latest release and new contributors if there are any.

> Keep in mind that creating a release will create a support branch at the position of the last version tag, if you introduced any breaking changes since then. Therefore it is recommended to run `create-release.yml` if you want a release that contains all your features and bug fixes for the last major version., before merging a PR with breaking changes.

### Inputs

|Input name|Description|Required|Default value|
|----------|-----------|--------|-------------|
|`generate_release_notes`|Generates release notes from the commits since the last release and new contributors and adds them to the GitHub page.|no|`false`|

## Action `develop.yml`

Workflow to build and publish the develop branch.

### Inputs

|Input name|Description|Required|Type|
|----------|-----------|--------|----|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|

## Action `feature-branch.yml`

Workflow to build and publish feature/fix branches.

### Inputs

|Input name|Description|Required|Type|
|----------|-----------|--------|----|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|

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

## Action `publish.yml`

Publish the build artifact (NuGet package) to a target nuget feed.

### Inputs

|Input name|Description|Required|Type|
|----------|-----------|--------|----|
|`target`|Define the target feed to push the artifact (e.g. nuget.org).|yes|`string`|

## Secrets

Two of the workflows require secrets to be passed on to work as intended:

* `publish.yml` pushes NuGet packages to nuget.org and needs a personal access token named `NUGET_FEED_PAT` with proper permissions
* `create-release.yml` creates commits, merges and creates branches and pushes to the target repository and need a GitHub personal access token name `PUSH_TO_GITHUB_REPO_PAT` with proper permissions
  - the only permission to be set is `public_repo`