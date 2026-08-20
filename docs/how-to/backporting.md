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

## Branch and tag conventions

Each `package-*` repository uses a [multi-branch model and version suffixes](/explanation/packaging#Version-suffix-and-branch-model) to separate nightly development from stable release maintenance.

### Tags

Tags on `package-<name>` reposotories correspond with the [Package Versions](/explanation/packaging#understanding-package-versions)

### Setting the version suffix

The `version_suffix` variable in `prepare_source` controls the tag:

- On `main`: no `version_suffix` line. The CI auto-appends `gl0` when `release: true` is set in the workflow call.
- On `rel-<N>`: set `version_suffix` explicitly in `prepare_source` (typically at the end of the file), for example:

  ```bash
  # in prepare_source on branch rel-2150
  version_suffix=gl0+bp2150
  ```

:::info
Inside `prepare_source`, you may also write `version_suffix=gl0~bp2150` (with a tilde). The tilde ensures the backport version sorts below the corresponding nightly version in APT. The build system translates `~` to `+` when creating the git tag.
:::

## Creating patch releases

A patch release delivers an updated package version to an already-shipped Garden Linux release. You work on the `rel-<N>` branch that corresponds to the target release.

The example below uses `package-curl` targeting Garden Linux 2150.x. The starting tag is `8.21.0-2gl0+bp2150` and the goal is to produce patch tag `8.21.0-2gl1+bp2150`.

### 1. Check out the release tag

```bash
git clone https://github.com/gardenlinux/package-curl
cd package-curl
git fetch --tags
git checkout 8.21.0-2gl0+bp2150
git switch -c patch/8.21.0-2gl1+bp2150
```

This creates a local working branch from the release tag so you can make and review changes before pushing to the permanent `rel-2150` branch.

### 2. Apply modifications

Make the necessary changes or backport patches from `main`. For detailed patching guidance, see the [Patching](/how-to/packaging/patching.md) guide.

### 3. Increment the version suffix

In `prepare_source`, increment the `gl` counter in `version_suffix`:

```bash
# before
version_suffix=gl0+bp2150

# after
version_suffix=gl1+bp2150
```

When targeting an older Garden Linux release, you may also need to update the `.container` file to use the matching build container image.

:::tip
Use the [`bin/find-build-container-for`](https://github.com/gardenlinux/gardenlinux/blob/016c3889e20cb4bd937da4338f88c380cd9a49be/bin/find-build-container-for) script to find the correct container image.

```bash
bin/find-build-container-for 2150.0.0
ghcr.io/gardenlinux/repo-debian-snapshot@sha256:99f72494ab45d33958a0385054de4742c7022ab07b4c0c2cef6f785f8ebfb378
```

:::

### Optional: Build locally

To verify your package builds before pushing, see the [Local Build How-To](/how-to/packaging/local-build.md).

### 4. Push to the release branch

```bash
git push origin HEAD:rel-2150
```

The workflow on `rel-2150` triggers on push. Because `version_suffix` is already set in `prepare_source`, the build system reads it, creates the tag `8.21.0-2gl1+bp2150`, and publishes the GitHub release automatically.

:::info
On `rel-<N>` branches, `release: true` does **not** need to be set in the workflow call. The `version_suffix` in `prepare_source` is sufficient for tag creation and release publishing. The `release: true` input is only needed on `main`, where no suffix is pre-set, to trigger auto-appending `gl0`.
:::

## Backporting examples

The examples below show how to configure `prepare_source` on a `rel-<N>` branch for different source scenarios. In each case, work on the branch that corresponds to the target Garden Linux release (for example, `rel-1877` for GL 1877.x or `rel-2150` for GL 2150.x).

### Backporting new upstream version not available (yet) on Salsa

When backporting a version not available (yet) in Debian salsa (for example, OpenSSL 3.1.7 targeted at GL 1443.x), configure `prepare_source` on branch `rel-1443` as follows:

```bash
version_orig=3.1.7
version="$version_orig-0"
git_src --branch "openssl-$version_orig" https://github.com/openssl/openssl.git
apt_src --ignore-orig openssl
version_suffix=gl0~bp1443
```

This configuration fetches source files from the upstream repository while using the Debian folder from the apt source package. If there are compatibility issues, patches may need to be added using the `apply_patches` function.

### Backporting package already in nightly releases

For packages available in Debian testing and tracked in salsa (for example, `jq` version `1.8.1` targeted at GL 1877.x), configure `prepare_source` on branch `rel-1877`:

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

For packages present in Garden Linux nightly snapshots but not in salsa (for example, `sqlite3` version `3.46.1` targeted at GL 1877.x), configure `prepare_source` on branch `rel-1877`:

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
