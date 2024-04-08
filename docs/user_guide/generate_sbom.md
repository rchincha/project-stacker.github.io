# Software Provenance Workflow Using SBOMs as OCI Artifacts

!!! info
    This article demonstrates a workflow for building an image while simultaneously creating a Software Bill of Materials (SBOM) as a related OCI artifact.

## About SBOMs for container images

The recent [Executive Order on Improving the Nation's Cybersecurity](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/) provides guidance for hardening the software supply chain, including "providing a purchaser a Software Bill of Materials (SBOM) for each product directly or by publishing it on a public website."

An SBOM is a comprehensive inventory of software components used in building a software image. The SBOM includes descriptions of all components, including libraries and modules, their relationships and hierarchies, and software supply chain origin information for each and every component of the image. The transparency provided by a detailed SBOM gives the user greater confidence in the security of the image and simplifies vulnerability assessment. 

You can create an SBOM for an existing container image by using commonly-available third-party tools to scan the image and generate the SBOM. However, as developers work to reduce the size of images, in some cases there may not be enough data or context to produce a correct SBOM. A better approach is to create the SBOM as part of the original build process. Stacker provides this capability.


## Workflow

The following sections describe a step-by-step workflow for creating a stacker YAML file, then building and publishing an alpine container image that includes the SBOM. 

This example workflow implements the following scenario:

- The stacker YAML file is named `b.yaml`
- The workflow builds an alpine container image
- The image is pushed to an OCI registry that is assumed to be running at `localhost:8080`

The first three steps in the workflow modify the stacker YAML file. To view the full stacker file used in this example workflow, see [Reference: Full stacker file](#full-yaml).

### Step 1: stacker file: Enable SBOM creation

The build-time creation of an SBOM is enabled by including the `bom` attribute in the stacker YAML file (with `generate: true`), along with `annotations`. Both `bom` and `annotations` are required. This excerpt shows an example of the necessary sections and settings:

    bom-test:
      from:
          type: docker
          url: docker://alpine:edge
      bom:
          generate: true
      annotations:
          org.opencontainers.image.authors: bom-test
          org.opencontainers.image.vendor: bom-test
          org.opencontainers.image.licenses: MIT

### Step 2: stacker file: Discover all installed packages

To assist in creating the SBOM, stacker includes a tool for discovering the packages that will be present in the image.  Include the following `run` command in your stacker YAML file:

      run: |
          /stacker/tools/static-stacker bom discover


### Step 3: stacker file: Enable cleanup after the build (optional)

When the build is complete, you can remove unnecessary files to reduce the image size, as in this example for an alpine build:

      run: | 
        <...other commands...>
        rm -f /etc/alpine-release /etc/apk/arch /etc/apk/repositories /etc/apk/world /etc/issue /etc/os-release /etc/secfixes.d/alpine /lib/apk/db/installed /lib/apk/db/lock /lib/apk/db/scripts.tar /lib/apk/db/triggers 


### Step 4: Build the image

Run the `stacker build` command, using the modified stacker file.
 
    $ stacker build -f b.yaml


### Step 5: Publish the image to the OCI registry

Run the `stacker publish` command to publish the container image to the OCI registry.

    $ ./stacker publish \
        --stacker-file b.yaml \
        --tag latest \
        --url docker://localhost:8080/ \
        --skip-tls


### Step 6: Inspect the artifact tree

!!! info
    Changes added in [OCI Distribution Spec v1.1.0-rc3](https://github.com/opencontainers/distribution-spec/releases/tag/v1.1.0-rc3) and [OCI Image Spec v1.1.0-rc4](https://github.com/opencontainers/image-spec/releases/tag/v1.1.0-rc4) (summarized [here](https://opencontainers.org/posts/blog/2023-07-07-summary-of-upcoming-changes-in-oci-image-and-distribution-specs-v-1-1/)) allow arbitrary artifact types and references. These changes support software supply chain use cases such as SBOMs.

Run `regctl artifact tree` to display the artifact tree of the container image in the registry. The artifact tree for this image includes two SBOM-related artifact files.

    $ ./regctl artifact tree localhost:8080/bom-test:latest

    Ref: localhost:8080/bom-test:latest  
    Digest: sha256:9172c5f692f2c65e4f773448503b21dba2de6454bd159905c4bf6d83176e4ea3
    Referrers:  
       - sha256:9c0655368b10ca4b2ffe39e4dd261fb89df25a46ae92d6eb4e6e1792a451883e: application/spdx+json
       - sha256:9df25a46ae92d6eb4e6e1792a451883e9c0655368b10ca4b2ffe39e4dd261fb8: application/vnd.stackerbuild.inventory+json



The displayed artifact tree shows that the original image (`localhost:8080/bom-test:latest`) contains two direct referrers:

- `application/spdx+json` is a comprehensive [SPDX](https://spdx.dev/) SBOM file in JSON format
- `application/vnd.stackerbuild.inventory+json` is a summary inventory, listing the file name, file size, SHA digest, and mode of every file in the SBOM 

The first two entries in the inventory file are shown here:

    $ cat .stacker/artifacts/bom-test/inventory.json | jq . | less
    
    {
      {
          "path": "/bin/arch",
          "size": 12,
          "checksum": "",
          "mode": "01000000777"
      },
      {
          "path": "/bin/ash",
          "size": 12,
          "checksum": "",
          "mode": "01000000777"
      }, 
      ...


## Adding additional files to a package and SBOM

 You can add to the image one or more files or directories that are not otherwise included in the build. This capability is useful if, for example, you are building a well-known distribution but you want to include some other content in the image. If the extra content is added after the `static-stacker bom discover` tool in the `run` section of the stacker file, you must explicitly provide the SBOM information in the `bom` section. Otherwise the SBOM creation will due to missing SBOM information, causing the build to fail.
 
 Use the `bom/packages` attribute in the stacker YAML file to provide SBOM information for the additional files or directory paths.  In the following example, the new content is added as a package. A name (`pkg1`), version, and license information is provided for the package.  Under the `paths` attribute, provide a bracketed, comma-separated list of the files or directories to add. In this example, a single file named `file1` is added.

    bom:
      generate: true
      packages:
        - name: pkg1
          version: 1.0.0
          license: Apache-2.0
          paths: [/file1]

With the SBOM information provided, you can now safely use `file1` in the `run` section, as shown here:

      run: |
          /stacker/tools/static-stacker bom discover
          touch /file1



<a name="full-yaml"></a>

## Reference: Full stacker file

> :bulb: To copy the file to the clipboard, click the copy icon that appears in the upper right corner of the expanded text box.

<details>
  <summary markdown="span">Click here to view the complete stacker file for this workflow example</summary>

```shell
bom-test:
  from:
    type: docker
    url: docker://alpine:edge
  bom:
    generate: true
    packages:
      - name: pkg1
        version: 1.0.0
        license: Apache-2.0
        paths: [/file1]
  annotations:
    org.opencontainers.image.authors: bom-test
    org.opencontainers.image.vendor: bom-test
    org.opencontainers.image.licenses: MIT
  run: |
    # discover installed pkgs
    /stacker/tools/static-stacker bom discover
    # run our cmds
    ls -al  /
    # some changes
    touch /file1
    # cleanup
    rm -f /etc/alpine-release /etc/apk/arch /etc/apk/repositories /etc/apk/world /etc/issue /etc/os-release /etc/secfixes.d/alpine /lib/apk/db/installed /lib/apk/db/lock /lib/apk/db/scripts.tar /lib/apk/db/triggers 
```

</details>
