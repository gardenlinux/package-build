---
title: Patching
description: Learn how to create and update Patches for Garden Linux Packages
order: 2
related_topics:
  - /explanation/packaging.md
  - /explanation/package-sources.md
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
github_source_path: docs/how-to/patching.md
github_target_path: docs/how-to/packaging/patching.md
---

# Patching

This guide covers how to create and manage patches for Garden Linux packages, following the best practices from [PATCHING.MD](https://github.com/gardenlinux/package-build/blob/main/PATCHING.MD).

## When to create patches

Patches are needed when:

- Fixing bugs in upstream source code
- Applying security fixes from upstream to older versions
- Customizing package behavior for Garden Linux
- Updating the debian/ folder configuration
- Backporting features or fixes to a specific version

## Prerequisites

To create patches, you need:

- The `package-build` tools installed locally
- Git
- Access to the package repository
- Basic understanding of quilt

## Step1: Prepare your local sources

First, you need the complete package sources with all patches applied exactly as the GitHub Actions pipeline would do.

```bash
./package-build/build --leave-artifacts --source-only package-<package_name>
```

This command:

- Calls the `package-build/bin/source` script in a container
- Places the source in `package-<package_name>/output/run-<datetime>/a`
- Creates a copy in `package-<package_name>/output/run-<datetime>/b`
- Keeps the sources even if patch application fails

:::warning
If building for arm64, include `--arch arm64` in the build command. Cross-build for generating sources may cause issues.
:::

:::info
If the source build fails to apply a patch, the sources are kept in the state where the process stopped, allowing you to fix the issue.
:::

## Step2: Spawn a patch environment

Use the edit mode to spawn a container with quilt already configured correctly:

```bash
./package-build/build --edit package-<package_name>
```

This starts a Debian container with the output folder from the previous step mounted. The container has quilt pre-configured according to Debian standards, which is important for maintaining compatibility with patches pulled from salsa.debian.org.

The preparation is defined in [package-build/bin/patchenv-init](https://github.com/gardenlinux/package-build/blob/main/bin/patchenv-init), so you can review those settings and use your local machine if preferred.

## Step3: Make your changes

When working on patches, keep the `a` directory unchanged and make all your changes in the `b` directory. This allows you to create a proper diff between the original and modified sources.

### Using Quilt

Quilt is the recommended tool for managing patches. Key commands:

| Command               | Description                           |
| --------------------- | ------------------------------------- |
| `quilt push`          | Apply a single patch                  |
| `quilt push -a`       | Apply all patches                     |
| `quilt push -f`       | Force apply patches                   |
| `quilt refresh`       | Update patch files after manual edits |
| `quilt add <file>`    | Add a new file to be patched          |
| `quilt remove <file>` | Remove a file from the patch          |

:::tip
Use `quilt series` to see all patches in order and `quilt applied` to see which are already applied.
:::

### External Quilt references

- [Debian Wiki: Using Quilt](https://wiki.debian.org/UsingQuilt) - Official guide
- [Quilt Tutorial](http://www.shakthimaan.com/downloads/glv/quilt-tutorial/quilt-doc.pdf) - Comprehensive tutorial (GNU FDL licensed)
- [Refresh a patch that failed to apply](https://wiki.debian.org/UsingQuilt#Refresh_a_patch_that_failed_to_apply) - Debian guide for fixing broken patches

## Step4: Create the patch

The diff between the original `a` directory and your modified `b` directory becomes your patch.

```bash
diff -Naur a/ b/ > <name-of-your-patch>.patch
```

:::tip
If you know which file was modified, create a focused patch:

```bash
diff -Naur a/path/to/file b/path/to/file > fixes_debian/my-fix.patch
```

:::

## Step5: Add the patch to the package

Depending on the type of patch, place it in the appropriate directory and update the `prepare_source` script.

### For debian/ folder patches

Place patches in a `fixes_debian` directory and apply them in `prepare_source`:

```bash
apply_patches fixes_debian
```

### For upstream source patches

Place patches in an `upstream_patches` directory and import them in `prepare_source`:

```bash
import_upstream_patches# upstream_patches
```

### Creating the directory

If the directory doesn't exist, create it and add the patch:

```bash
mkdir -p fixes_debian
mv <name-of-your-patch>.patch fixes_debian/
```

Then add the patch to the series file:

```bash
echo "<name-of-your-patch>" >> fixes_debian/series
```

:::info
The `series` file lists all patches in the order they should be applied.
:::

## Patch organization

Follow these conventions for patch organization:

| Patch Type                  | Directory             | Application Method         |
| --------------------------- | --------------------- | -------------------------- |
| debian/ folder changes      | `fixes_debian`        | `apply_patches`            |
| upstream source changes     | `upstream_patches`    | `import_upstream_patches#` |
| build dependency patches    | `fixes_build_dep`     | `apply_patches`            |
| local customization patches | `gardenlinux_patches` | `import_upstream_patches#` |

## Related topics

<RelatedTopics />
