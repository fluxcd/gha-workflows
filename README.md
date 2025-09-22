# gha-workflows

[![license](https://img.shields.io/github/license/fluxcd/gha-workflows.svg)](https://github.com/fluxcd/gha-workflows/blob/main/LICENSE)
[![release](https://img.shields.io/github/release/fluxcd/gha-workflows/all.svg)](https://github.com/fluxcd/gha-workflows/releases)

This repository contains reusable GitHub Workflows and Composite Actions shared across the Flux controller repositories.

## Workflows

### Release Flux controller

The [controller-release](.github/workflows/controller-release.yaml) workflow automates the release of
Flux controllers by performing the following steps:

- Builds multi-arch images for `linux/amd64`, `linux/arm64` and `linux/arm/v7` with Docker.
- Generates SBOMs for each architecture with Syft.
- Pushes the images to `ghcr.io/fluxcd` and `docker.io/fluxcd`.
- Signs the images with Cosign and GitHub OIDC.
- Creates a GitHub Release with GoReleaser.
- Outputs metadata for SLSA attestations.

Example usage:

```yaml
name: release
on:
  push:
    tags: [ 'v*' ]
  workflow_dispatch:
    inputs:
      tag:
        description: 'image tag prefix'
        default: 'rc'
        required: false
jobs:
  release:
    permissions:
      contents: write # for creating the GitHub release.
      id-token: write # for creating OIDC tokens for signing.
      packages: write # for pushing and signing container images.
    uses: fluxcd/gha-workflows/.github/workflows/controller-release.yaml@vX.Y.Z
    with:
      controller: ${{ github.event.repository.name }}
      release-candidate-prefix: ${{ github.event.inputs.tag }}
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
```

3rd-party actions used:

- [docker/setup-qemu-action](https://github.com/docker/setup-qemu-action)
- [docker/setup-buildx-action](https://github.com/docker/setup-buildx-action)
- [docker/login-action](https://github.com/docker/login-action)
- [docker/metadata-action](https://github.com/docker/metadata-action)
- [docker/build-push-action](https://github.com/docker/build-push-action)
- [sigstore/cosign-installer](https://github.com/sigstore/cosign-installer)
- [anchore/sbom-action](https://github.com/anchore/sbom-action)
- [goreleaser/goreleaser-action](https://github.com/goreleaser/goreleaser-action)

Outputs:

- `release-digests`: Release artifacts digests compatible with SLSA
- `image-name`: Published container image name (without the registry)
- `image-digest`: Published container image digest

### Backport to Release Branches

The [backport](.github/workflows/backport.yaml) workflow automates the backporting of merged pull
requests to release branches based on labels in the format `backport:release/semver`
(e.g. `backport:release/v2.0.x`).

Example usage:

```yaml
name: backport
on:
  pull_request_target:
    types: [closed, labeled]
jobs:
  backport:
    permissions:
      contents: write # for reading and creating branches.
      pull-requests: write # for creating pull requests against release branches.
    uses: fluxcd/gha-workflows/.github/workflows/backport.yaml@vX.Y.Z
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

3rd-party actions used:

- [korthout/backport-action](https://github.com/korthout/backport-action)

### Code Scanning and License Validation

The [code-scan](.github/workflows/code-scan.yaml) workflow analyzes the code for security vulnerabilities
using [CodeQL](https://codeql.github.com/) and validates the licenses of the dependencies
using [FOSSA](https://fossa.com/).

Example usage:

```yaml
name: code-scan
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
jobs:
  analyze:
    permissions:
      contents: read # for reading the repository code.
      security-events: write # for uploading the CodeQL analysis results.
    uses: fluxcd/gha-workflows/.github/workflows/code-scan.yaml@vX.Y.Z
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      fossa-token: ${{ secrets.FOSSA_TOKEN }}
```

The CodeQL analysis uploads the results to GitHub Code Scanning Alerts,
and the FOSSA analysis uploads the results to the FOSSA dashboard.

3rd-party actions used:

- [fossas/fossa-action](https://github.com/fossas/fossa-action)

### Sync Repository Labels

The [labels-sync](.github/workflows/labels-sync.yaml) workflow synchronizes the
[standard](https://github.com/fluxcd/community/blob/main/.github/standard-labels.yaml)
and custom labels to the current repository.

Example usage:

```yaml
name: sync-labels
on:
  workflow_dispatch:
  push:
    branches: [ main ]
    paths:
      - .github/labels.yaml
jobs:
  sync-labels:
    permissions:
      contents: read # for reading the labels file.
      issues: write # for creating and updating labels.
    uses: fluxcd/gha-workflows/.github/workflows/labels-sync.yaml@vX.Y.Z
    with:
      labels-file: .github/labels.yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

3rd-party actions used:

- [EndBug/label-sync](https://github.com/EndBug/label-sync)

## Composite Actions

### Setup Kubernetes

The [setup-kubernetes](.github/actions/setup-kubernetes/action.yaml) composite action configures
the GitHub runner to build and test Flux controllers with Kubernetes Kind clusters.

Example usage:

```yaml
name: e2e
on:
  pull_request:
  push:
    branches: [ main ]
jobs:
  kind:
    runs-on: ubuntu-latest
    permissions:
      contents: read # for reading the repository code.
    steps:
      - name: Test suite setup
        uses: fluxcd/gha-workflows/.github/actions/setup-kubernetes@vX.Y.Z
        with:
          go-version: 1.25.x
          kind-version: v0.30.0
      - name: Run tests
        run: make test
```

3rd-party actions used:

- [docker/setup-qemu-action](https://github.com/docker/setup-qemu-action)
- [docker/setup-buildx-action](https://github.com/docker/setup-buildx-action)
- [helm/kind-action](https://github.com/helm/kind-action)

## Contributing

- The workflows must be placed in the `.github/workflows` directory and
  the filenames must be in the format `<my-workflow>.yaml`. The filename must match the workflow name.
- All workflows requiring repository access must expose a `github-token` secret input.
- The repo permissions must be set in the workflow file, and not rely on the default permissions.
- All the actions used in workflows must be pinned to a commit SHA (Dependabot is configured to keep them up to date).
- The usage of third-party actions should be limited to well-known actions with a good security track record.
- Changed to workflows should be tested in a fork before opening a pull request,
  especially those that trigger on **push tag** events.

## Releasing new versions

To release a new version of the workflows, push a **signed** git tag with the version number (e.g. `v1.2.3`).

Dependabot is configured in the Flux controllers repositories to keep the workflows up
to date with the latest released version.
