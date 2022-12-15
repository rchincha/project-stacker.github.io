# A "Hello Stacker" Image

Stacker builds OCI images using the YAML definition in a file referred to as 
the stacker file. So let's create a stacker file definition that packages a 
simple "Hello Stacker!" script into an OCI container image.

```yaml title="Hello Stacker"
hello-stacker:
  from:
    type: docker
    url: docker://zothub.io/tools/busybox:stable
  run: |
    mkdir -p /hello-stacker-app
    echo 'echo "Hello Stacker!"' > /hello-stacker-app/hello.sh
    chmod +x /hello-stacker-app/hello.sh
  entrypoint: /hello-stacker-app/hello.sh

```

`hello-stacker`, the first key in the above YAML, builds an OCI image named
__hello-stacker__.

The `from` section defines that `hello-stacker` image builds on top of an 
existing image called `zothub.io/tools/busybox:stable`, and that the base image is of type 
`docker` downloaded from URL `docker://zothub.io/tools/busybox:stable`.

The `run` section defines a build script. This script is executed on top of the
ubuntu image's root file system, creates a file called 
`/hello-stacker-app/hello.sh`, and makes it executable.

`entrypoint` defines that `/hello-stacker-app/hello.sh` is executed as the `init`
process on creating this container by a container runtime.

We can create the `hello-stacker` image with this input as a
stacker file name `hello_stacker.yaml`.

```bash title="Hello Stacker Build"
$ stacker build -f hello_stacker.yaml
preparing image hello-stacker...
loading docker://zothub.io/tools/busybox:stable
Copying blob cf92e523b49e done
Copying config cb52c703ef done
Writing manifest to image destination
Storing signatures
+ mkdir -p /hello-stacker-app
+ echo echo "Hello Stacker!"
+ chmod +x /hello-stacker-app/hello.sh
filesystem hello-stacker built successfully
```
stacker build used `hello-stacker.yaml` as input, downloaded the `zothub.io/tools/busybox:stable`
image from docker hub and generated an OCI image with tag `hello-stacker`. We 
can verify this using `stacker inspect`:

```bash title="Stacker Inspect"
$ stacker inspect
hello-stacker
        layer 0: cf92e523b49e... (30 MB, application/vnd.oci.image.layer.v1.tar+gzip)
        layer 1: 2910d371807c... (176 B, application/vnd.oci.image.layer.v1.tar+gzip)
Annotations:
  io.stackeroci.stacker.stacker_yaml: hello-stacker:
  from:
    type: docker
    url: docker://zothub.io/tools/busybox:stable
  run: |
    mkdir -p /hello-stacker-app
    echo 'echo "Hello Stacker!"' > /hello-stacker-app/hello.sh
    chmod +x /hello-stacker-app/hello.sh
  entrypoint: /hello-stacker-app/hello.sh


Image config:
{
  "created": "2022-10-24T03:47:24.374578534Z",
  "author": "stacker-dev",
  "architecture": "amd64",
  "os": "linux",
  "config": {
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Entrypoint": [
      "/hello-stacker-app/hello.sh"
    ]
  },
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:17f623af01e277c5ffe6779af8164907de02d9af7a0e161662fc735dd64f117b",
      "sha256:a33ac624796108439088267fb525f1038dccb6f6a50796a1593786a31d97a4bd"
    ]
  },
  "history": [
    {
      "created": "2022-10-04T23:35:20.465021967Z",
      "created_by": "/bin/sh -c #(nop) ADD file:6cd2e13356aa5339c1f2abd3c210a52f6ed74fae05cd61aa09f37b6a4764f65c in / "
    },
    {
      "created": "2022-10-04T23:35:20.857335994Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"bash\"]",
      "empty_layer": true
    },
    {
      "created": "2022-10-24T03:47:24.372946511Z",
      "created_by": "stacker build of hello-stacker"
    },
    {
      "created": "2022-10-24T03:47:24.374578534Z",
      "created_by": "stacker build",
      "author": "root@ins15-pp24-ru11",
      "empty_layer": true
    }
  ]
}
```

The next thing to note is that if we rebuild the image without any modifications
to the stacker file, less things happen:

```bash title="stacker caching"
$ stacker build -f hello_stacker.yaml
preparing image hello-stacker...
loading docker://zothub.io/tools/busybox:stable
Copying blob cf92e523b49e skipped: already exists
Copying config cb52c703ef done
Writing manifest to image destination
Storing signatures
found cached layer hello-stacker
```

Stacker will cache all inputs to the stacker file and rebuilds the image only if
something changed in the stacker file. The cache and the metadata required to 
track the build state live in the `.stacker` directory where the stacker command 
is run. Stacker cache can be cleaned using `stacker clean` command.
