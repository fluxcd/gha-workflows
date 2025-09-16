# gha-workflows

This repository contains reusable GitHub Workflows shared across the Flux organization.

## Workflows

- [backport](.github/workflows/backport.yaml):
  Automates the backporting of merged pull requests to release branches based on labels.
- [code-scan](.github/workflows/code-scan.yaml):
  Analyzes the code for security vulnerabilities using [CodeQL](https://codeql.github.com/) and
  validates the licenses of the dependencies using [FOSSA](https://fossa.com/).
- [labels-sync](.github/workflows/labels-sync.yaml):
  Synchronizes the [standard](https://github.com/fluxcd/community/blob/main/.github/standard-labels.yaml)
  and custom labels to the current repository.
