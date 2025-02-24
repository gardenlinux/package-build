# Developer Hints

This document can help you get a broken build working, most probably this happens when you're updating an upstream package version.

## How to debug package builds

Changes to make `package-build` more debug friendly:

```diff
diff --git a/bin/source b/bin/source
index 1009b1e..ad594f3 100755
--- a/bin/source
+++ b/bin/source
@@ -96,7 +96,9 @@ version=
 version_suffix=gardenlinux~dev

 dir="$(mktemp -d)"
-trap 'cd / && rm -rf "$dir"' EXIT
+trap 'echo $dir' EXIT
+echo "Linux sources are located in:"
+echo "\t$dir"

 mkdir "$dir/src"

diff --git a/build b/build
index c7a12ce..14ee7b0 100755
--- a/build
+++ b/build
@@ -63,7 +63,7 @@ if [ -n "$build_dep_dir" ]; then
 fi

 if [ -z "$skip_source" ]; then
-       podman run --arch "$arch" --rm "${mount_opts[@]}" -w "/opt/package_build/workdir" "$container" /opt/package_build/bin/source
+       podman run -it --arch "$arch" "${mount_opts[@]}" -w "/opt/package_build/workdir" "$container" /opt/package_build/bin/source
 fi
 if [ -z "$skip_binary" ]; then
        podman run --arch "$arch" --rm "${mount_opts[@]}" -w "/opt/package_build/workdir" "$container" /opt/package_build/bin/binary "$build"
```

Now you can put a `bash` statement in your `prepare_source` script at any place like this

```bash
apt_src --ignore-orig containerd
bash # use this only as a 'debugger breakpoint' and remove once you're done debugging
```

This allows you to interactivly debug the build.

## Fixing patches

It might happen that patches we have don't apply to a new version of the tool.
This can be fixed by setting up an interactive build container as described above and then applying and manually fixing patches.

It helps to be familiar with [quilt](https://en.wikipedia.org/wiki/Quilt_(software))

[tldr cheat sheet for quilt](https://tldr.inbrowser.app/pages/common/quilt)

Required configuration

```bash
export QUILT_PATCHES=debian/patches
```

Useful commands:

`quilt push` apply a single patch

`quilt push -a` apply all patches

`quilt push -f` force apply patches

`quilt refresh` update patch files after manually editing source

