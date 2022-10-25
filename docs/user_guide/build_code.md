A common first step before building a container image is to build your code in a
consistent environment. Stacker provides a hermetically sealed environment in a
build-only container to produce the binaries and other artifacts required for
the container image. Whether you are building a container image or not, it is a 
good idea to use stacker's container-based builds to produce your binaries,
especially when you need a consistent build environment for all the developers 
collaborating on the same project.

## Go Code Example

As an example, let's build a simple go file using stacker:

```bash title="Go build using stacker"
cat > "hello_stacker.go" << EOF
package main

import "fmt"

func main() {
	fmt.Println("Hello Stacker!")
}
EOF

cat > "go_build.stacker.yaml" << EOF
go-build-hello-stacker:
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

    # Build code
    go build -o hello_stacker hello_stacker.go

    # Test code
    ./hello_stacker
EOF

stacker build -f go_build.stacker.yaml
preparing image hello-go-stacker...
using cached copy of /dev/shm/ravi/hello_stacker.go
loading docker://zothub.io/c3/ubuntu/go-devel-amd64:1.19.2
Copying blob b900f44d647a skipped: already exists
Copying config 3ece5b544e done
Writing manifest to image destination
Storing signatures
cache miss because layer definition was changed
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
+ go build -o hello_stacker hello_stacker.go
+ ./hello_stacker
Hello Stacker!
```

The above script creates a go file called `hello_stacker.go`, then uses a go 
development container from `zothub.io/c3/ubuntu/go-devel-amd:1.19.2` to build 
`hello_stacker.go`.

Let's look at details of the stacker file sections used:

| Stacker file Keyword | Description                                           |
| -------------------- | ----------------------------------------------------- |
| `from`               | specifies the base container to be used for this build|
| `import`             | import file from host into the build container        |
| `build_only`         | mark this as a build container to avoid layer creation|
| `run`                | build script to execute                               |

Note `cp /stacker/hello_stacker.go .` in the `run` script. Stacker bind mounts
all the imported files under a special read-only directory `/stacker`. This
directory can be used in the `run` script to access the host mounted files in
the build script.
