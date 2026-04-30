---
title: Removing Packages from Repository
description: Procedure to switch back to Debian-provided packages instead of Garden Linux-built packages
order: 8
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
github_source_path: docs/how-to/remove-packages.md
github_target_path: docs/how-to/packaging/remove-packages.md
---

# Removing Packages from Repository

There may be cases where we need to switch back to using Debian-provided packages instead of Garden Linux-built packages.

## Important Considerations

:::danger
DO NOT RENAME PACKAGE REPOS

Renaming package repositories will break future patch releases, as the `package-releases` file references repositories by their exact names. Additionally, you should disable GitHub Actions for the repository to prevent version updates from overwriting the null release.
:::

## Procedure to Nullify a Package

To remove a Garden Linux package and revert to the Debian version:

1. **Create an archive branch**

   ```bash
   git checkout -b archive-<PACKAGE_NAME>
   ```

2. **Disable GitHub Actions and remove files**

   ```bash
   git rm -r .github/workflows
   git rm -r <ALL_FILES>
   ```

3. **Create a README documenting the package**

   ```bash
   git add README.md
   git commit -a -m "null"
   git push -u origin archive-<PACKAGE_NAME>
   ```

4. **Create the null tag and release**

   ```bash
   git tag null
   git push origin null

   # Create a non-empty file for the release (required by GitHub)
   echo null > ~/null
   gh release create null ~/null --latest --notes null --title 'ignore package for repo import'
   ```

5. **Archive the repository in GitHub**
   - Go to: `github.com/gardenlinux/package-<NAME>_repo`
   - Navigate to Settings → Danger Zone → Archive

This process ensures that:

- The package repository is preserved for historical reference
- No new builds will be triggered for the package
- The `repo` will import the package from Debian instead of using a Garden Linux build
- Future patch releases for the Garden Linux version will continue to work correctly
