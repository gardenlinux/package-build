---
title: Github Actions Package Building
description: Learn how Garden Linux Packages are built in Github Actions
order: 4
related_topics:
  - /explanation/release-hierarchy.md
  - /explanation/packaging.md
  - /explanation/package-sources.md
  - /explanation/repo-infrastructure.md
  - /explanation/os-releases.md
  - /explanation/semver.md
  - /how-to/releases/package-releases.md
  - /how-to/packaging/local-build.md
  - /how-to/packaging/github-actions.md
  - /how-to/packaging/creating-packages.md
  - /how-to/packaging/backporting.md
  - /how-to/packaging/patching.md
  - /how-to/packaging/build-dependencies.md
  - /how-to/packaging/null-releases.md
  - /reference/packaging-rules.md
migration_status: "done"
migration_issue: "https://github.com/gardenlinux/gardenlinux/issues/4705"
migration_stakeholder: "@tmangold, @yeoldegrove, @ByteOtter"
migration_approved: false
github_org: gardenlinux
github_repo: package-build
github_source_path: docs/how-to/github-actions.md
github_target_path: docs/how-to/packaging/github-actions.md
---

# GitHub Actions Package Building

This guide covers how to build Garden Linux packages using GitHub Actions workflows.

## Workflow Configuration

To build packages using GitHub Actions, define a job that uses the package-build workflow:

```yaml
jobs:
  build:
    uses: gardenlinux/package-build/.github/workflows/build.yml@main
```

## Job Inputs

The GitHub Actions workflow accepts several inputs to customize the build process.

### `release` (boolean)

Flag to indicate if this is a release build. When set to `true`:

- Automatically appends `gl0` as the version suffix
- Creates a GitHub release from the resulting source and binary packages

Example usage:

```yaml
with:
  release: ${{ github.ref == 'refs/heads/main' }}
```

### `build_dep` (string)

A list of other GitHub repositories to pull custom build-time dependencies from, in the format `<repo> <tag>`.

:::warning
Build-time dependencies between packages are not updated automatically and need to be adjusted manually when needed.
:::

### `runs-on`, `runs-on-amd64`, `runs-on-arm64` (string)

Specify the GitHub Actions runner on which to execute the job.

## Complete Example

A full GitHub workflow file that builds a package and periodically checks for new versions:

```yaml
on:
  push:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * *"
jobs:
  build:
    uses: gardenlinux/package-build/.github/workflows/build.yml@main
    with:
      release: ${{ github.ref == 'refs/heads/main' }}
```

## Related topics

<RelatedTopics />
