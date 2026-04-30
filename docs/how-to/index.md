---
title: Packaging
description: Comprehensive guide to the Garden Linux packaging system and release lifecycle
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
github_source_path: docs/how-to/index.md
github_target_path: docs/how-to/packaging/index.md
---

# Packaging

The Garden Linux packaging system enables the creation of customized operating system images through a modular approach. This guide provides an overview of the complete package release lifecycle and links to detailed how-to guides for each stage.

## Release Hierarchy

Garden Linux uses a [three-tier release hierarchy](/explanation/release-hierarchy.md) to deliver a complete operating system.

This document is about the first tier, the [Packaging Ecosystem](/how-to/releases/package-releases).

## Package Release Lifecycle

Creating and releasing packages in Garden Linux follows a structured process:

1. **[Creating Packages](/how-to/packaging/creating-packages.md)**: Start with this guide to learn how to create a new package or modify an existing one
2. **[Applying Patches](/how-to/packaging/patching.md)**: Learn how to create and manage patches for both debian/ folder changes and upstream source modifications
3. **Building Packages**: Choose your preferred method:
   - **[Local Builds](/how-to/packaging/local-build.md)**: Build packages on your development machine
   - **[GitHub Actions](/how-to/packaging/github-actions.md)**: Automate builds with CI/CD
4. **[Backporting](/how-to/packaging/backporting.md)**: Create patch releases for specific Garden Linux versions
5. **[Managing Dependencies](/how-to/packaging/build-dependencies.md)**: Handle special build-time dependencies with bp-package repositories
6. **[Excluding Packages](/how-to/packaging/null-releases.md)**: Use NULL releases to temporarily exclude problematic packages from builds

## Core Concepts

- **[Packaging Rules](/reference/packaging-rules.md)**: Authoritative reference for all naming conventions and procedural rules
- **[Package Sources](/explanation/package-sources.md)**: Understand the different types of package sources in Garden Linux
- **[Repository Infrastructure](https://github.com/gardenlinux/repo/blob/main/docs/explanation/infrastructure.md)**: Learn how packages are assembled into apt repositories

## Getting Started

1. Review the [Packaging Rules](/reference/packaging-rules.md) for naming conventions and repository structure
2. Choose a package to work on (either a new package or modifying an existing one)
3. Follow the [Creating Packages](/how-to/packaging/creating-packages.md) guide to set up your package
4. Use the appropriate build method (local or GitHub Actions)
5. Test your package thoroughly before releasing
6. Create a GitHub release to make it available for inclusion in Garden Linux

For maintainers creating packages for the first time, we recommend reading through all the guides in order to understand the complete workflow.
