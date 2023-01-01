# Features

The key features of `stacker` are:

## Single Binary

One statically built binary for simplified download and installation with no
additional dependencies or services.

## Unprivileged

Build using user namespaces and avoid privileges on host.

## Repeatable

Hermetically sealed environment for image builds using LXC containers so that
builds are repeatable without side-effects.

## Incremental

Build only the images necessary and rebuild only if any input changed.

## Open source

Released as an open source project under Apache2 License.

## GitHub Action

Integrated with GitHub as a build-and-push
[action](https://github.com/marketplace/actions/stacker-build-and-push-action).
