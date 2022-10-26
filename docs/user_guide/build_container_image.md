Usually, developers use a two-stage build process to build a container image. 
In the first stage, make the binary for the container image, and in the second
stage, build the runtime container image, including that binary. To accomplish
this, stacker "build only" container builds the required binary and does not
generate the corresponding OCI layer. Then copy this binary into the RFS of the 
runtime image such that the final container has __only__ the required binary.

```bash title="Two-stage container image build"
cat > "hello_stacker.go" << EOF
package main

import "fmt"

func main() {
    fmt.Println("Hello Stacker!")
}
EOF

cat > "hello.stacker.yaml" << EOF
build-hello-stacker:
  from:
    type: docker
    url: docker://zothub.io/c3/ubuntu/go-devel-amd64:1.19.2
  import:
    - ./hello_stacker.go
  build_only: true
  run: |
    # Source Go toolchain env
    . /etc/profile

    # Setup Go ENV
    mkdir -p /go
    export GOPATH=/go
    mkdir -p /go/src
    cd /go/src

    # Copy source code to go path
    cp /stacker/hello_stacker.go .

    # Build a static go binary
    go build -ldflags="-extldflags=-static" -o hello_stacker.static hello_stacker.go

    cp hello_stacker.static /
hello:
  from:
    type: docker
    url: docker://zothub.io/tools/busybox:stable
  import:
    - stacker://build-hello-stacker/hello_stacker.static
  run: |
    cp /stacker/hello_stacker.static /hello_stacker
    chmod +x /hello_stacker
    /hello_stacker # test
EOF

stacker build -f hello.stacker.yaml
preparing image build-hello-stacker...
copying /dev/shm/ravi/hello_stacker.go
loading docker://zothub.io/c3/ubuntu/go-devel-amd64:1.19.2
Copying blob b900f44d647a done
Copying config 3ece5b544e done
Writing manifest to image destination
Storing signatures
+ . /etc/profile
+ export 'HOME=/go'
+ export 'GOROOT=/opt/go'
+ export 'PATH=/opt/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
+ mkdir -p /tmp/go/cache
+ export 'GOCACHE=/tmp/go/cache'
+ mkdir -p /go
+ export 'GOPATH=/go'
+ mkdir -p /go/src
+ cd /go/src
+ cp /stacker/hello_stacker.go .
+ go build '-ldflags=-extldflags=-static' -o hello_stacker.static hello_stacker.go
+ cp hello_stacker.static /
preparing image hello...
loading docker://zothub.io/tools/busybox:stable
Copying blob f5b7ce95afea done
Copying config 74c82eccc6 done
Writing manifest to image destination
Storing signatures
+ cp /stacker/hello_stacker.static /hello_stacker
+ chmod +x /hello_stacker
+ /hello_stacker
Hello Stacker!
filesystem hello built successfully
```

!!! info

    `stacker://build-hello-stacker/hello_stacker.static` import statement imports
    the specific binary file from the `build-hello-stacker` that is present in the
    build cache

Lets verify that the hello layer has /hello_stacker file:

```bash title="stacker chroot"
stacker chroot -f hello.stacker.yaml hello
This chroot is temporary, any changes will be destroyed when it exits.
/ # ls -al /hello_stacker
-rwxrwxr-x    1 root     root       1809694 Oct 26 01:27 /hello_stacker
/ # exit
```