# Installing Stacker

Stacker is a single static binary tool with most of the dependencies built into 
the binary. Stacker, however, depends on specific kernel capabilities and system
tools to provide various features.

```bash title="Download Stacker"
wget https://github.com/project-stacker/stacker/releases/latest/download/stacker
chmod +x ./stacker
sudo cp ./stacker /usr/bin/stacker
stacker --version
stacker check
```
## Kernel dependencies

Stacker requires overlayfs backend. That works with any kernel >= 4.14. 
However, for unprivileged usage, the overlayfs backend requires a new 
kernel change available on all kernels >= 5.8. 

!!! info 

    Overlayfs kernel patches required for unprivileged use:

    * vfs: allow unprivileged whiteout creation - a3c751a50fe6 
    * ovl: unprivieged mounts - 459c7c565ac3

Some distributions may have ported these patches into older versions of their
kernels. For example, Ubuntu 20.04 and 22.04 kernels already have these patches.

Stacker includes checks to ensure that it can run with all these requirements, and will fail if it can't perform a function it should be able perform.

```bash title="Stacker Check"
stacker check && echo "stacker is ready to use!"
```

## Overlay filesystem

An underlying overlayfs cannot back stacker becuase the stacker needs to create 
whiteout files, and the kernel (rightfully) forbids manual creation of whiteout 
files on overlay filesystems. No additional userspace dependencies are required
to use the overlayfs backend.

!!! warning

    Do not use a overlayfs based filesystem as a storage for stacker root
    directory.

## Unprivileged setup

Running stacker as an unprivileged user requires stacker to run as part of a `user`
namespace owned by the user that executed the command. Stacker will 
map `65k` user and group IDs to meet the POSIX standard. So, to run stacker,
the user's `/etc/sub{u,g}id` should be configured with enough UIDs to map components
correctly. This configuration can be run automatically via 
`stacker unpriv-setup`.

```bash title="Stacker unprivileged setup"
sudo stacker unpriv-setup
cat /etc/subgid
cat /etc/subuid
```

## Squashfs support

In order to generate squashfs images, stacker invokes the `mksquashfs` binary.
This binary needs to be installed and available in `$PATH`.

```bash title="Install mksquashfs on ubuntu"
sudo apt-get install -y squashfs-tools
```
