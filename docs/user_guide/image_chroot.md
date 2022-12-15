# Chrooting Into Built Images

After an image is built, you can run a command in a chroot in that built image.

```
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
```
