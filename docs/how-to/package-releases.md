---
title: Creating Package Releases
description: Comprehensive guide to creating Garden Linux Package releases
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
migration_status: "done"
migration_issue: "https://github.com/gardenlinux/gardenlinux/issues/4705"
migration_stakeholder: "@tmangold, @yeoldegrove, @ByteOtter"
migration_approved: false
github_org: gardenlinux
github_repo: gardenlinux
github_source_path: docs/how-to/package-releases.md
github_target_path: docs/how-to/releases/package-releases.md
---

# Creating Package Releases

This guide explains how to create Garden Linux Package releases.

## Release Hierarchy

Garden Linux uses a [three-tier release hierarchy](/explanation/release-hierarchy.md) to deliver a complete operating system.

This document is about the first tier, the [Packaging](/explanation/packaging).

## Understanding Packaging

Please read the [Packaging Explanation](/explanation/packaging.md) to get familiar with the concepts of Packaging.

## Prerequisites

Before creating an OS release, ensure you have:

- Write access to the `gardenlinux/package-*` repositories
- `git` and `gh` CLI tool installed and configured

## Phase 0: Preparation

### Step 1: Decide What to Include

In tier one, individual [packages](/explanation/packaging.md) will need updates based on e.g. recent CVEs. Check for potential upgrades in important packages, too:

- golang
- containerd
- runc
- curl
- openssl
- systemd
- glibc
- linux kernel

**For each package requiring an update**, we need to follow the [Package Backporting Guide](/how-to/packaging/backporting.md) later.

1. Navigate to the respective `github.com/gardenlinux/package-{name}` repository
2. Create a new package release if needed (see [package-build documentation](/explanation/packaging.md) for details)
3. Note the new version tag for use in the APT repository release

The complexity of updating a package varies:

- **Simple**: Packages that rebuild Debian packages with minimal adaptions
- **Complex**: Packages with extensive adaptions that may not apply to newer versions, or packages built from upstream sources

For complex updates, thoroughly test the new package version before including it in a release.

## Phase 1: Creating the Release

### Step 1: Create APT Repository Release

In tier two, the [APT repository](/explanation/repo-infrastructure.md) must be created before building OS images.

**For Major Releases:**

Major releases automatically create APT repositories via the nightly `update.yml` workflow in the `repo` repository. The workflow:

- Collects latest package versions from all `package-*` repositories
- Creates a new Debian snapshot
- Generates `package-releases` and `.container` files
- Publishes the APT repository to S3

No manual action is required for major release APT repositories.

**For Minor Releases:**

Follow the comprehensive guide: [Creating APT Repository Releases](/how-to/releases/apt-repos.md)

Quick summary:

1. Checkout the base APT repository release tag in the `repo` repository
2. Modify `package-releases` and `package-imports` files
3. Commit and create a new tag (e.g., `2150.1.0`)
4. Push the tag to trigger the build

**Verify APT Repository:**

- Wait for the `repo` workflow to complete successfully
- Verify the APT repository is published to S3

### Step 2: Build Garden Linux OS

Once the APT repository is ready, we can build the OS images in tier three:

**Checkout or Create Release Branch:**

```bash
# For major releases, create a new release branch from main
git checkout main
git pull
git checkout -b rel-MAJOR
git push origin rel-MAJOR

# For minor releases, checkout the existing release branch
git checkout rel-MAJOR
git pull
```

**Update VERSION File:**

```bash
# Edit the VERSION file to contain the new version
echo "MAJOR.MINOR.0" > VERSION

# Example for minor release:
echo "2150.1.0" > VERSION

# Commit the change
git add VERSION
git commit -m "Bump version to MAJOR.MINOR.0"
git push origin rel-MAJOR
```

**Trigger Build Workflow:**

The build process varies depending on the Garden Linux version:

**For releases 2016.0.0 and later (using [SemVer](/reference/glossary.html#semver)):**

Use the [manual release workflow](https://github.com/gardenlinux/gardenlinux/actions/workflows/manual_release.yml):

```bash
# Using gh CLI:
gh workflow run "Build and publish a release" \
--ref rel-MAJOR \
-f target=release \
-f version=MAJOR.MINOR.0

# Example:
gh workflow run "Build and publish a release" \
--ref rel-2150 \
-f target=release \
-f version=2150.1.0
```

Or via the GitHub UI:

1. Go to [Actions → Build and publish a release](https://github.com/gardenlinux/gardenlinux/actions/workflows/manual_release.yml)
2. Click "Run workflow"
3. Select the `rel-MAJOR` branch
4. Set version to `MAJOR.MINOR.0`
5. Leave target `release`
6. Leave other parameters at defaults

**For older releases (1443, 1592 using non-SemVer):**

Use the [nightly workflow](https://github.com/gardenlinux/gardenlinux/actions/workflows/nightly.yml) with version parameter:

```bash
# Using gh CLI:
gh workflow run nightly.yml \
--ref rel-MAJOR \
-f version=MAJOR.MINOR

# Example for 1443:
gh workflow run nightly.yml \
--ref rel-1443 \
-f version=1443.3
```

**Monitor the Build:**

- Watch the workflow progress in GitHub Actions
- Verify all build jobs complete successfully
- Check that artifacts are published

### Step 3: Create GitHub Release

After the build completes, create the official GitHub release page:

```bash
# Run from main branch (not the release branch!)
git checkout main

# Using gh CLI:
gh workflow run "release page" \
--ref main \
-f run_id=<BUILD-RUN-ID>
-f is_latest=true # if this is the latest major.minor.0 version

# Example:
gh workflow run "release page" \
--ref main \
-f run_id=23802291489 \
-f is_latest=true # if this is the latest major.minor.0 version
```

:::tip
Get the "Run ID" from the URL of the "Build and publish a release" workflow run, e.g. https://github.com/gardenlinux/gardenlinux/actions/runs/23802291489
:::

Or via the GitHub UI:

1. Go to [Actions → release page](https://github.com/gardenlinux/gardenlinux/actions/workflows/manual_gh_release_page.yml)
2. Click "Run workflow"
3. Select `main` branch
4. Set Build workflow run ID to `BUILD-RUN-ID` (e.g. `23802291489`)

:::warning
Always review the generated release notes before publishing, especially the "Changes" section.

The "Changes" section lists:

- Upgraded packages in the minor release
- Fixed CVEs

This data is generated by [glvd](https://github.com/gardenlinux/glvd) and may not be perfect. Verify:

- CVE fixes match your expectations
- Package upgrades are correctly listed
- No unexpected changes appear

:::

**Tag Manifest for Non-SemVer Releases:**

:::warning
For releases that do not use semantic versioning (e.g., `1877.5`), add an extra tag for compatibility with Gardener (especially for USI images).

```bash
# Example: Tag 1877.5 as 1877.5.0
oras tag ghcr.io/gardenlinux/gardenlinux:1877.5 1877.5.0
```

:::

### Step 4: [ONLY FOR MAJOR RELEASES] Create rel-MAJOR Branch in repo

:::danger
This step only applies to new major releases, not minor releases.
:::

Create a release branch in the `gardenlinux/repo` repository based on the new release tag:

```bash
cd /path/to/repo
git pull --tags
git checkout MAJOR.0.0
git checkout -b rel-MAJOR
git push -u origin rel-MAJOR
```

Example:

```bash
cd /path/to/repo
git pull --tags
git checkout 2150.0.0
git checkout -b rel-2150
git push -u origin rel-2150
```

This branch is used for creating future minor releases of the APT repository.

### Step 5: Generate CPE File

Generate the Common Platform Enumeration (CPE) file for the new release:

```bash
# Run from main branch
gh workflow run "Generate and upload CPE to a release" \
--ref main \
-f version=MAJOR.MINOR.0

# Example:
gh workflow run "Generate and upload CPE to a release" \
--ref main \
-f version=2150.1.0
```

Or via GitHub UI:

1. Go to [Actions → Generate and upload CPE](https://github.com/gardenlinux/gardenlinux/actions/workflows/cpe.yml)
2. Click "Run workflow"
3. Select `main` branch
4. Set version to `MAJOR.MINOR.0`

### Step 6: Update Garden Linux Documentation

:::info
TODO: add steps on how to update the documentation
:::

## Phase 2: Post-Release Work

### Verify Release Completeness

- Verify the GitHub release page is complete and accurate
- Check that all expected artifacts are attached to the release
- Confirm the CPE file was generated and uploaded
- Test installation using the new release

### Notify Stakeholders

- Notify relevant teams about the new release
- Gardener OS Extension Team
- Update any documentation referencing supported versions

## Related Topics

<RelatedTopics />
