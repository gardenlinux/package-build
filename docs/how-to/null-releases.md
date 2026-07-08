---
title: NULL Releases
description: Learn how to use NULL releases to exclude Packages from Garden Linux builds
order: 7
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
github_source_path: docs/how-to/null-releases.md
github_target_path: docs/how-to/packaging/null-releases.md
---

# NULL Releases

This guide explains how to use NULL releases to exclude packages from Garden Linux builds, following the process described in [MAINTAINER.md](https://github.com/gardenlinux/package-build/blob/main/MAINTAINER.md#disable-a-package-xyz-repository-null-release-handling).

## What is a NULL Release?

A NULL release is a special GitHub release that tells the Garden Linux repository infrastructure to ignore a package for inclusion in apt repositories. When the `fetch_releases` script encounters a NULL release marked as the latest, it skips that package during the repository build process.

NULL releases are used to:

- Temporarily disable a package that is causing build failures
- Remove a package from the distribution
- Exclude packages with licensing or security issues
- Mark deprecated packages

## How NULL Releases Work

The null release mechanism operates through several components in the Garden Linux infrastructure:

1. **`fetch_releases` script**: Part of the gardenlinux/repo GitHub Actions, this script iterates through all gardenlinux/package-\* repositories and gets the latest release tag
2. **`download_pkgs` script**: Downloads the latest release files from all package-\* GitHub Releases
3. **Null file detection**: If the latest release of a package-\* repository contains a `null` file, it is ignored and not included in the apt repository

This creates a simple but effective way to exclude packages without modifying configuration files or deleting repositories.

## Creating a NULL Release

To create a NULL release that excludes a package from Garden Linux builds:

```bash
git checkout --detach
git commit --allow-empty -m "ignore package for repo import"
git tag null
git push origin null
echo "" > null
gh release create null ./null --title "Null release" --notes "null"
```

**Step-by-step explanation**:

1. `git checkout --detach`: Create a detached HEAD to avoid affecting any branches
2. `git commit --allow-empty -m "ignore package for repo import"`: Create an empty commit with descriptive message
3. `git tag null`: Create a lightweight tag named "null"
4. `git push origin null`: Push the null tag to the remote repository
5. `echo "" > null`: Create an empty file named "null"
6. `gh release create null ./null --title "Null release" --notes "null"`: Create a GitHub release with the null file attached

## Re-enabling a Package

to re-enable a package that was previously disabled with a NULL release, you have two options:

**Option 1: Create a new regular release**

- Create a new version of the package
- Build and release it normally
- The new release will become the latest and override the NULL release

**Option 2: Set the appropriate version as latest**

- If the package already has a proper release after the NULL release, you can manually set it as the latest release on GitHub
- This effectively bypasses the NULL release

## Important Considerations

:::danger
If you want to permanently remove/archive a package from Garden Linux, follow the [official guide](https://github.com/gardenlinux/repo?tab=readme-ov-file#remove-garden-linux-packages-from-the-repo) in the gardenlinux/repo repository.
:::

:::warning
NULL releases should only be used for packages maintained by the Garden Linux team. For third-party packages or external dependencies, other exclusion mechanisms may be more appropriate.
:::

:::info
NULL releases are particularly useful for temporarily disabling packages that are failing to build, allowing the rest of the release process to continue while the issue is investigated.
:::

## Related Topics

<RelatedTopics />
