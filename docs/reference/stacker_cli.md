# Stacker Command-Line Interface Reference

This document lists the command line interface (CLI) commands for stacker version 1.0.0.

<table>
<tr><td><a href="#stacker">stacker</a></td><td>Stacker builds OCI images</td></tr>
<tr><td><a href="#stacker-build">stacker build</a></td><td>Builds a new OCI image from a stacker yaml file</td></tr>
<tr><td><a href="#stacker-recursive-build">stacker recursive-build</a></td><td>Finds stacker yaml files under a directory and builds all OCI images they define</td></tr>
<tr><td><a href="#stacker-convert">stacker convert</a></td><td>Converts a Dockerfile into a stacker yaml file</td></tr>
<tr><td><a href="#stacker-publish">stacker publish</a></td><td>Publishes OCI images previously built from one or more stacker yaml files</td></tr>
<tr><td><a href="#stacker-chroot-exec">stacker chroot</a></td><td>Runs a command in a chroot (same as <code>stacker exec</code>)</td></tr>
<tr><td><a href="#stacker-clean">stacker clean</a></td><td>Cleans up after a stacker build</td></tr>
<tr><td><a href="#stacker-inspect">stacker inspect</a></td><td>Prints the json representation of an OCI image</td></tr>
<tr><td><a href="#stacker-grab">stacker grab</a></td><td>Grabs a file from the image's filesystem</td></tr>
<tr><td><a href="#stacker-unpriv-setup">stacker unpriv-setup</a></td><td>Does the necessary unprivileged setup for stacker build to work without root</td></tr>
<tr><td><a href="#stacker-gc">stacker gc</a></td><td>Garbage collection of unused OCI imports/outputs snapshots</td></tr>
<tr><td><a href="#stacker-check">stacker check</a></td><td>Checks that all runtime required items (like kernel features) are present</td></tr>
<tr><td><a href="#stacker-help">stacker help</a></td><td>Shows a list of commands or help for one command</td></tr>
</table>

<a name="stacker"></a>

## stacker

        NAME:
        stacker - stacker builds OCI images

        USAGE:
        stacker [global options] command [command options] [arguments...]

        VERSION:
        stacker <version> liblxc <digest>

        COMMANDS:
        build            builds a new OCI image from a stacker yaml file
        recursive-build  finds stacker yaml files under a directory and builds all OCI layers they define
        convert          converts a Dockerfile into a stacker yaml file (experimental, best-effort)
        publish          publishes OCI images previously built from one or more stacker yaml files
        chroot, exec     run a command in a chroot
        clean            cleans up after a `stacker build`
        inspect          print the json representation of an OCI image
        grab             grabs a file from the layer's filesystem
        unpriv-setup     do the necessary unprivileged setup for stacker build to work without root
        gc               gc unused OCI imports/outputs snapshots
        check            checks that all runtime required things (like kernel features) are present
        help, h          Shows a list of commands or help for one command

        GLOBAL OPTIONS:
        --work-dir value      set the working directory for stacker's cache, OCI output and rootfs output
        --stacker-dir value   set the directory for stacker's cache (default: ".stacker")
        --oci-dir value       set the directory for OCI output (default: "oci")
        --roots-dir value     set the directory for the rootfs output (default: "roots")
        --config value        stacker config file with defaults (default: "/home/<username>/.config/stacker/conf.yaml")
        --debug               enable stacker debug mode
        -q, --quiet           silence all logs
        --log-file value      log to a file instead of stderr
        --log-timestamp       whether to log a timestamp prefix
        --storage-type value  storage type (must be "overlay", left for compatibility) (default: "overlay")
        --no-progress         disable progress when downloading container images or files
        --help, -h            show help
        --version, -v         print the version

<a name="stacker-build"></a>

## stacker build

        NAME:
        stacker build - builds a new OCI image from a stacker yaml file

        USAGE:
        stacker build [command options] [arguments...]

        OPTIONS:
        --no-cache                      don't use the previous build cache
        --substitute value              variable substitution in stackerfiles, FOO=bar format
        --substitute-file value         file containing variable substitution in stackerfiles, 'FOO: bar' yaml format
        --on-run-failure value          command to run inside container if run fails (useful for inspection)
        --shell-fail                    exec /bin/sh inside the container if run fails (alias for --on-run-failure=/bin/sh)
        --layer-type value              set the output layer type (supported values: tar, squashfs); can be supplied multiple times (default: "tar")
        --no-squashfs-verity            do not append dm-verity data to squashfs archives
        --require-hash                  require all remote imports to have a hash provided in stackerfiles
        --order-only                    show the build order without running the actual build
        --annotations-namespace value   set OCI annotations namespace in the OCI image manifest (default: "io.stackeroci")
        --stacker-file value, -f value  the input stackerfile (default: "stacker.yaml")

<a name="stacker-recursive-build"></a>

## stacker recursive-build

        NAME:
        stacker recursive-build - finds stacker yaml files under a directory and builds all OCI layers they define

        USAGE:
        stacker recursive-build [command options] [arguments...]

        OPTIONS:
        --no-cache                              don't use the previous build cache
        --substitute value                      variable substitution in stackerfiles, FOO=bar format
        --substitute-file value                 file containing variable substitution in stackerfiles, 'FOO: bar' yaml format
        --on-run-failure value                  command to run inside container if run fails (useful for inspection)
        --shell-fail                            exec /bin/sh inside the container if run fails (alias for --on-run-failure=/bin/sh)
        --layer-type value                      set the output layer type (supported values: tar, squashfs); can be supplied multiple times (default: "tar")
        --no-squashfs-verity                    do not append dm-verity data to squashfs archives
        --require-hash                          require all remote imports to have a hash provided in stackerfiles
        --order-only                            show the build order without running the actual build
        --annotations-namespace value           set OCI annotations namespace in the OCI image manifest (default: "io.stackeroci")
        --stacker-file-pattern value, -p value  regex pattern to use when searching for stackerfile paths (default: "\\/stacker.yaml$")
        --search-dir value, -d value            directory under which to search for stackerfiles to build (default: ".")

<a name="stacker-convert"></a>

## stacker convert

        NAME:
        stacker convert - converts a Dockerfile into a stacker yaml file (experimental, best-effort)

        USAGE:
        stacker convert [command options] [arguments...]

        OPTIONS:
        --docker-file value, -i value      the input Dockerfile (default: "Dockerfile")
        --output-file value, -o value      the output stacker file (default: "stacker.yaml")
        --substitute-file value, -s value  the output file containing detected substitutions (default: "stacker-subs.yaml")

<a name="stacker-publish"></a>

## stacker publish

        NAME:
        stacker publish - publishes OCI images previously built from one or more stacker yaml files

        USAGE:
        stacker publish [command options] [arguments...]

        OPTIONS:
        --stacker-file value, -f value          the input stackerfile (default: "stacker.yaml")
        --stacker-file-pattern value, -p value  regex pattern to use when searching for stackerfile paths (default: "\\/stacker.yaml$")
        --substitute-file value                 file containing variable substitution in stackerfiles, 'FOO: bar' yaml format
        --search-dir value, -d value            directory under which to search for stackerfiles to publish
        --url value                             url where to publish the OCI images
        --username value                        username for the registry where the OCI images are published
        --password value                        password for the registry where the OCI images are published
        --skip-tls                              skip tls verify on upstream registry
        --tag value                             tag to be used when publishing
        --substitute value                      variable substitution in stackerfiles, FOO=bar format
        --show-only                             show the images to be published without actually publishing them
        --force                                 force publishing the images present in the OCI layout even if they should be rebuilt
        --layer-type value                      set the output layer type (supported values: tar, squashfs); can be supplied multiple times (default: "tar")
        --image value                           image to be published; can be specified multiple times

<a name="stacker-chroot-exec"></a>

## stacker chroot, exec

        NAME:
        stacker chroot - run a command in a chroot

        USAGE:
        stacker chroot [command options] [tag] [cmd]

        <tag> is the built tag in the stackerfile to chroot to, or the first tag if
        none is specified.

        <cmd> is the command to run, or /bin/sh if none is specified. To specify cmd,
        you must specify a tag.

        OPTIONS:
        --stacker-file value, -f value  the input stackerfile (default: "stacker.yaml")
        --substitute value              variable substitution in stackerfiles, FOO=bar format

<a name="stacker-clean"></a>

## stacker clean

        NAME:
        stacker clean - cleans up after a `stacker build`

        USAGE:
        stacker clean [command options] [arguments...]

        OPTIONS:
        --all  no-op; this used to do something, and is left in for compatibility

<a name="stacker-inspect"></a>

## stacker inspect

        NAME:
        stacker inspect - print the json representation of an OCI image

        USAGE:
        stacker inspect [tag]

        <tag> is the tag in the stackerfile to inspect. If none is supplied, inspect
        prints the information on all tags.

<a name="stacker-grab"></a>

## stacker grab

        NAME:
        stacker grab - grabs a file from the layer's filesystem

        USAGE:
        stacker grab <tag>:<path>

        <tag> is the tag in a built stacker image to extract the file from.

        <path> is the path to extract (relative to /) in the image's rootfs.

<a name="stacker-unpriv-setup"></a>

## stacker unpriv-setup

        NAME:
        stacker unpriv-setup - do the necessary unprivileged setup for stacker build to work without root

        USAGE:
        stacker unpriv-setup [command options] [arguments...]

        OPTIONS:
        --uid value       the user to do setup for (defaults to $SUDO_UID from env)
        --gid value       the group to do setup for (defaults to $SUDO_GID from env)
        --username value  the username to do setup for (defaults to $SUDO_USER from env)

<a name="stacker-gc"></a>

## stacker gc

        NAME:
        stacker gc - gc unused OCI imports/outputs snapshots

        USAGE:
        stacker gc [arguments...]

<a name="stacker-check"></a>

## stacker check

        NAME:
        stacker check - checks that all runtime required things (like kernel features) are present

        USAGE:
        stacker check [arguments...]

<a name="stacker-help"></a>

## stacker help

        NAME:
            - Shows a list of commands or help for one command

        USAGE:
            [command]

