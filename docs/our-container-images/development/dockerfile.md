# Dockerfile

The `Dockerfile` file is a standard [Dockerfile][dockerfile] with a few
additions.

## Base images

Be sure to use one of the [base k8s@spiti images](./base-images.md) for the
[FROM][from] instruction.

### Ubuntu

```Docker
# hadolint ignore=DL3007
FROM ghcr.io/k8s-at-spiti/ubuntu:latest
```

### Alpine

```Docker
# hadolint ignore=DL3007
FROM ghcr.io/k8s-at-spiti/alpine:latest
```

## package_info

Also, be sure to add the `package_info` file to the image so that others can
find the source of the container image.

```Docker
RUN printf "UpdateMethod=docker\nPackageVersion=%s\nPackageAuthor=[Team k8s-at-spiti](https://github.com/k8s-at-spiti)" "${VERSION}" > /app/package_info
```

## UMASK and ca-certificates

Ensure that you change the permissions of the `/app` folder, set the `UMASK`
environmental variable, and update `ca-certificates`.

```Docker
RUN \
  chmod -R u=rwX,go=rX /app \
  && printf "umask %d" "${UMASK}" >> /etc/bash.bashrc \
  && update-ca-certificates
```

## COPY

When using the [COPY][copy] instruction, be sure to start from the root of the
repository so that the `github actions` can build the container image
properly.

```Docker
COPY ./apps/<app name>/entrypoint.sh /entrypoint.sh
```

## USER

Be sure set the USER to `kah`.

See [Configuration](../configuration.md) for details.

```Docker
USER kah
```

## shim scripts

If custom [shim scripts](./creating-a-new-container-image.md#shim-scripts)
are used, be sure to copy them to the container image.

```Docker
COPY ./apps/<app name>/shim/<app name>-preferences.sh /shim/<app name>-preferences.sh
```

## Linting

[Hadolint][hadolint] needs to be used to lint the Dockerfile before publishing.

Hadolint is also used as a check in the [Apps - Build, Test, Push github action][apps.yaml]

```shell
hadolint Dockerfile
```

## Best practices

Be sure to use [best practices] up  for writing Dockerfiles in order to reduce the
container image size.

[dockerfile]: https://docs.docker.com/engine/reference/builder/
[from]: https://docs.docker.com/engine/reference/builder/#from
[copy]: https://docs.docker.com/engine/reference/builder/#copy
[best practices]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
[hadolint]: https://github.com/hadolint/hadolint
[apps.yaml]: https://github.com/k8s-at-spiti/container-images/actions/workflows/apps.yaml
