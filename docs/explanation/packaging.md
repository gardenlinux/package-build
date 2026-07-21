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

Each custom-built package has its own GitHub repository following the naming convention `package-{package_name}` (e.g., `package-containerd`, `package-openssh`). These repositories contain:

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

Garden Linux packages typically follow versioning like:

```
<upstream_version>-<debian_revision>~gardenlinux<sequence>
```

For example: `1.2.3-1~gardenlinux0`

Where:

- `1.2.3` is the upstream version
- `-1` is the Debian revision
- `~gardenlinux0` indicates this is the first Garden Linux rebuild

When creating patch releases or backports, the suffix is incremented (e.g., `~gardenlinux1`, `~gardenlinux2`) or modified for specific use cases (like `~bp1443` for backports).

## Related topics

<RelatedTopics />
