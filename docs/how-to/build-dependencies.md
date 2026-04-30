---
title: Build Dependencies
description: Learn how to create and manage one-shot build dependencies
order: 6
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
github_source_path: docs/how-to/build-dependencies.md
github_target_path: docs/how-to/packaging/build-dependencies.md
---

# Build Dependencies

This guide covers how to create and manage one-shot build dependencies for Garden Linux packages using bp-package repositories.

## What are Build Dependencies?

Some packages require specific versions of library or tool dependencies that are not available in the base Garden Linux environment or in Debian. These are called **build-time dependencies** - they are only needed during the compilation process, not at runtime.

In Garden Linux, we use **bp-package repositories** ("build-package") to handle these special build dependencies. These repositories are one-shot solutions that provide exactly what's needed for a specific package build.

## When to Use bp-package Repositories

Use a bp-package repository when:

- A package (ABC) requires a build dependency (XYZ) in a specific version
- The required version of XYZ is not available in the base Garden Linux environment
- The dependency is only needed for building, not for runtime
- You need to apply specific patches to the dependency before building
- The dependency is temporary or experimental

**Examples**:

- Building a package against a newer version of a library
- Applying security patches to a build tool
- Using a forked version of a dependency
- Experimental features requiring custom dependencies

## Creating a bp-package Repository

To create a bp-package repository:

1. Create a new repository with the naming convention `bp-package-<name> <version>`
2. Follow the same packaging rules as regular packages
3. Build and release the package to GitHub

The bp-package repository is **not** included in the `package-releases` file of gardenlinux/repo, but it is included in the build of the dependent package.

:::danger
In rescent Garden Linux Release, some `bp-*` packages ended up in the `package-releases` files. Please be aware of this.
:::

## Using Build Dependencies in Workflows

Specify the build dependency in the GitHub Actions workflow of the main package using the `build_dep` parameter.

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
      build_dep: gardenlinux/package-XYZ 0.0.1-0gl0+bpwhatever
```

**Parameters**:

- `build_dep`: Specifies the dependency repository and version in the format `<repo> <tag>`
- The dependency is automatically fetched and made available during the build

## Complete Example

For a package that requires a patched version of libtool:

1. Create `bp-package-libtool` repository with the patched version
2. Release version `0.0.1-0gl0+bp1`
3. In the main package workflow, add:

```yaml
build_dep: gardenlinux/bp-package-libtool 0.0.1-0gl0+bp1
```

The build system will automatically fetch the dependency and use it during compilation.

## Important Notes

:::warning
Build-time dependencies between packages are not updated automatically and need to be adjusted manually when needed.
:::

:::info
bp-package repositories should be kept minimal and focused only on providing the necessary build dependency. They are not intended for general use or long-term maintenance.
:::

:::tip
For widely-used dependencies that will be needed by multiple packages, consider adding them to the base Garden Linux environment instead of using bp-package.
:::

## Related Topics

<RelatedTopics />
