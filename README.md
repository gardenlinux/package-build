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

## Package build API

Package builds make use of shell functions which are defined in this repo.

### `prepare_source`

`apt_src [--ignore-orig] $package_name`

Use source code from a given apt source package.

`git_src $git_url`

Use source code from a remote git repository

`import_src $path`

Use source code from a local path (todo: how does that work?)

`apply_patched [patch_dir(default=./patches)]`

`import_upstream_patches [patch_dir(default=./upstream_patches)]`

### `prepare_binary`

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

### Patch releases

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
3. Increment the version suffix. The GitHub action job which created the `<VERSION>gardenlinux0` tag automatically added a `version_suffix=gardenlinux0` line to the `prepare_source` script. Simply increment this suffix.
4. Push the branch
   ```
   git push origin <VERSION>
   ```
   If the action is setup to run on `push`, as in the example above, this will trigger the build job to run which detects that this is a new version and thus causes it to be released, setting up the necessary tags and GitHub releases.
