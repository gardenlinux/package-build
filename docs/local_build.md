# Local Development
By following these steps, you will be able to build the container image for your desired architecture and generate packages using the containerized environment. This enables efficient local development, allowing you to iterate on your code, test packages, and validate changes effectively.

## Build container

To build the container, follow the steps below:

Install `podman` on your system: https://podman.io/docs/installation

Update unqualified search in `/etc/containers/registries.conf`:

```
unqualified-search-registries = ["docker.io"]
```

On amd64 architecture:
```
podman build --build-arg arch=amd64 -f container/build.containerfile -t build:amd64 .
```

On arm64 architecture:
```
podman build --build-arg arch=arm64v8 -f container/build.containerfile -t build:arm64v8 .
```

Note: You do not need to use `sudo` as the build should work just fine with unprivileged containers.

## Build a package

To build a package, perform the following steps inside the package repository:

Create a .build directory:
```
mkdir .build
```

On amd64 architecture:
```
podman run --rm -v $PWD:/input -v $PWD/.build:/output build:amd64 build
```

On arm64 architecture:
```
podman run --rm -v $PWD:/input -v $PWD/.build:/output build:arm64v8 build
```

Alternatively, you can use pre-built containers from the GitHub Container Registry (ghcr) by accessing the following link: https://github.com/gardenlinux/package-build/pkgs/container/package-build. These pre-built containers can be used as an alternative to building the container locally.