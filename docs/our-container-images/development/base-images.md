# Base images

The k8s@spiti base images are meant to be used as an image to build
all other app container images on top of.

## Distributions

The following distributions are available as a base image:

- [Ubuntu][ubuntu]
- [Alpine][alpine]

The source code can be found [here].

## shim scripts

shim scripts have been added to the Ubuntu base image to perform startup tasks
before the app is launched.

!!! note
    The shim scripts must be sourced in the [entrypoint.sh] file in order
    to work.

### config.sh

The `config.sh` shim script is used to create the `/config` folder.

### umask.sh

The `umask.sh` shim script is used to set the setting of a mask that controls
how file permissions are set for newly created files.

### vpn.sh

The `vpn.sh` shim script is used to allow the container to wait for the VPN
connection before running the app.

## Tini

The base images use the [Tini][tini] init in order to create the smallest
image possible while still waiting for a child to exit all the while
reaping zombies and performing signal forwarding.

<!-- markdown-link-check-disable -->
[ubuntu]: https://github.com/k8s-at-spiti/container-images/pkgs/container/ubuntu
[alpine]: https://github.com/k8s-at-spiti/container-images/pkgs/container/alpine
<!-- markdown-link-check-enable -->
[here]: https://github.com/k8s-at-spiti/container-images/tree/main/base
[tini]: https://github.com/krallin/tini/
[entrypoint.sh]: ./creating-a-new-container-image.md#entrypointsh
