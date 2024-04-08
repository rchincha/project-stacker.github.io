# What's New

## [v1.0.0](https://github.com/project-stacker/stacker/releases/tag/v1.0.0-rc9)

### Convert a Dockerfile for stacker

- A new [`stacker convert`](reference/stacker_cli.md#stacker-convert) command performs a conversion of a Dockerfile into a stacker.yaml file. During the conversion, some variables from the Dockerfile may be exported to a substitution file that can be included in `stacker build` using the `--substitute-file <filename>` command option.

    :pencil2: The conversion is a best-effort process and may not be successful in all cases.

### Publish specific images

- By default, the [`stacker publish`](reference/stacker_cli.md#stacker-publish) command pushes all images in a stacker.yaml file instead of only the required images. Using a new command option, `--image <value>`, you can explicitly specify which images are to be published.  This command option can be specified multiple times, selecting each image to be included.

### Specify a single working directory

- A new [`stacker`](reference/stacker_cli.md#stacker) command option, `--work-dir`, sets the working directory for stacker's cache, OCI output, and rootfs output. The existing command options `--stacker-dir`, `--oci-dir`, and `--roots-dir` can then be omitted or used to override the `--work-dir` setting.

### Import contents when no shell exists in the base image

- Import directives can include destination paths. This feature is useful to simplify `run` section scripts, and for when images are built without a base image. With no base image, there is no shell to run the script in a `run` section. Prior to this change, a `run:` section was required to invoke a shell and to explicitly copy files to be imported into the image. For example, you can now write a directive such as the following, with no `run:` section:

        test:
          from:
            type: scratch
          imports:
            - path: test_file
              dest: /files/
            - path: test_file2
              dest: /file2

### Generate SBOMs during the build

- Changes added in [OCI Distribution Spec v1.1.0-rc3](https://github.com/opencontainers/distribution-spec/releases/tag/v1.1.0-rc3) and [OCI Image Spec v1.1.0-rc4](https://github.com/opencontainers/image-spec/releases/tag/v1.1.0-rc4) (summarized [here](https://opencontainers.org/posts/blog/2023-07-07-summary-of-upcoming-changes-in-oci-image-and-distribution-specs-v-1-1/)) allow arbitrary artifact types and references. These changes support software supply chain use cases such as SBOMs.

- For a demonstration of an OCI artifacts workflow that generates an SBOM, see [Software Provenance Workflow Using OCI Artifacts](user_guide/generate_sbom.md).

### Report kernel version and fs type

- The [`stacker check`](reference/stacker_cli.md#stacker-check) command now reports this information.

### Build improvements

***

## [v0.40.1](https://github.com/project-stacker/stacker/releases/tag/v0.40.1)

### Support for `scratch`

- Prior to v0.40.1, `stacker` did not support empty root filesystems to be used as a base container image. The support has now been [added](reference/stacker_file.md#from) which can be used to host statically built binaries.

### Support for `import`ing content into container image

- Prior to v0.40.1, copying content into a scratch image permanently involved bind mounting a shell such as busybox and invoking appropriate commands using the `run` directive. Now the `import` directive [allows](reference/stacker_file.md#import-dest) for the `dest` option to achieve the same.

### Publish with substitutions specified in a file
  
- Using a new `build` command option, [`substitute-file <value>`](reference/stacker_cli.md#stacker-build), you can now declare variable substitutions in a file instead of the command line. The substitution file uses a 'FOO: bar' key-value yaml format to declare substitutions.

### Some `squashfs` improvements

- While building squashfs layers, use `squashfuse_ll` if available which is faster.
