# OverlayFS Support

stacker relies on [overlayFS](https://docs.kernel.org/filesystems/overlayfs.html) in the Linux kernel, which should be available in kernel versions 4.x and above.

You can also check by running the command `lsmod | grep overlay` if compiled
and available as a module or if compiled inline by enabling `CONFIG_OVERLAY_FS`
in the Linux kernel build configuration.
