# Reusable GitHub Actions

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
|`publish_target`|Define the publish target (None/nuget.org).|yes|`string`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|6.0.x|

## Action `create-release.yml`

`create-release.yml` uses [release-action by ncipollo](https://github.com/ncipollo/release-action) to automatically generate release notes from the commits since the latest release and new contributors if there are any.

> Keep in mind that creating a release will create a support branch at the position of the last version tag, if you introduced any breaking changes since then. Therefore it is recommended to run `create-release.yml` if you want a release that contains all your features and bug fixes for the last major version., before merging a PR with breaking changes.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`generate_release_notes`|Generates release notes from the commits since the last release and new contributors and adds them to the GitHub page.|no|`boolean`|`false`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|6.0.x|

## Action `develop.yml`

Workflow to build and publish the develop branch.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|6.0.x|

## Action `feature-branch.yml`

Workflow to build and publish feature/fix branches.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`do_pack`|If this build should be packed as NuGet package.|yes|`boolean`|
|`dotnet_version`|The .NET SDK version that should be used by the runner|no|`string`|6.0.x|

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

## Action `deploy-gh-pages.yml`

Build GitHub Pages content using Jekyll and deploy the artifactes to the GitHub Pages of your repository.

### Inputs

|Input name|Description|Required|Type|Default value|
|----------|-----------|--------|----|-------------|
|`source`|The directory to build GitHub Pages from.|no|`string`|`./`|
|`destination`|The directory to build GitHub Pages to.|no|`string`|`./_site`|

## Secrets

Two of the workflows require secrets to be passed on to work as intended:

* `publish.yml` pushes NuGet packages to nuget.org and needs a personal access token named `NUGET_FEED_PAT` with proper permissions
* `create-release.yml` creates commits, merges and creates branches and pushes to the target repository and need a GitHub personal access token name `PUSH_TO_GITHUB_REPO_PAT` (classic token with the following permissions: `public_repo`, `read:user` and `user:email`)
