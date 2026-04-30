---
title: Packaging Rules
description: Reference of all authoritative Garden Linux Packaging Rules
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
github_source_path: docs/reference/packaging-rules.md
github_target_path: docs/reference/packaging-rules.md
---

# Packaging Rules

This reference document outlines the authoritative rules for creating and maintaining Garden Linux packages. These rules ensure consistency, security, and maintainability across the package ecosystem.

## Package Repository Naming

To ensure clear identification and standardized tooling, package repositories follow a strict naming convention.

### Rule 1: Package repositories must be named accordingly

```
package-<package-source>
```

- **Must start with** `package-`
- `<package-source>` **must be** the name of the source package as defined in Debian
- **Exception**: If the package does not exist in Debian, use the upstream source name

**Examples**:

- `package-containerd` - Containerd from Debian
- `package-openssl` - OpenSSL from Debian
- `package-metalbond` - Metalbond (not in Debian)
- `package-iproute2` - iproute2 from Debian

### Rule 2: One-shot build dependency repositories must start with bp-package

Package repositories required only as a build dependency for another backported package must start with `bp-package-*`.

**Examples**:

- `bp-package-xyz-0.0.1` - Build dependency only needed for package-ABC
- `bp-package-libfoo-1.2.3` - Build dependency for a backported library

## Git Branch Naming

Branch names in package repositories follow standardized patterns to indicate their purpose and compatibility.

### Rule 3: Git branches must be named accordingly

**Branch types**:

- `main`: Builds against the latest Garden Linux environment
- `rel-<MAJOR>`: Builds against `<MAJOR>` version of Garden Linux (e.g., `rel-2150`)
- `fix/*`, `feat/*`, `chore/*`, `doc/*`: Temporary branches for work in progress

**Branch purpose**:

- The `main` branch is used for regular builds and nightly releases
- `rel-<MAJOR>` branches are used for backporting packages to specific major versions
- Temporary branches (`fix/*`, `feat/*`) are for development work and should be deleted after merging

## Source Acquisition

### Rule 4: Get debian/ folder from Garden Linux snapshot apt repository

For existing Debian packages, obtain the `debian/` folder from the Garden Linux snapshot apt repository.

The `apt_src` helper function must be used in the `prepare_source` script:

```bash
apt_src --ignore_orig <source_package_name>
```

:::tip
All Debian packages from testing are mirrored daily in a Garden Linux snapshot apt repository. If a source package is missing in a snapshot, the Debian salsa repository may be used instead.
:::

**Recommended workflow**:

1. Check `https://packages.gardenlinux.io/debian-release.snapshot/` for the package
2. If present, use `apt_src`
3. If not present, check salsa.debian.org for current debian/ folder
4. Use `git_src` to get from salsa if needed

## Package Creation

### Rule 5: Create debian/ folder only if Debian package does not exist

If there is **no existing Debian package**, the `debian/` folder must be created manually and checked into the `package-` repository.

This applies to:

- Packages not available in Debian at all
- Upstream-only software
- Custom patches that significantly alter the package

The `debian/` folder should include:

- `control` file with package metadata
- `copyright` file
- `rules` file
- `source/format`
- Appropriate patches in `patches/`

## Upstream Source

### Rule 6: Get upstream source from upstream git repository

To enable automatic upstream version tracking, get source code directly from the upstream git repository, not from Debian source packages or patches.

This allows automatic triggering of builds when new upstream versions are available without waiting for Debian maintainers to update salsa.

The `git_src` helper function must be used in the `prepare_source` script:

```bash
version_orig=<VERSION>
git_src --branch <BRANCH_NAME> <GIT_REPO_URL>
```

**Examples**:

```bash
# OpenSSL from upstream git
git_src --branch "openssl-$version_orig" https://github.com/openssl/openssl.git

# iproute2 from Debian salsa
git_src -b "debian/$version" https://salsa.debian.org/debian/iproute2.git
```

## Patching

### Rule 7: Patching the debian/ folder

Changes to files in the `debian/` folder should be made as patches, not by direct edits. The `debian/patches` folder is treated as source code.

Patches for the debian folder should be placed in a `fixes_debian` folder and applied with the `apply_patches` helper function.

**Example**:

```bash
# In prepare_source script
apply_patches fixes_debian
```

**Workflow to create a debian patch**:

1. Build sources with `build --leave-artifacts`
2. Spawn patch environment with `build --edit`
3. Make changes in the `b` directory using quilt
4. Create patch with `diff -Naur a/debian b/debian > fixes_debian/your-fix.patch`
5. Add patch to `fixes_debian/series`

### Rule 8: Append to debian/patches for upstream patches

Patches for upstream source code should be placed in an `upstream_patches` folder and applied using the `import_upstream_patches` helper function.

This function copies patches to `debian/patches` and appends them to `debian/patches/series`, enabling easy maintenance of patches from upstream sources.

**Example**:

```bash
# In prepare_source script
import_upstream_patches# upstream_patches
```

:::tip
This approach allows convenient import and maintenance of patches from upstream (e.g., cherry-picking an upstream commit on a different branch to fix a CVE).
:::

## Related Topics

<RelatedTopics />
