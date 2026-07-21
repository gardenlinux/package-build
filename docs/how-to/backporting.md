---
title: Patch Releases and Backporting
description: Learn how to update and backport Garden Linux Packages
order: 5
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
github_source_path: docs/how-to/backporting.md
github_target_path: docs/how-to/packaging/backporting.md
---

# Patch Releases and Backporting

This guide covers how to create patch releases and backport packages in Garden Linux.

For foundational knowledge on package creation and repository structure, see the [Packaging Rules](/reference/packaging-rules.md) reference.

## Creating patch releases

When running the GitHub Actions job with `release: true`, it automatically creates a new release with version suffix `gl0` (or `gardenlinux0` in older packages). To create a patch release for this version:

### 1. Check out the release tag

```bash
git fetch --tags
git checkout <VERSION>gl0
git branch <VERSION>
git checkout <VERSION>
```

### 2. Apply modifications

Make the necessary changes or backport patches from the main branch. For detailed patching guidance, see the [Patching](/how-to/packaging/patching.md) guide.

### 3. Increment the version suffix

The GitHub Actions job that created the `<VERSION>gl0` tag automatically added a `version_suffix=gl0` line to the `prepare_source` script. Simply increment this suffix.

In certain scenarios where you want to backport to an older Garden Linux version, you will also need to update the `.container` file and use the appropriate container image.

:::tip
You can use the [`bin/find-build-container-for`](https://github.com/gardenlinux/gardenlinux/blob/016c3889e20cb4bd937da4338f88c380cd9a49be/bin/find-build-container-for) script to find the correct container image.

e.g.

```bash
bin/find-build-container-for 2150.0.0
ghcr.io/gardenlinux/repo-debian-snapshot@sha256:99f72494ab45d33958a0385054de4742c7022ab07b4c0c2cef6f785f8ebfb378
```

:::

### Optional: Build Locally

To check if your package builds before pushing it, refer to the [local Build How-To](/how-to/packaging/local-build.md).

### 4. Push the branch

```bash
git push origin <VERSION>
```

If the action is set up to run on `push` (as in the basic example), this will trigger the build job. The job will detect that this is a new version and create the necessary tags and GitHub releases.

## Backporting examples

### Backporting new upstream version not available (yet) on Salsa

When backporting a version not available (yet) in Debian salsa (e.g., OpenSSL 3.1.7), configure the `prepare_source` script as follows:

```bash
version_orig=3.1.7
version="$version_orig-0"
git_src --branch "openssl-$version_orig" https://github.com/openssl/openssl.git
apt_src --ignore-orig openssl
version_suffix=gl0~bp1443
```

This configuration fetches source files from the upstream repository while using the Debian folder from the apt source package. If there are compatibility issues, patches may need to be added using the `apply_patches` function.

### Backporting package already in nightly releases

For packages available in Debian testing and tracked in salsa (e.g., `jq` version `1.8.1`):

```bash
pkg=jq
version_orig=1.8.1
version="$version_orig-3"
# look up in salsa what the correct branch is
git_src -b "debian/$version" https://salsa.debian.org/debian/jq.git

version_suffix=gl0+bp1877
```

### Backporting missing Salsa sources (last resort)

:::warning
Use this method only as a last resort when source code is not available in Debian salsa or when upstream sources differ significantly. In all other cases, this method is discouraged.
:::

For packages present in Garden Linux nightly snapshots but not in salsa (e.g., `sqlite3` version `3.46.1`):

```bash
pkg=sqlite3
version_orig=3.46.1
version="${version_orig}-7"
version_suffix="gl0+bp1877"

# Option 1: Use a known snapshot timestamp
# snapshot=1754316590

# Option 2: Search through Garden Linux nightly versions
snapshot_gl_version="1943" # Start searching from this GL nightly version and search forward 30 versions
snapshot=$(pkg_version_in_gl_version "$pkg" "$version" "$snapshot_gl_version" 30)

# Get sources from snapshot
snapshot_src "$pkg" "$snapshot_timestamp"
```

:::info
Debian snapshots can be useful for determining when Garden Linux snapshots contain a certain package version. For example, visit https://snapshot.debian.org/package/sqlite3/3.46.1-7/ to find the date `2025-07-26`, then use `bin/garden-version --major 2025-07-26` to get `1943` as a candidate version for `snapshot_gl_version`.
:::

## Related topics

<RelatedTopics />
