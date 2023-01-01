# OCI Image Layout

Stacker is a standalone OCI-native container image builder which produces
images in [OCI Image Layout](https://github.com/opencontainers/image-spec/blob/main/image-layout.md)
format.

    OCI Image Layout Specification

        The OCI Image Layout is the directory structure for OCI content-addressable blobs and location-addressable references (refs).
        This layout MAY be used in a variety of transport mechanisms: archive formats (e.g. tar, zip), shared filesystem environments (e.g. nfs), or networked file fetching (e.g. http, ftp, rsync).
