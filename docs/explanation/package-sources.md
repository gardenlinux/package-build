---
title: Package Sources
description: Learn about different types of Garden Linux Package Sources
order: 102
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
github_source_path: docs/explanation/package-sources.md
github_target_path: docs/explanation/package-sources.md
---

# Package Sources

Garden Linux packages are built using [the `package-build` tools](https://github.com/gardenlinux/package-build).

Each package has its own GitHub repository following the naming convention `package-{package_name}`. These packages are picked up by [the workflows of the 'repo'](https://github.com/gardenlinux/repo).

There are different types of package sources used in Garden Linux:

## Source from Debian

Packages built from Debian sources are used when:

- Applying Garden Linux-specific patches on top of existing Debian packages
- Rebuilding with a specific build environment (e.g., newer versions of golang, glibc, gcc) when a newer compiler or runtime is needed
- A package we include was temporarily removed from Debian testing (this happens occasionally and is expected)

## Source from Upstream Project

Packages built from upstream sources are used when:

- Debian does not maintain a package for the software we need
- Debian maintains only an older version, and we need to create a package for a newer version available in the upstream project

## Native Source

Packages with native sources are used when:

- The source code is developed and maintained by the Garden Linux developers, so Debian does not package it

## Related Topics

<RelatedTopics />
