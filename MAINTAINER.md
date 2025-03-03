# Maintainer Guide 

This guide is written for Garden Linux maintainers, explaining with context the steps required for creating a package for Garden Linux. 



## Overview
| Rule Number | Description                                             |
|-------------|-------------------------------------------------------|
| [Rule 1](#rule-1-package-git-repositories-must-be-named-accordingly) | Package git repositories must be named accordingly     |
| [Rule 2](#rule-2-git-branches-of-package-repositories-must-be-named-accordingly) | Git branches of package repositories must be named accordingly |
| [Rule 3](#rule-3-one-shot-backport-repositories-must-start-with-bp-package) | One-shot backport repositories must start with bp-package |
| [Rule 4](#rule-4-get-source-from-salsa) | Get source from salsa                                    |
| [Rule 5](#rule-5-create-debian-folder-in-package-repo-only-if-debian-package-does-not-exist) | Create `debian/` folder in package repo only if Debian package does not exist |
| [Rule 6](#rule-6-get-upstream-source-from-upstream-git) | Get upstream source from upstream git                   |
| [Rule 7](#rule-7-patching-the-patches) | Patching the patches |
| [Rule 8](#rule-8-append-to-debian-patches) | Append to debian patches        |


# Git Repository conventions

## Repository Naming 


### Rule 1: Package git repositories must be named accordingly
```
package-<package-source>
```
- must start with package- 
- <package-source> must be the name of the source package as it is defined in debian
   - exception: if package does not exist in debian 


### Rule 2: Git branches of package repositories must be named accordingly

Convention: 
- `main`: builds against latest Garden Linux environment
- `rel-<MAJOR>`: builds against `<MAJOR>` version of Garden Linux. 
- `fix/*`, `feat/*`, `other/*`: are allowed to indicate that the branch is not used for   


### Rule 3: One-shot backport repositories must start with bp-package

Package repositories only required as a dependency for a backported package must start with bp-package-*.

Please note that the bp-package-* is only included in the targeted package-*  backport.

# Package build process 

Overview of steps to make a package for Garden Linux:
```
1. prepare the package source
2. make source package
3. make binary package(s)
4. test binary package(s)
5. handover to repo build 
```

We will walk through each step in detail below.


## Prepare the package Source

The source to create a package consists of three parts. 
The **upstream source code** + the **debian/ folder** + **Garden Linux patches**

### Get debian/ folder

The debian folder contains patches, configurations and rules to make and install the software. For more details about the required content of that debian folder, please read [debian documentation](https://www.debian.org/doc/manuals/maint-guide/dreq.en.html).

To create a Garden Linux package, we also need a **debian/ folder** including all the required files. In the following we define how we **SHOULD** get the debian folder, depending on the case:


#### Rule 4: Get source from salsa
If there exists a debian package, we must get the **debian/ folder** from salsa git repository, and use git tags to retrieve the corresponding version. 
The **debian/ folder** used by debian packages are in version control and publicly accessible via the [salsa GitLab instance](https://salsa.debian.org/public) 


## Rule 5: Create `debian/` folder in package repo only if Debian package does not exist

If there does **NOT** exist a debian package, we must define the **debian/ folder** ourself and check it in our `package-` repository. 


### Get upstream source  

We **MUST** watch upstream git repository automatically, and automatically trigger pipelines to build and test new upstream versions without waiting for a debian maintainer to upgrade salsa. 
For that, we use a scan tooling based on debian's uscan. Rules for this tool are defined per package in `debian/watch`, and include what target upstream repo to watch, and what to watch including what semversion changes should be pulled (e.g. only patchlevel).
:> [!WARNING]
> Guide on how to define these `debian/watch` rules is to be done!

## Rule 6: Get upstream source from upstream git
To enable automatic upstream version tracking, get source from upstream git repository. 
This means do NOT use apt source packages, do NOT use patches to update to a version.

### Garden Linux Patches 

Garden Linux Patches are applied on top of debian patches. 

:> [!WARNING]
> please see patching guide --- insert link here --- 

#### Rule 7: Patching the patches 
Patching the patches. We consider the debian/patches folder as source, and changes wen make to debian/patches are done and tracked via patches. 


#### Rule 8: Append to `debian/patches`
Upstream patches are appended to debian/patches/series and patched as new files into the debian/patches folder. 


# Make source package 
A source package contains all the necessary files to build the binaries, and will be used as input by the next step [Make binary package](##Make-binary-package). 

A definition of a debian source package can be found [here](https://wiki.debian.org/Packaging/SourcePackage).


The central gardenlinux/package-build repo contains reusable actions, that are used to automatically perform this step. For reference, see that reusable action [here](https://github.com/gardenlinux/package-build/blob/621c4c8f530a93884f7b9a4dfc348a50a2d19aa5/.github/workflows/build.yml#L29)


# Make binary package 

Input for this stage is a debian source package created by the previous stage.
The central gardenlinux/package-build repo contains reusable actions, that are used to automatically perform this step, as well. 

# Test binary package 

TBD

# Handover to repo build 

## Daily release 
Packages are uploaded to GitHub Release Page. A daily scheduled gardenlinux/repo action collects all latest releases and creates a new apt repository based on latest packages. 

## Patch release 
Packages for patch releases are also uploaded to the GitHub Release page, but must be manually selected in the gardenlinux/repo to be included for a certain patch release apt repository.  


# Notes 
- Building the package 
   - Build environments 
   - Backport environment 
    - Package Backports 
        - build environment for backports 
            - https://github.com/gardenlinux/repo-debian-snapshot
    - handling null releases (#2736)
- Build Dependency Management 
    - override build dependencies 
    - use default build dependencies
- Testing Packages (#2735)
