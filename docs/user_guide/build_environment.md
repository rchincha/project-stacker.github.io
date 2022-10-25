Stacker builds container images in a canonically defined environment, allowing 
stacker to guarantee repeatable builds by reproducing the same environment for 
all the builds for a given version of the stacker file.

## Runtime Environment

Except for the term and proxy settings, none of the host environment
variables leak into the stacker build environment. The `run` section of your 
build script can depend on these environment variables to perform the build.

Let's examine the shell environment variables that stacker exposes to the build
script in the run section of your stacker file: 

```bash title="Stacker Build Environment"
cat > env_stacker.yaml << EOF
env-stacker:
  from:
    type: docker
    url: docker://zothub.io/tools/busybox:stable
  run: |
    env
  build_only: true
EOF

stacker build -f env_stacker.yaml
preparing image env-stacker...
loading docker://zothub.io/tools/busybox:stable
Copying blob f5b7ce95afea skipped: already exists
Copying config 74c82eccc6 done
Writing manifest to image destination
Storing signatures
cache miss because layer definition was changed
+ env
HTTPS_PROXY=http://173.36.224.109:8080
no_proxy=localhost,127.0.0.1,localaddress,.localdomain.com,.cisco.com
SHLVL=1
NO_PROXY=localhost,127.0.0.1,localaddress,.localdomain.com,.cisco.com
container=lxc
https_proxy=http://173.36.224.109:8080
TERM=screen-256color
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
STACKER_LAYER_NAME=env-stacker
PWD=/
```

Note that only `HTTPS_PROXY`, `https_proxy`, `NO_PROXY`, `no_proxy`, and `TERM`
are imported from the host, all other variables are standard shell variables.

## Root File System (RFS)

Stacker creates a root file system defined by the layer in the `from` section and
launches the build script in the `run` section using this file system. Stacker
expects a "sane" file system in the base container so that it can execute `sh` to 
implement the `run` section.

Let's examine the root file system that stacker exposes to the `run` script in
the following example:

```bash title="Build RFS"
cat > rfs_stacker.yaml << EOF
rfs-stacker:
  from:
    type: docker
    url: docker://zothub.io/tools/busybox:stable
  run: |
    ls -al /
    find /etc
    which cat
  build_only: true
EOF

stacker build -f rfs_stacker.yaml
preparing image rfs-stacker...
loading docker://zothub.io/tools/busybox:stable
Copying blob f5b7ce95afea skipped: already exists
Copying config 74c82eccc6 done
Writing manifest to image destination
Storing signatures
+ ls -al /
total 0
drwxr-xr-x    1 root     root           120 Oct 25 18:30 .
drwxr-xr-x    1 root     root           120 Oct 25 18:30 ..
drwxr-xr-x    2 root     root          8080 Oct  3 22:04 bin
drwxr-xr-x    4 root     root           340 Oct 25 18:30 dev
drwxr-xr-x    1 root     root            60 Oct 25 18:30 etc
drwxr-xr-x    2 nobody   nobody          40 Oct  3 22:04 home
dr-xr-xr-x 2299 nobody   nobody           0 Oct 25 18:30 proc
drwx------    2 root     root            40 Oct  3 22:04 root
drwxr-xr-x    2 root     root            60 Oct 25 18:30 stacker
dr-xr-xr-x   13 nobody   nobody           0 Sep 22 22:12 sys
drwxrwxrwt    2 root     root            40 Oct  3 22:04 tmp
drwxr-xr-x    3 root     root            60 Oct  3 22:04 usr
drwxr-xr-x    4 root     root            80 Oct  3 22:04 var
+ find /etc
/etc
/etc/shadow
/etc/passwd
/etc/network
/etc/network/if-up.d
/etc/network/if-pre-up.d
/etc/network/if-post-down.d
/etc/network/if-down.d
/etc/localtime
/etc/group
/etc/resolv.conf
+ which cat
/bin/cat
```

The `docker://zothub.io/tools/busybox:stable` container defines the above file
system, which has all the necessary utilities like /bin/cat and a basic system
configuration required to operate most of the Linux utilities.

## Newtworking Setup

Stacker builds the container images in the host network namespace. It bind 
mounts the host's /etc/resolv.conf into the build container's root file system 
to allow for correct DNS resolution as defined by the host. Finally, stacker 
re-exports the proxy settings to the environment to enable proxy-based access to
any artifacts required to build the container image.

Let's examine the networking setup that stacker exposes to the `run` script in
the following example:

```bash title="Networking setup"
cat > network_stacker.yaml << EOF
network-stacker:
  from:
    type: docker
    url: docker://zothub.io/tools/busybox:stable
  run: |
    echo "HTTPS_PROXY=$HTTPS_PROXY"
    cat /etc/resolv.conf
    ip addr
  build_only: true
EOF

stacker build -f network_stacker.yaml
preparing image network-stacker...
loading docker://zothub.io/tools/busybox:stable
Copying blob f5b7ce95afea skipped: already exists
Copying config 74c82eccc6 done
Writing manifest to image destination
Storing signatures
+ echo HTTPS_PROXY=http://173.36.224.109:8080
HTTPS_PROXY=http://173.36.224.109:8080
+ cat /etc/resolv.conf
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search cisco.com insieme.local
+ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp97s0f0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether 3c:fd:fe:7f:27:a0 brd ff:ff:ff:ff:ff:ff
3: enp97s0f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether 3c:fd:fe:7f:27:a1 brd ff:ff:ff:ff:ff:ff
4: enp97s0f2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether 3c:fd:fe:7f:27:a2 brd ff:ff:ff:ff:ff:ff
5: enp97s0f3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether 3c:fd:fe:7f:27:a3 brd ff:ff:ff:ff:ff:ff
6: enp1s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq qlen 1000
    link/ether b4:96:91:a4:44:d4 brd ff:ff:ff:ff:ff:ff
    inet 172.20.61.165/24 brd 172.20.61.255 scope global enp1s0f0
       valid_lft forever preferred_lft forever
    inet6 fe80::b696:91ff:fea4:44d4/64 scope link
       valid_lft forever preferred_lft forever
7: enp1s0f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq qlen 1000
    link/ether b4:96:91:a4:44:d5 brd ff:ff:ff:ff:ff:ff
8: lxcbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue qlen 1000
    link/ether 00:16:3e:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.0.3.1/24 brd 10.0.3.255 scope global lxcbr0
       valid_lft forever preferred_lft forever
9: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue
    link/ether 02:42:30:0d:77:ad brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

Since stacker is only used for building images, this is safe and intuitive for 
users on corporate networks with complicated proxy and other setups. However, 
it does mean that container packaging that expects to be able to modify things
in `/sys` will fail since /sys is bind mounted from the host's `/sys`. `sysfs`
cannot be mounted in a network namespace that a user doesn't own.
