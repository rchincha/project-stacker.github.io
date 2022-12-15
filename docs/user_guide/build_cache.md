# Build Cache

A build cache with snapshot support is maintained throughout the build process
in order to store intermediate build artifacts and avoid unnecessary rebuilds.
Furthermore, you can also avoid rebuilding layers across many stacker build
invocations by preserving this build cache.
