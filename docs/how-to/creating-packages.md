---
title: Creating Packages
description: Learn how to create Garden Linux Packages
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
github_repo: package-build
github_source_path: docs/how-to/creating-packages.md
github_target_path: docs/how-to/packaging/creating-packages.md
---

# Creating Packages

This guide covers the complete workflow for creating Garden Linux packages, from initial setup to handover to the repository infrastructure.

## Overview

The package creation process follows these stages:

1. Prepare the package source
2. Make source package
3. Make binary package
4. Test binary package
5. Handover to repo build

This guide walks through each stage in detail, integrating the rules and best practices for Garden Linux packages.

## Step 1: Prepare the Package Source

The package source consists of three components that must be assembled:

1. **Upstream source code**
2. **The debian/ folder**
3. **Garden Linux patches**

The `prepare_source` script is responsible for assembling these components and is executed as a step in the [gardenlinux/package-build:bin/source](https://github.com/gardenlinux/package-build/blob/main/bin/source) workflow.

### Get the debian/ folder

The method for obtaining the debian/ folder depends on whether the package exists in Debian:

| Scenario                            | Method                                        |
| ----------------------------------- | --------------------------------------------- |
| Package in Debian testing           | Get from Garden Linux snapshot apt repository |
| Package not in testing but in salsa | Get from salsa.debian.org                     |
| Package not in Debian at all        | Create and maintain in package repository     |

**For packages in Debian testing**:

Use the `apt_src` helper function (according to [Rule 4](/reference/packaging-rules.html#rule-4-get-debian-folder-from-garden-linux-snapshot-apt-repository)):

```bash
apt_src --ignore_orig <source_package_name>
```

**For packages not in Debian**:

Manually create the `debian/` folder with all required files and check it into the `package-` repository.

:::tip
Use existing similar packages as templates and follow the Debian maintainer documentation.
:::

### Get the upstream source

Get the source code from the upstream git repository, not from Debian source packages. This enables automated version tracking using uscan.

Use the `git_src` helper function:

```bash
git_src --branch <BRANCH_NAME> <GIT_REPO_URL>
```

For example, to get OpenSSL 3.1.7:

```bash
version_orig=3.1.7
git_src --branch "openssl-$version_orig" https://github.com/openssl/openssl.git
```

:::info
For packages already in Debian, get the source from salsa.debian.org using the appropriate Debian branch.
:::

### Apply Garden Linux patches

Apply both debian/ folder patches and upstream source patches in the `prepare_source` script.

**For debian/ folder patches**:

Place patches in a `fixes_debian` directory and use:

```bash
apply_patches fixes_debian
```

**For upstream source patches**:

Place patches in an `upstream_patches` directory and use:

```bash
import_upstream_patches# upstream_patches
```

For detailed patching procedures, see the [Patching](/how-to/packaging/patching.md) guide.

## Step 2: Make the Source Package

A source package contains all necessary files to build the binaries and is the input for the binary build step.

The gardenlinux/package-build repository provides reusable actions to perform this step automatically. See the [build.yml workflow](https://github.com/gardenlinux/package-build/blob/621c4c8f530a93884f7b9a4dfc348a50a2d19aa5/.github/workflows/build.yml#L29) for reference.

The source package generation process:

1. Executes the `prepare_source` script to assemble all components
2. Creates a .tar.gz archive of the source
3. Generates the .dsc file with package metadata
4. Computes checksums for all files

## Step 3: Make the Binary Package

Input for this stage is the debian source package created in the previous step.

The gardenlinux/package-build repository provides reusable actions to automatically perform the binary build. This process:

1. Extracts the source package
2. Applies all patches (from debian/patches and imported upstream patches)
3. Compiles the source code
4. Packages the compiled binaries into .deb files

A `prepare_binary` script can be added to the package repository to perform additional preparations before the build, such as:

- Reconfiguring debian build profiles
- Installing additional build dependencies
- Adding logging flags for debugging

## Step 4: Test the Binary Package

Currently, tests are disabled by default using the `nocheck` debian build profile. However, manual testing is recommended for critical packages.

To enable testing, remove the `nocheck` profile in `DEB_BUILD_OPTIONS` and ensure appropriate test dependencies are available.

:::tip
For packages where security is critical, consider adding automated tests in a separate workflow.
:::

## Step 5: Handover to Repo Build

The gardenlinux/repo repository is responsible for collecting all required packages and creating the apt repositories.

### Daily releases

A daily scheduled action in gardenlinux/repo collects the latest releases and creates a new apt repository:

1. Gets the list of `gardenlinux/package-*` repositories
2. Downloads each package/version from the GitHub Releases of the respective package-\* repository

### Patch releases

Packages for patch releases are uploaded to GitHub Releases but must be manually included in the gardenlinux/repo configuration:

1. Create the package release as usual
2. Update the `package-releases` and `package-imports` files in the `repo` repository
3. Create a new tag for the APT repository release

For complete details on apt repository creation, see the [Creating APT Repository Releases](/how-to/releases/apt-repos.md) guide.

## Related topics

<RelatedTopics />
