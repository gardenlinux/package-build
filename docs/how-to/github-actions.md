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

## Release branches (`rel-*`)

Each `package-*` repository uses a [multi-branch model and version suffixes](/explanation/packaging#Version-suffix-and-branch-model) to separate nightly development from stable release maintenance.

### How the workflow differs on `rel-<N>` branches

There are two key differences compared to `main`:

**1. The `release` input is always `false` for pushes to `rel-<N>`.**

The condition `${{ github.ref == 'refs/heads/main' }}` evaluates to `false` on a push to `rel-2150`. This is intentional: on `rel-<N>` branches, the `version_suffix` is already set explicitly in `prepare_source` (for example, `version_suffix=gl0+bp2150`). The build system reads that suffix and creates the tag and GitHub release automatically, without needing `release: true` to append the suffix.

**2. `build_dep` pins dependencies to the same `+bp<N>` versions.**

On `main`, [build dependencies](/how-to/packaging/build-dependencies) are not usually pinned. On a `rel-<N>` branch, other packages may also exist only as backported `+bp<N>` releases that are not in the default nightly apt repository. The `build_dep` input lists these explicit dependencies:

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
      build_dep: |
        gardenlinux/bp-package-ngtcp2 1.22.1-1gl0+bp2150
        gardenlinux/package-nghttp2 1.68.1-1gl0+bp2150
        gardenlinux/package-openssl 3.5.5-1gl21+bp2150
        gardenlinux/package-gnutls 3.8.13-1gl0+bp2150
```

This example is taken from [`package-curl` on `rel-2150`](https://github.com/gardenlinux/package-curl/blob/e8f171157619e6ee39e75e42dab097e41eb32590/.github/workflows/build.yml).

## Related topics

<RelatedTopics />
