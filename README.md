# gha-workflows

[![license](https://img.shields.io/github/license/fluxcd/gha-workflows.svg)](https://github.com/fluxcd/gha-workflows/blob/main/LICENSE)
[![release](https://img.shields.io/github/release/fluxcd/gha-workflows/all.svg)](https://github.com/fluxcd/gha-workflows/releases)

This repository contains reusable GitHub Workflows shared across the Flux controller repositories.

## Workflows

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
      contents: write
      pull-requests: write
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
      contents: read
      security-events: write
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
    branches:
      - main
    paths:
      - .github/labels.yaml
jobs:
  sync-labels:
    permissions:
      issues: write
      contents: read
    uses: fluxcd/gha-workflows/.github/workflows/labels-sync.yaml@vX.Y.Z
    with:
      labels-file: .github/labels.yaml
    secrets:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```

3rd-party actions used:

- [EndBug/label-sync](https://github.com/EndBug/label-sync)

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
