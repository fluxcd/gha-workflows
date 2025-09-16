# gha-workflows

[![license](https://img.shields.io/github/license/fluxcd/gha-workflows.svg)](https://github.com/fluxcd/gha-workflows/blob/main/LICENSE)
[![release](https://img.shields.io/github/release/fluxcd/gha-workflows/all.svg)](https://github.com/fluxcd/gha-workflows/releases)

This repository contains reusable GitHub Workflows shared across the Flux controller repositories.

## Workflows

- [backport](.github/workflows/backport.yaml):
  Automates the backporting of merged pull requests to release branches based on labels.
- [code-scan](.github/workflows/code-scan.yaml):
  Analyzes the code for security vulnerabilities using [CodeQL](https://codeql.github.com/) and
  validates the licenses of the dependencies using [FOSSA](https://fossa.com/).
- [labels-sync](.github/workflows/labels-sync.yaml):
  Synchronizes the [standard](https://github.com/fluxcd/community/blob/main/.github/standard-labels.yaml)
  and custom labels to the current repository.

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
