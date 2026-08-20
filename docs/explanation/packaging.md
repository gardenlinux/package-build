---
title: Packaging
description: Understand how Garden Linux packages are built and built using the package-build tools
order: 101
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
github_source_path: docs/explanation/packaging.md
github_target_path: docs/explanation/packaging.md
---

# Packaging Ecosystem

This document explains how Garden Linux packages are built using the tools in the `package-build` repository and how they fit into the broader packaging ecosystem.

## Release hierarchy

Garden Linux uses a [three-tier release hierarchy](/explanation/release-hierarchy.md) to deliver a complete operating system.

This document is about the first tier, the [Packaging](/explanation/packaging).

## Package Build Tools Repository (`package-build`)

The [`package-build`](https://github.com/gardenlinux/package-build) repository provides:

- Standardized build scripts for creating Debian packages
- GitHub Actions workflows for automated building
- Common functions and utilities used by all package repositories
- Documentation for local and CI-based package building

## Source Package Repositories (`package-*`)

Each custom-built package has its own GitHub repository following the naming convention `package-{package_name}` (for example, `package-containerd`, `package-openssh`). These repositories contain:

- The source code or references to upstream sources
- Build scripts and configuration
- Debian packaging files (debian/ directory)
- Package-specific build instructions

These repositories are built using the tools in the `package-build` repository.

## Package flow process overview

1. **Source Management**: Package source lives in `package-{package_name}` repos
2. **Building**: Packages are built using tools from `package-build` repo
   - Can be done locally or via GitHub Actions
3. **Release Assembly**: Built packages are collected and assembled into APT repositories by the `repo` workflow
4. **Distribution**: Final APT repository is published and made available to users

## Key relationships

- Each `package-*` repo declares its build dependencies in its source
- The `package-build` repo provides common tooling used by all `package-*` repos
- The [`repo`](https://github.com/gardenlinux/repo) orchestrates the final release by collecting built packages and dependencies
- Users access packages through the standard APT system pointing to Garden Linux repositories

## Understanding package versions

Garden Linux packages follow versioning like:

```
<upstream_version>-<debian_revision>gl<gl_increment>[+bp<garden_linux_major>]
```

For example: `8.21.0-2gl0+bp2150`

Where:

- `8.21.0` is the upstream version
- `-2` is the Debian revision
- `gl0` is the Garden Linux build increment, starting at `gl0` (`gardenlinux0` for older releases) and incremented for each new GL rebuild of the same upstream+Debian version (for example, `gl1`, `gl2`)
- `+bp2150` identifies the Garden Linux major release this build targets (2150.x in this example)

### Version suffix and branch model

Each `package-*` repository uses a multi-branch model:

| Branch | Purpose | Tag suffix |
|---|---|---|
| `main` | Nightly / development track. Feeds daily GL snapshot releases. | None — tags like `8.21.0-2gl0` |
| `rel-<N>` | Maintenance branch for a specific supported GL major release `N`. | `+bp<N>` — tags like `8.21.0-2gl0+bp2150` |

The `version_suffix` variable in the `prepare_source` script controls which suffix the build system appends:

- On `main`, the CI auto-appends `gl0` when `release: true` is set in the workflow input.
- On `rel-<N>` branches, `version_suffix` is set explicitly in `prepare_source`, for example `version_suffix=gl0+bp2150`.

The build system reads `version_suffix` to form the package version and the git tag. When the same upstream version is rebuilt (for example, to fix a build dependency), the increment is bumped: `gl0` → `gl1`.

### Examples using `package-curl`

| Tag | Branch | Meaning |
|---|---|---|
| `8.21.0-2gl0` | `main` | First GL build of curl `8.21.0-2`; nightly |
| `8.21.0-2gl0+bp1877` | `rel-1877` | Backport of curl `8.21.0-2` for GL 1877.x |
| `8.21.0-2gl0+bp2150` | `rel-2150` | Backport of curl `8.21.0-2` for GL 2150.x |
| `8.20.0-2gl1+bp2150` | `rel-2150` | Second Backport of curl `8.20.0-2` for GL 2150.x |

::: note Version encoding

The `+` character in git tags corresponds to `~` (tilde) in the Debian package version string stored inside `prepare_source` (for example, `version_suffix=gl0~bp2150`). The tilde sorts lower in Debian version comparison, which ensures the backport version sorts below the corresponding nightly version in APT. The build system translates `~` to `+` when creating the git tag.

:::

## Related topics

<RelatedTopics />
