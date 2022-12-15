# Publishing images

After building the image, you can publish the image to an [OCI Distribution
Spec](https://github.com/opencontainers/distribution-spec/blob/main/spec.md)
conformant registry.

```
NAME:
   stacker publish - publishes OCI images previously built from one or more stacker yaml files

USAGE:
   stacker publish [command options] [arguments...]

OPTIONS:
   --stacker-file value, -f value          the input stackerfile (default: "stacker.yaml")
   --stacker-file-pattern value, -p value  regex pattern to use when searching for stackerfile paths (default: "\\/stacker.yaml$")
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
```
