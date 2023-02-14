# CI build of Backstage

[1]: https://quay.io/repository/janus-idp/redhat-backstage-build
[2]: https://github.com/janus-idp/redhat-backstage-build/pkgs/container/redhat-backstage-build

[![Build Status](https://github.com/janus-idp/redhat-backstage-build/workflows/Build/badge.svg?branch=main)](https://github.com/janus-idp/redhat-backstage-build/actions?workflow=Build)
[![Quay.io registry](https://img.shields.io/badge/Quay.io-janus--idp/redhat--backstage--build:latest-white?logoColor=white&labelColor=grey&logo=data:image/svg%2bxml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSIzOS43IiB2aWV3Qm94PSIwIDAgMzkuNyAzOS43Ij48ZyB0cmFuc2Zvcm09Im1hdHJpeCguMzk0NTggMCAwIC4zOTQ1OCAxLjA4MiAuMTA1KSI+PGNpcmNsZSBjeD0iNTUuMDY5IiBjeT0iNTAiIHI9IjUwIiBmaWxsPSIjRDcxRTAwIi8+PHBhdGggZmlsbD0iI0MyMUEwMCIgZD0iTTkwLjQyOSAxNC42NDFjMTkuNTI5IDE5LjUyOSAxOS41MjkgNTEuMTkgMCA3MC43MTgtMTkuNTI5IDE5LjUzLTUxLjE5MiAxOS41My03MC43MiAwbDcwLjcyLTcwLjcxOHoiLz48cGF0aCBmaWxsPSIjRkZGIiBkPSJtNTkuNjA4IDQ5Ljk5IDE1LjA2IDMxLjg2OWgtMTIuODNMNDYuNzg5IDQ5Ljk5bDE1LjA0OS0zMS44NWgxMi44M3oiLz48cGF0aCBmaWxsPSIjQjdCN0I3IiBkPSJtNzQuNjY4IDgxLjg1OSAxNS4wNS0zMS44NjktMTUuMDUtMzEuODUtNi40MSAxMy41NiA4LjY0IDE4LjI5LTguNjQgMTguMzAxeiIvPjxwYXRoIGZpbGw9IiNGRkYiIGQ9Im0zMy4yMzkgNDkuOTkgMTUuMDYgMzEuODY5aC0xMi44M0wyMC40MTkgNDkuOTlsMTUuMDUtMzEuODVoMTIuODN6Ii8+PHBhdGggZmlsbD0iI0I3QjdCNyIgZD0ibTQ4LjY1OSA0Ni4wNCA2LjQxLTEzLjU3LTYuNzctMTQuMzMtNi40MiAxMy41N3pNNDEuODc5IDY4LjI5MWw2LjQyIDEzLjU2OCA2Ljc3LTE0LjMzLTYuNDEtMTMuNTY4eiIvPjwvZz48L3N2Zz4K)][1]
[![ghcr.io registry](https://img.shields.io/badge/ghcr.io-janus--idp/redhat--backstage--build:latest-white?logoColor=white&labelColor=grey&logo=github)][2]

## Usage

This image is available in [Quay.io][1] and [ghcr.io][2] registries:

```bash
docker pull quay.io/janus-idp/redhat-backstage-build:latest
# OR
docker pull ghcr.io/janus-idp/redhat-backstage-build:latest
```

## Minimal Backstage instance boilerplate

To get started with your own Backstage instance, follow the [Getting Started](https://backstage.io/docs/getting-started/) instructions.

This repository uses a vanilla Backstage app created via following command:

```bash
echo backstage | npx @backstage/create-app
```

## Docker/Podman

This repository provides a `Dockerfile` allowing you to build Backstage as a container based on [UBI9/nodejs-18 image](https://catalog.redhat.com/software/containers/ubi9/nodejs-18/62e8e7ed22d1d3c2dfe2ca01):

```bash
BACKSTAGE_LOCATION=<path_to_backstage>
IMAGE=<target_image_tag>

cp ./Dockerfile ./.dockerignore $BACKSTAGE_LOCATION
docker build -t $IMAGE $BACKSTAGE_LOCATION
```

## Source-to-Image

The Backstage image can be also created via [Source-to-Image](https://github.com/openshift/source-to-image).

Please install the `s2i` cli tool to build locally.

### Using docker daemon

```bash
BACKSTAGE_LOCATION=<path_to_backstage>
IMAGE=<target_image_tag>

s2i build \
  $BACKSTAGE_LOCATION \
  --scripts-url https://raw.githubusercontent.com/janus-idp/redhat-backstage-build/main/.s2i/bin/ \
  registry.access.redhat.com/ubi9/nodejs-18:latest \
  $IMAGE
```

### Using rootless podman

```bash
BACKSTAGE_LOCATION=<path_to_backstage>
IMAGE=<target_image_tag>

tmp_dir=$(mktemp -d)
s2i build \
  $BACKSTAGE_LOCATION \
  --scripts-url https://raw.githubusercontent.com/janus-idp/redhat-backstage-build/main/.s2i/bin/ \
  registry.access.redhat.com/ubi9/nodejs-18:latest \
  --as-dockerfile ${tmp_dir}/Containerfile

cd $tmp_dir
podman build -t $IMAGE .
rm -rf $tmp_dir
```

### Building in OpenShift cluster

Remote builds are also possible in OpenShift. S2I can be used remotely against any Backstage repository without a local Dockerfile.

```bash
GIT_REPOSITORY=<your_backstage_instance_git_repository>
GIT_REF=<branch_or_commit>

cat << EOF | oc apply -f -
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: backstage
spec:
  output:
    to:
      kind: ImageStreamTag
      name: backstage
  source:
    type: Git
    git:
      uri: $GIT_REPOSITORY
      ref:  $GIT_REF
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        name: "nodejs:latest"
        namespace: openshift
      scripts: https://raw.githubusercontent.com/redhat-developer/redhat-backstage-build/main/.s2i/bin/
  triggers:
  - type: ConfigChange
EOF
```
