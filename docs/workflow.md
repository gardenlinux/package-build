# Gardenlinux build-package workflows

This repository contains the Gardenlinux workflows for building and deploying containers and provide a resuable workflow for other packages to build and depolying.

- [Workflow](#workflows)
- [Workflow Detail](#workflow-details)
  - [Build Container](#build-container)
    - [Workflow Configuration](#workflow-configuration)
    - [Job Details](#job-details)
  - [Build Package](#build-package)
    - [Workflow Configuration](#workflow-configuration-1)
    - [Job Details](#job-details-1)
    - [Usage](#usage)
- [Contributing](#contributing)

## Workflows

The following workflows are defined in this repository:

- `build_container.yml`: This workflow builds container images based on the provided .containererfile and pushes it to a container registry.
- `build_pkg.yml`: This is reusable workflow builds a package based on the provided input configurations and pushes it to a package repository.

## Workflow Details

### Build Container

The `build_container.yml` workflow is responsible for building container images and pushing it to a container registry.

#### Workflow Configuration

The `container/` folder in this project contains all its configurations. The `build_container.yml` workflow is triggered on every push to the repository. 

#### Job Details

The `build` job within the `build_container.yml` workflow performs the following steps:
- Checks out the repository.
- Sets up the binfmt using a privileged Podman container.
- Builds the container images based on the provided containerfiles, considering the `host` and `target` matrix values that builds amd64:amd64, arm64v8:amd64, amd64:arm64v8 and arm64v8:arm64v8 images.
- Publishes the built container images to the container registry.

### Build Package

The `build_pkg.yml` workflow is a reusable workflow that can be called by other packages. It builds a package based on the provided input configurations and pushes it to a package repository.

#### Workflow Configuration

The `build_pkg.yml` workflow can be triggered by calling it with specific input parameters.

Input Parameters available in `build_pkg.yml`:
| Parameter | Type | Description |
| --------- | ---- | ------------|
|`ref`|**Type:** string<br>**Default:** `${{ github.sha }}`| The ref (commit or branch) that should be checked out from the `package-build` repo while processing the package build.|
|`build_container`|**Type:** string<br>**Default:** `ghcr.io/gardenlinux/package-build`| The container image used for building the package.|
|`source`|**Type:** string| The source name of the package. There are three values that one can choose from:<br>- Debian Source Package: `{SOURCE PACKAGE NAME}`<br>- Git Source: `git+{GIT URL}`<br>- Native Build: `native`|
|`debian_source`|**Type:** string| Defines from which source the `debian/` directory should be used. There are three values that one can choose from:<br>- Debian Source Package: `{SOURCE PACKAGE NAME}`<br>- Git Source: `git+{GIT URL}`<br>- Native Build: `native`|
|`email`|**Type:** string<br>**Default:** `contact@gardenlinux.io`| The Debian package maintainer email.|
|`maintainer`|**Type:** string<br>**Default:** `Garden Linux Builder`| The Debian package maintainer full name.|
|`distribution`|**Type:** string<br>**Default:** `gardenlinux`| The target distribution for the package.|
|`message`|**Type:** string<br>**Default:** `Rebuild for Garden Linux.`| The changelog entry message for the package build.|
|`build_option`|**Type:** string| Additional build options for the package build. Build option `terse` is always set.|
|`build_profiles`|**Type:** string| Additional build profiles for the package build.|
|`build_dependencies`|**Type:** string| List of build dependencies that should be included in the build. Each new line is a new dependency.<br><br>The format looks like this:<br>`{REPO}@{"latest" \| GIT TAG}` <br>`gardenlinux/package-libyang2@latest`<br>`gardenlinux/package-libyang2@gardenlinux/2.1.128-0gardenlinux0`|
|`git_filter`|**Type:** string<br>**Default:** `.*`<br>**Scope:** `source: git`| This parameter let's you filter Git tags that should be skipped for the upstream version determination. |
|`git_tag_match`|**Type:** string<br>**Default:** `(.*)`<br>**Scope:** `source: git`| This parameter defines what part of a given Git Tag is considered to be the upstream version. The first regex match group is always the designated upstream version. |


#### Job Details

The `build_pkg.yml` workflow consists of the following jobs:
- `source`: This job retrieves the source package, sets up the build environment, and builds the source package.
- `packages`: This job builds the binary packages for the specified architecture.
- `publish`: This job publishes the drafted release.
- `cleanup`: This job deletes the drafted release in case of failure.

#### Usage

To use the `build_pkg.yml` workflow defined in this repository, follow these steps:

1. Create a new workflow file (e.g., `build.yaml`) in the `.github/workflows/` folder of your package-project.
2. In the new workflow file, include the following content, eg: htop:

```
   name: project
   on: push
   jobs:
     htop:
       uses: gardenlinux/package-build/.github/workflows/build_pkg.yml@branchname
```
Replace `project` to other package name of your package-project. Replace `branchname` with the specific branch or tag of the build_pkg.yaml workflow you want to use. The default is `main` branch.

3. Call the gardenlinux/package-build/.github/workflows/build_pkg.yml@branchname workflow with the specific input parameters under `with:` specific to your project. such as `email`, `maintainer`, `distribution`, `message` and others mentioned in the Workflow Configuration section. The default values will be use if wihout provide any input parameters.
```
       uses: gardenlinux/package-build/.github/workflows/build_pkg.yml@branchname
       with:
         email: your@address
         maintainer: "Your Name"
```
4. Commit and push the new workflow file to your project's repository.
5. The `build_pkg.yml` workflow will be triggered based on the provided inputs.

## Contributing

Contributions to this repository are welcome. If you encounter any issues or have suggestions for improvements, please open an issue or submit a pull request.
