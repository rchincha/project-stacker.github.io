# What's New

## [v0.40.1](https://github.com/project-stacker/stacker/releases/tag/v0.40.1)

* Support for `scratch`

Prior to v0.40.1, `stacker` did not support empty root filesystems to be used a
base container image. The support has now been [added](reference/stacker_file.md#from) which can be used to host
statically built binaries.

* Support for `import`ing content into container image

Prior to v0.40.1, copying content into a layer permanently involved bind
mounting a shell such as busybox and invoking appropriate commands using the
`run` directive. Now `import` directive [allows](reference/stacker_file.md#import-dest) for the `dest` option to achieve
the same.

* Some `squashfs` improvements

While building squashfs layers, use `squashfuse_ll` if available which is faster.
