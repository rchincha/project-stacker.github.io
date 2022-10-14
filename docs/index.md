# Stacker - OCI-native Container Image Builder

Stacker is a tool for building OCI images natively via a declarative yaml format.

## Features

* Single binary
* Unprivileged builds
* Hermetically sealed builds using LXC containers
* [GitHub action](https://github.com/marketplace/actions/stacker-build-and-push-action)

### Installation

Stacker has various [build](installation.md) and [runtime](runtime.md)
dependencies.

### Usage

See the [tutorial](tutorial.md) for a short introduction to how to use stacker.

See the [`stacker.yaml` specification](stacker_yaml.md) for full details on
the `stacker.yaml` specification.

Additionally, there are some [tips and tricks](tricks.md) for common usage.

### Hacking

See the [hacking](hacking.md) guide for tips on hacking/debugging stacker.

### TODO / Roadmap

* Upstream something to containers/image that allows for automatic detection
  of compression
* Design/implement OCIv2 drafts + final spec when it comes out

### Conference Talks

* An Operator Centric Way to Update Application Containers FOSDEM 2019
    * [video](https://archive.fosdem.org/2019/schedule/event/containers_atomfs/)
    * [slides](talks/FOSDEM_2019.pdf)
* Building OCI Images without Privilege OSS EU 2018
    * [slides](talks/OSS_EU_2018.pdf)
* Building OCI Images without Privilege OSS NA 2018
    * [slides](talks/OSS_NA_2018.pdf)

(Note that despite the similarity in name of the 2018 talks, the content is
mostly disjoint; I need to be more creative with naming.)

