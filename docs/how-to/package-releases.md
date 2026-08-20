---
title: Creating Package Releases
description: Complete guide to creating Garden Linux Package releases
order: 1
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
github_repo: gardenlinux
github_source_path: docs/how-to/package-releases.md
github_target_path: docs/how-to/releases/package-releases.md
---


# Creating package releases

This guide explains how to create Garden Linux Package releases.

## Release hierarchy

Garden Linux uses a [three-tier release hierarchy](/explanation/release-hierarchy.md) to deliver a complete operating system.

This document is about the first tier, the [Packaging](/explanation/packaging).

## Understanding packaging

Read the [Packaging Explanation](/explanation/packaging.md) to get familiar with the concepts of Packaging.

## Prerequisites

Before creating a package release, ensure you have:

- Write access to the `gardenlinux/package-*` repositories
- `git` and `gh` CLI tools installed and configured

## Phase 0: Preparation

### Step 1: Decide what to include

In tier one, individual [packages](/explanation/packaging.md) will need updates based on recent CVEs or upstream releases. Check for potential upgrades in important packages:

- golang
- containerd
- runc
- curl
- openssl
- systemd
- glibc
- linux kernel

For each package requiring an update, follow the [Package Backporting Guide](/how-to/packaging/backporting.md).

1. Navigate to the respective `github.com/gardenlinux/package-<name>` repository.
2. Create a new package release if needed — see [Package Backporting Guide](/how-to/packaging/backporting.md) for details.
3. Note the new version tag for use in the [APT Repository Release](/how-to/releases/apt-repos.md).

The complexity of updating a package varies:

- **Simple**: Packages that rebuild Debian packages with minimal adaptations.
- **Complex**: Packages with extensive adaptations that may not apply to newer versions, or packages built from upstream sources. These often require [patching](/how-to/packaging/patching.md) and may have special [build dependencies](/how-to/packaging/build-dependencies.md).

For complex updates, thoroughly test the new package version before including it in a release.

:::tip
To permanently exclude a package from the repo - for example if it unwanted for some reason - use a [NULL release](/how-to/packaging/null-releases.md) instead of removing the package repository (`github.com/gardenlinux/package-<name>`).
:::

## Phase 1: Creating the package release

### Step 1: Build the package

Once you have made your changes in the `package-<name>` repository, push the branch to GitHub and verify that the build workflow runs successfully.

The build is triggered automatically on push if the workflow is configured accordingly. You can also trigger it manually:

```bash
# Trigger the build workflow manually for a specific package repository on the main (nightly) branch
gh workflow -R gardenlinux/package-<name> run build.yml --ref main

# Trigger the build workflow manually for a specific package repository on a release branch
gh workflow -R gardenlinux/package-<name> run build.yml --ref rel-2150
```

Monitor the workflow:

```bash
# Watch the workflow status
gh run watch -R gardenlinux/package-<name>
```

Or check the GitHub Actions page for the package repository.

For full details on how the GitHub Actions build workflow works, see [GitHub Actions Package Building](/how-to/packaging/github-actions.md). To build and verify the package locally before pushing, see [Local Package Building](/how-to/packaging/local-build.md).

### Step 2: Verify the release was created

After a successful build with `release: true`, the workflow automatically creates a GitHub release with the appropriate version tag (for example, `3.5.5-1gl0+bp2150`).

Confirm the release exists:

```bash
# List the latest releases for the package repository
gh release list -R gardenlinux/package-<name> --limit 5
```

Check that:

- The expected version tag is present.
- The release contains `.deb` build artifacts.
- No build errors are shown in the workflow summary.

### Step 3: Note the version tag

Record the exact version tag from the GitHub release — for example, `3.5.5-1gl0+bp2150`. You will need this when updating the `package-releases` file in the next tier.

:::info
The version tag format is `<upstream-version>-<debian-revision>gl<increment>+bp<garden-linux-major>`. See the [Packaging Explanation](/explanation/packaging.md) for details.
:::

## Verification

After the package release is published, verify it is ready for inclusion in an APT repository release.

### Check the GitHub release

```bash
# Show the release details
gh release view <VERSION-TAG> -R gardenlinux/package-<name>
```

Confirm that:

- The release is marked as the latest (or the intended tag for a patch release).
- All expected `.deb` artifacts are attached.
- The release notes accurately reflect the changes.

### Test the package locally

Use the [Local Package Building](/how-to/packaging/local-build.md) guide to build from the release tag and install the resulting `.deb` in a test environment:

```bash
# Install the package in a test Garden Linux system to verify it works
dpkg -i <package-name>_<VERSION>_amd64.deb
```

## Next step

Once all required package releases are ready, proceed to [Creating APT Repository Releases](/how-to/releases/apt-repos.md) to include the updated packages in an APT repository release.

## Related topics

<RelatedTopics />
