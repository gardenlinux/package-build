# Gardenlinux APT repo - package build tools

## Local package build

1. clone this repository
   ```
   git clone https://github.com/gardenlinux/package-build.git
   ```
2. clone the package repository, e.g. *iproute2*
   ```
   git clone https://github.com/gardenlinux/package-iproute2.git
   ```
3. run `build` from the *package-build* repo with the package repository as argument
   ```
   ./package-build/build package-iproute2
   ```
   - *optional*: If a package build depends on other custom build packages you can provide the `--build-dependencies` flag with a directory containing the `.deb` files of build-time dependencies

The build artifacts (`deb` files and others) are placed in a `.build` directory in your package, for example in `./package-iproute2/.build/`.

### Build options

The `build` script takes arguments which may be used to customize the build.

- `--arch amd64`: Specify the architecture to build
   - Choices: `amd64` (default), `arm64` 
- `--source-only`: Only build source archive
- `--binary-only`: Only build the binary archives
- `--leave-artifacts`: creates the sources folder and keeps them in a `package-XYZ/output/run_<date_time>` folder of  local `package-XYZ` folder
- `--edit`: spawns a gardenlinux/repo-debian-snapshort container with `package-XYZ/output` mounted, quilt installed and configured.  

## GitHub action build

To build using GitHub actions simply define a job with

```
uses: gardenlinux/package-build/.github/workflows/build.yml@main
```

### Job inputs

The GitHub action jobs accepts various inputs:

#### `release` (*boolean*)
Flag if this is a release build.
If set to `true` this will cause the build job to automatically append `gardenlinux0` as the version suffix and create a GitHub release from the resulting source and binary packages.

#### `build_dep` (*string*)
A list of other GitHub repositories to pull custom build-time dependencies from. In the format `<repo> <tag>`

> [!Important]
> Build-time dependencies between packages are not updated automatically and need to be adjusted manually when needed

#### `runs-on`, `runs-on-amd64`, `runs-on-arm64` (*string*)
Specify the GitHub action runner on which to run the job

### Example

A full github workflow file to build a package and regularly try if a new version should be build looks as follows:

```
on:
  push:
  workflow_dispatch:
  schedule:
    - cron: '0 0 * * *'
jobs:
  build:
    uses: gardenlinux/package-build/.github/workflows/build.yml@main
    with:
      release: ${{ github.ref == 'refs/heads/main' }}
```

### Patch releases / backporting

When running the GitHub action job with `release: true` it automatically creates a new release with version suffix `gardenlinux0`.
To create a patch release for this version:

1. Check out the release tag and branch off from it
   ```
   git fetch --tags
   git checkout <VERSION>gardenlinux0
   git branch <VERSION>
   git checkout <VERSION>
   ```
2. Apply modifications or backport patches from main
3. Increment the version suffix. The GitHub action job which created the `<VERSION>gardenlinux0` tag automatically added a `version_suffix=gardenlinux0` line to the `prepare_source` script. Simply increment this suffix. In certain scenarios you might want to backport this to an older GardenLinux version; in this case you will need to also update the _.container_ file and use the proper one; e.g. _ghcr.io/gardenlinux/repo-debian-snapshot@sha256:3577da968aa41816bf255e189925c6c67df1266effe800a7d0cd66cddc5760ca_ for version 1443.
4. Push the branch
   ```
   git push origin <VERSION>
   ```
   If the action is setup to run on `push`, as in the example above, this will trigger the build job to run which detects that this is a new version and thus causes it to be released, setting up the necessary tags and GitHub releases.

#### Example: backport new upstream version not (yet) available on salsa

If we want to, for example, backport openssl 3.1.7 but this version does not exist in debian salsa step 3 of the above instructions would be to set the `prepare_source` script to

```
version_orig=3.1.7
version="$version_orig-0"
git_src --branch "openssl-$version_orig" https://github.com/openssl/openssl.git
apt_src --ignore-orig openssl
version_suffix=gl0~bp1443
```

This will cause the package build to fetch all actual source file from the upstream openssl repo on github.com, while taking the debian folder from the apt source package.
In the case that there are compatibility issues between the apt source debian folder and the new upstream source some patches might need to be added (see the apply_patches function in the source script).

#### Example: Backporting a Package Already in nightly Releases

Suppose you want to backport a package (for example, `jq` version `1.8.1`) that was previously available in Debian testing and is therefore tracked in debian's salsa repository. In this case, you can use the following approach in your `prepare_source` script:

```
pkg=jq
version_orig=1.8.1
version="$version_orig-3"
# look up in salsa what the correct branch is
git_src -b "debian/$version" https://salsa.debian.org/debian/jq.git

version_suffix=gl0+bp1877
```

This approach tells the package build system to simply checkout the debian salsa repo in a specific branch. It then rebuilds the package exactly as it was in that nightly release.

#### Example: Backporting a Package Already in nightly Releases - last resort - missing salsa sources

> [!Warning]
> Use this method only as a last resort if the source code (or parts of it) is not available in debian salsa or upstream sources differ a lot.
> In all other cases, this method is discouraged.

Suppose you want to backport a package (for example, `sqlite3` version `3.46.1`) that was previously available in Debian testing, but is not—or only partially—available in Debian's salsa repository. However, the package is still present in our daily repository snapshots, which are also used for our nightly releases. In this situation, you can use the following approach in your `prepare_source` script:

```
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

> [!Note]
> The debian snapshots can be a useful a hint since when GardenLinux snapshots contain a certain package version.
> e.g. goto https://snapshot.debian.org/package/sqlite3/3.46.1-7/ look for the date `2025-07-26` and but that into
> `bin/garden-version --major 2025-07-26`, you will get `1943` as a candidate version for `snapshot_gl_version`.

This approach tells the package build system to retrieve the `.orig.tar` and `.debian.tar` source archives for the specified version from the earliest available GardenLinux snapshot (starting from `snapshot_gl_version` and searching forward). It then rebuilds the package exactly as it was in that nightly release.
