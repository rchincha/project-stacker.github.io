# Grab Content From Built Images

After an image is built, you can grab a file from the built image. This is
useful to inspect the file or its contents to ensure the build process has
completed properly.

```
NAME:
   stacker grab - grabs a file from the layer's filesystem

USAGE:
   stacker grab <tag>:<path>

<tag> is the tag in a built stacker image to extract the file from.

<path> is the path to extract (relative to /) in the image's rootfs.
```
