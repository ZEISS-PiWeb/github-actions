# github-actions
Repository for re-usable GitHub actions.

Two of the workflows require secrets to be passed on to work as intended:

* `publish.yml` pushes NuGet packages to nuget.org and needs a personal access token named `NUGET_FEED_PAT` with proper permissions
* `create-release.yml` creates commits, merges and creates branches and pushes to the target repository and need a GitHub personal access token name `PUSH_TO_GITHUB_REPO_PAT` with proper permissions
  - the only permission to be set is `public_repo`