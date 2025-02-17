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
- `--debug`: keeps sources of source step inside an output folder. Used to prepare sources for manual patching.  
### Create/Fix Patches

In the following we will go through one way of creating a patch for a gardenlinux package. 
We will use package-linux as an example.

#### 1. Prepare your local sources

First, we need the sources of the package-linux with all patches applied exactly as the GitHub package-build pipeline would do. 
Later on, when we have the sources, we can fix, remove, add patches to the sources easily. 


```
./package-build/build --debug --source-only package-linux
```
This step has called the [package-build/bin/source](https://github.com/gardenlinux/package-build/blob/main/bin/source) script inside a container, and placed the sources inside the flocal folder `package-linux/output/run-<date-time>/a`, and another copy `package-linux/output/run-<date-time>/b` right next to it. Even if the source script fails to apply a certain patch, you will have the source folders exactly in the state where the source build left bailed out. 


> [!Warning]
> If you run this on arm64, then you need to also pass `--arch arm64` for the source build. Cross-build for generating sources is not required and might cause issues.

> [!Note]
> If the package-build/bin/source has failed, the sources are kept and are in the state where the package-build/bin/sources exited. 

#### 2. Spawn a temporary linux container to use quilt 

Quilt is a versatile tool for working with patches. You can use it to fix, add or remove patches. 
Patches inherited from debian (the patches inside the debian/patches folder that we pull) are in allmost all cases maintained with quilt.

If we want to fix a patch from debian, then we need to make sure to use the same patch format as debian. Quilt can be [configured accordingly](https://wiki.debian.org/UsingQuilt#Using_quilt_with_Debian_source_packages). 

The following command spawns a container with quilt already configured correctly.


```
./package-build/build --edit package-linux
```

This starts a debian container with the output folder generated from the previous step mounted. 

Preparations are defined in [package-build/bin/patchenv-init](https://github.com/gardenlinux/package-build/blob/main/bin/patchenv-init),
you can review those and use your local machine instead. 

#### 3. Make your changes inside folder b 
Debian guide references [this](http://www.shakthimaan.com/downloads/glv/quilt-tutorial/quilt-doc.pdf) tutorial, licensed under the GNU Free Documentation License, and may be a good starting point for you. 


#### 4. Create the patch 

The diff between folder a and your edited folder b is now the patch. We just need to create it in an acceptable format:

```
diff -Naur a/ b/ > name-of-your-patch.patch 
```

> [!Tip]
> If you already know what file has changed, e.g. because you fix a debian/patch/file, then you can instead use
> ```
> diff -Naur a/path/to/file b/path/to/file > name-of-your-patch.patch 
> ```

#### 5. Append the patch to the patches folder

Depending on the package you are working with, there may or may not already exists an appropriate patches folder where you can add the `name-of-your-patch.patch` to.

If it does not exist yet, you need to create a folder (e.g. called `fixes_debian`) and then adapt the prepare_source script to apply patches by adding for example this line
```
apply_patches fixes_debian
```

apply_patches is a bash function sourced from [package-build/bin/source](https://github.com/gardenlinux/package-build/blob/main/bin/source) and applies all patches as defined by the series file. 
See example [package-linux](https://github.com/gardenlinux/package-linux/blob/18baefb947b6fb3a4abaa9c58b6a42be3117e6dd/prepare_source#L30C1-L30C27)


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
3. Increment the version suffix. The GitHub action job which created the `<VERSION>gardenlinux0` tag automatically added a `version_suffix=gardenlinux0` line to the `prepare_source` script. Simply increment this suffix. In certain scenarios you might want to backport this to an older GardenLinux version; in this case you will need to also update the _.containerfile_ and use the proper one; e.g. _ghcr.io/gardenlinux/repo-debian-snapshot@sha256:3577da968aa41816bf255e189925c6c67df1266effe800a7d0cd66cddc5760ca_ for version 1443.
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

##

