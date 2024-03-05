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

## GitHub action build

To build using GitHub actions simply define a job with

```
uses: gardenlinux/package-build/.github/workflows/build.yml@main
```

### Job inputs

The GitHub action jobs accepts various inputs:

- `release` (*boolean*):  
   Flag if this is a release build.
   If set to `true` this will cause the build job to automatically append `gardenlinux0` as the version suffix and create a GitHub release from the resulting source and binary packages.
- `build_dep` (*string*):  
   A list of other GitHub repositories to pull custom build-time dependencies from. In the format `<repo> <tag>`
   > [!Important]
   > Build-time dependencies between packages are not updated automatically and need to be adjusted manually when needed
- `runs-on`, `runs-on-amd64`, `runs-on-arm64` (*string*):  
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
