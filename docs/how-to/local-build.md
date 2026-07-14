---
title: Local Package Building
description: Learn how Garden Linux Packages are built locally
order: 3
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
github_source_path: docs/how-to/local-build.md
github_target_path: docs/how-to/packaging/local-build.md
---

# Local package building

This guide covers how to build Garden Linux packages locally using the `package-build` tools.

## Prerequisites

- Git installed on your system
- Sufficient disk space for builds
- Access to the Garden Linux package repositories

## Build process

### 1. Clone the package-build repository

```bash
git clone https://github.com/gardenlinux/package-build.git
```

### 2. Clone the target package repository

Choose the package you want to build and clone its repository. For example, to build iproute2:

```bash
git clone https://github.com/gardenlinux/package-iproute2.git
```

### 3. Run the build script

Execute the build script from the package-build repository, passing your package directory as an argument:

```bash
./package-build/build package-iproute2
```

:::info
If your package build depends on other custom build packages, you can provide the `--build-dependencies` flag with a directory containing the `.deb` files of build-time dependencies.
:::

## Build artifacts

The build artifacts (`.deb` files and others) are placed in a `.build` directory within your package repository. For example: `./package-iproute2/.build/`.

## Build options

The `build` script supports several options to customize the build process:

- `--arch amd64`: Specify the architecture to build (default: `amd64`, choices: `amd64`, `arm64`)
- `--source-only`: Build only the source archive
- `--binary-only`: Build only the binary archives
- `--leave-artifacts`: Create the sources folder and keep them in a `package-XYZ/output/run_<date_time>` folder of the local `package-XYZ` directory
- `--build-dependencies`: Path to a directory containing `.deb` files to be used as build-time dependencies
- `--edit`: Spawn a gardenlinux/repo-debian-snapshort container with `package-XYZ/output` mounted, quilt installed and configured

## Related topics

<RelatedTopics />
