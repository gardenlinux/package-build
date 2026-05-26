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

# Package Repository Conventions
## naming
If a package repository is called package-* and if it has the build.yml workflow then it will be built every night. Sometimes one doesn't want that and then the repository name is prefixed differently, most likely bp-package-something.

## branches within package-* repositories
Each package repository should have `rel-*` branches that relate to the Garden Linux version for which this package is being built. This is to ensure the correct versions are being built with the correct dependencies for the correct Garden Linux versions. For example `package-openssh` could be built in one version for Garden Linux 1877 and another version for Garden Linux 2150. This means there should be branches called `rel-1877` and `rel-2150`. Packages should never be built main. Although it is a convention and not enforced.

Also important on the `rel-*` branches is the `.container` file

## .container file
which includes the hash of the Garden Linux repo at the day of the release of the Major Garden Linux release, for example:
```$ wget -qO- "https://raw.githubusercontent.com/gardenlinux/repo/refs/tags/1877.0/.container"
ghcr.io/gardenlinux/repo-debian-snapshot@sha256:0348e829222f0e62c61cb68e630f1766b62e4311303bc35b02cbd5c62a98abef
$ wget -qO- "https://raw.githubusercontent.com/gardenlinux/repo/refs/tags/1877.1/.container"
ghcr.io/gardenlinux/repo-debian-snapshot@sha256:0348e829222f0e62c61cb68e630f1766b62e4311303bc35b02cbd5c62a98abef
$ wget -qO- "https://raw.githubusercontent.com/gardenlinux/repo/refs/tags/1877.2/.container"
ghcr.io/gardenlinux/repo-debian-snapshot@sha256:0348e829222f0e62c61cb68e630f1766b62e4311303bc35b02cbd5c62a98abef```
it is crucial to pin down the build environment in that way with which the package is being built. Otherwise packages from debian testing at the point of time of when a package is being built are being used. Which might even mean a package doesn't build anylonger without a code change or doesn't work within the same Garden Linux release any longer for example there could be a different `libc` version in debian testing at that point as opposed to at release time.



## Related Topics

<RelatedTopics />
