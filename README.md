# Publish Image to Various Registries

[![Dependabot Updates](https://github.com/wesley-dean/publish_container/actions/workflows/dependabot/dependabot-updates/badge.svg)](https://github.com/wesley-dean/publish_container/actions/workflows/dependabot/dependabot-updates)
[![MegaLinter](https://github.com/wesley-dean/publish_container/actions/workflows/megalinter.yml/badge.svg)](https://github.com/wesley-dean/publish_container/actions/workflows/megalinter.yml)
[![Scorecard supply-chain security](https://github.com/wesley-dean/publish_container/actions/workflows/scorecard.yml/badge.svg)](https://github.com/wesley-dean/publish_container/actions/workflows/scorecard.yml)

## Description

This project is a GitHub Action that builds a Docker-compatible image on
various platforms, tags the image, and then publishes a Docker image to various
registries. The action can be triggered by a push to a branch, a pull request,
a release, a schedule, or a manual trigger.

### Features

#### Image Labels

The action will add OCI metadata to the image, including the following:

- title
- description
- url
- source
- version
- created
- revision
- license

#### Image Descriptions

The action will also update the description of the image on DockerHub, Quay,
and Harbor using the README.md file in the repository.

#### Image Attestations

The action will also generate Software Bill of Materials (SBOM) files in
SPDX format and use them to generate attestations for each image that
is built.

#### Composite Action Components

The action is a composite action that uses several other actions to build and
publish images. The action uses the following actions:

- [docker/build-push-action](https://github.com/docker/build-push-action)
- [docker/login-action](https://github.com/docker/login-action)
- [docker/setup-qemu-action](https://github.com/docker/setup-qemu-action)
- [docker/setup-buildx-action](https://github.com/docker/setup-buildx-action)
- [docker/metadata-action](https://github.com/docker/metadata-action)
- [docker/build-push-action](https://github.com/docker/build-push-action)
- [anchore/sbom-action](https://github.com/anchore/sbom-action)
- [actions/attest-sbom](https://github.com/actions/attest-sbom)
- [christian-korneck/update-container-description-action](https://github.com/christian-korneck/update-container-description-action)

#### Image Platforms

By default, the action builds images for the following platforms:

- linux/amd64
- linux/arm64
- linux/arm/v6
- linux/arm/v7

The action can be configured to build images for other platforms by setting the
`platforms` input.  The input should be a comma-separated list of platforms.

The `arm` images are built using QEMU emulation so that they can run on
ARM-based devices such as the Raspberry Pi.

#### Supported Registries

The action can be configured to publish images to the following registries:

- DockerHub (docker.io)
- GitHub Container Registry (ghcr.io)
- Quay (quay.io)
- Harbor (goharbor.io)
- Amazon Elastic Container Registry (ECR)
- custom registry

The action can be configured to publish images to other registries by setting
`_username` and `_token` inputs for the respective registries:

- **DockerHub**: `dockerhub_username` and `dockerhub_token`
- **GitHub Container Registry**: `ghcr_username` and `ghcr_token`
- **Quay**: `quay_username` and `quay_token`
- **Harbor**: `harbor_username` and `harbor_token`
- **Amazon Elastic Container Registry (ECR)**: `ecr_access_key_id` and `ecr_secret_access_key`
- **Custom Registry**: `custom_registry_username` and `custom_registry_token`

#### Image Tags

The action adds various tags to the image, including the following:

- every build
  - `edge`
  - `{short sha}`: (short SHA hash of the commit)
  - `{long sha}` (long SHA hash of the commit)
- when the action is triggered by a push to a branch
  - `latest`
  - `{major}.{minor}.{patch}`
  - `{major}.{minor}`
  - `{major}`

## Usage

To use the action, add the following to your GitHub Actions workflow:

```yaml
  - name: Publish Image to Various Registries
    uses: wesdean/publish-image-to-various-registries@v1
    with:
      platforms: 'linux/amd64,linux/arm64,linux/arm/v6,linux/arm/v7'
      dockerfile: 'Dockerfile'
      context: '.'

      # to push to DockerHub, use:
      dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
      dockerhub_token: ${{ secrets.DOCKERHUB_TOKEN }}

      # to push to GitHub Container Registry (GHCR), use:
      ghcr_username: ${{ secrets.GHCR_USERNAME }}

      # to push to Quay, use:
      quay_username: ${{ secrets.QUAY_USERNAME }}
      quay_token: ${{ secrets.QUAY_TOKEN }}

      # to push to Harbor, use:
      harbor_username: ${{ secrets.HARBOR_USERNAME }}
      harbor_token: ${{ secrets.HARBOR_TOKEN }}

      # to push to Amazon Elastic Container Registry (ECR), use:
      ecr_access_key_id: ${{ secrets.ECR_ACCESS_KEY_ID }}
      ecr_secret_access_key: ${{ secrets.ECR_SECRET_ACCESS_KEY }}
      ecr_region: 'us-east-1'

      # to push to a custom registry, use:
      custom_registry_username: ${{ secrets.CUSTOM_REGISTRY_USERNAME }}
      custom_registry_token: ${{ secrets.CUSTOM_REGISTRY_TOKEN }}
      custom_registry: 'registry.example.com'
```

All of the registry-specific inputs are optional.  That said, if none of the
registry inputs are set, the action will build the image and tag it, but it
will not publish the image.  It'll just eat your billing minutes.  I
recommend against that.

### Inputs

The following inputs are available:

- **basic**
  - context: The build context for the Docker build. Default: `.`.
  - dockerfile: The path to the Dockerfile. Default: `Dockerfile`.
  - platforms: The platforms to build images for. Default: `linux/amd64,linux/arm64,linux/arm/v6,linux/arm/v7`.
  - repository_name: The name of the repository.  Consider using
    `${{ github.repository }}` to get the repository name.  **REQUIRED**
  - github_ref: The GitHub ref that triggered the action.  Consider using
    `${{ github.ref }}` to get the ref that triggered the action. **REQUIRED**
- **DockerHub**
  - dockerhub_image: The name of the image on DockerHub. Default:
    `{ dockerhub username }/{ repository name }`.
  - dockerhub_username: The username for DockerHub.  Required to use DockerHub.
  - dockerhub_token: The token for DockerHub.  Required to use DockerHub.
  - dockerhub_registry: The URL for DockerHub. Default: `docker.io`.
- **GitHub Container Registry (GHCR)**
  - ghcr_image: The name of the image on GHCR. Default:
    `{ ghcr username }/{ repository name }`.
  - ghcr_username: The username for GHCR.  Required to use GHCR.
  - ghcr_token: The token for GHCR.  Required to use GHCR.
  - ghcr_registry: The URL for GHCR. Default: `ghcr.io`.
- **Quay**
  - quay_image: The name of the image on Quay. Default:
    `{ quay username }/{ repository name }`.
  - quay_username: The username for Quay.  Required to use Quay.
  - quay_token: The token for Quay.  Required to use Quay.
  - quay_registry: The URL for Quay. Default: `quay.io`.
- **Harbor**
  - harbor_image: The name of the image on Harbor. Default:
    `{ harbor username }/{ repository name }`.
  - harbor_username: The username for Harbor.  Required to use Harbor.
  - harbor_token: The token for Harbor.  Required to use Harbor.
  - harbor_registry: The registry for Harbor. Default: `goharbor.io`.
- **Amazon Elastic Container Registry (ECR)**:
  - ecr_image: The name of the image on ECR. Default: `{ repository name }`.
  - ecr_access_key_id: The access key ID for ECR.  Required to use ECR.
  - ecr_secret_access_key: The secret access key for ECR.  Required to use ECR.
  - ecr_region: The region for ECR.  Default: `us-east-1`.
  - ecr_registry: The URL to use with ECR.  Default:
    `{ ecr account id }.dkr.ecr.{ region }.amazonaws.com`.
- **Custom Registry**
  - custom_registry_image: The name of the image on the custom registry.
    Default: `{ repository name }`.
  - custom_registry_username: The username for the custom registry.
    Required to use a custom registry.
  - custom_registry_token: The token for the custom registry.  Required to use
    a custom registry.
  - custom_registry: The URL to the custom registry.

### Outputs

The following outputs are available:
- **DockerHub**
  - dockerhub: `true` if the image was published to DockerHub, `false` otherwise.
  - dockerhub_image: The name of the image on DockerHub.
- **GitHub Container Registry (GHCR)**
  - ghcr: `true` if the image was published to GHCR, `false` otherwise.
  - ghcr_image: The name of the image on GHCR.
- **Quay**
  - quay: `true` if the image was published to Quay, `false` otherwise.
  - quay_image: The name of the image on Quay.
- **Harbor**
  - harbor: `true` if the image was published to Harbor, `false` otherwise.
  - harbor_image: The name of the image on Harbor.
- **Amazon Elastic Container Registry (ECR)**
  - ecr: `true` if the image was published to ECR, `false` otherwise.
  - ecr_image: The name of the image on ECR.
  - ecr_image_url: The URL of the image on ECR.
- **Custom Registry**
  - custom_registry: `true` if the image was published to a custom registry,
    `false` otherwise.
  - custom_image: The name of the image on the custom registry.

## Examples

### Build and Publish Image to DockerHub

This is a minimal example that builds an image and publishes it to DockerHub.

The action is triggered by a push to the `main` branch or a tag that starts with
`v` and a version number with a major release level greater than 0 (i.e., it
would run for `v1.0.0`, but not for `v0.1.0`).  The action can also be triggered
manually.

```yaml
---
name: Build an Awesome Image

# yamllint disable-line rule:truthy
on:
  push:
    branches:
      - "main"
    tags:
      - "v[1-9][0-9]*.*"
  workflow_dispatch:

permissions: read-all

jobs:
  publish_image:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout
        uses: actions/checkout@v4.2.2
        with:
          depth: 0
          tags: true
      - name: Build and Publish Image
        uses: wesley-dean/publish_container@v1
        with:
          dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
          dockerhub_token: ${{ secrets.DOCKERHUB_TOKEN }}
          github_ref: ${{ github.ref }}
          repository_name: ${{ github.repository }}
```

### Build and Publish Image to DockerHub and GHCR

This example pushes to both DockerHub and GHCR.  The image is given different
names on each registry.

```yaml
---
name: Build an Awesome Image

# yamllint disable-line rule:truthy
on:
  push:
    branches:
      - "main"
    tags:
      - "v[1-9][0-9]*.*"
  workflow_dispatch:

permissions: read-all

jobs:
  publish_image:
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout
        uses: actions/checkout@v4.2.2
        with:
          depth: 0
          tags: true
      - name: Build and Publish Image
        uses: wesley-dean/publish_container@v1
        with:
          github_ref: ${{ github.ref }}
          repository_name: ${{ github.repository }}

          dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
          dockerhub_token: ${{ secrets.DOCKERHUB_TOKEN }}
          dockerhub_image: "myname/awesomeimage"

          ghcr_username: ${{ secrets.GHCR_USERNAME }}
          ghcr_token: ${{ secrets.GHCR_TOKEN }}
          ghcr_image: "my-name/awesome-image"
```

## License

This project is licensed under the Creative Commons License 1.0 Universal
License - see the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome. Please read the [CONTRIBUTING.md](CONTRIBUTING.md)
file for details.

### Code of Conduct

Please read the [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) file for details on
the code of conduct.  Long story short, be nice to each other and treat each
other with respect, compassion, and empathy, especially when you disagree.

## Authors

- Wes Dean
