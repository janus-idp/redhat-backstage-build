# CI build of Backstage

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
