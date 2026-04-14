# publish-to-codeartifact

A GitHub Action that builds and publishes Python packages to AWS CodeArtifact using [uv](https://docs.astral.sh/uv/).

## Features

- Builds Python packages with `uv build` (works with any PEP 517 build backend)
- Publishes to AWS CodeArtifact via OIDC authentication
- Computes fallback versions for projects with a static `version` in `pyproject.toml`:
  - **Push to branch**: `X.Y.Z.devN` (dev version based on latest tag + run number)
  - **Pre-release**: `X.Y.ZrcN` (release candidate)
  - **Release**: version from the release tag
- Projects using dynamic versioning (e.g., `hatch-vcs`) determine their own version at build time
- Writes the actual published version and install instructions to the GitHub job summary

## Usage

```yaml
name: Publish to CodeArtifact

on:
  push:
    branches: ["**"]
  release:
    types: [released, prereleased]

permissions:
  id-token: write   # Required for AWS OIDC
  contents: read

jobs:
  publish:
    runs-on: ubuntu-24.04
    steps:
      - uses: CVector-Energy/publish-to-codeartifact@main
        with:
          package-name: my-package
          iam-role: arn:aws:iam::123456789012:role/MyDeploymentRole
          codeartifact-domain: my-domain
          codeartifact-domain-owner: "123456789012"
          codeartifact-repository: my-repo
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `package-name` | Yes | | Python package name (used in job summary) |
| `iam-role` | Yes | | ARN of the AWS IAM role to assume via OIDC |
| `aws-region` | No | `us-east-1` | AWS region |
| `codeartifact-domain` | Yes | | CodeArtifact domain name |
| `codeartifact-domain-owner` | Yes | | CodeArtifact domain owner AWS account ID |
| `codeartifact-repository` | Yes | | CodeArtifact repository name |
| `uv-index-name` | No | | Name of the uv index (from `[[tool.uv.index]]`) to authenticate for reading private dependencies during build |

## Prerequisites

The calling workflow must set `permissions: id-token: write` for AWS OIDC authentication to work.

The IAM role must have permissions to:
- `codeartifact:GetAuthorizationToken` on the domain
- `codeartifact:GetRepositoryEndpoint`, `codeartifact:PublishPackageVersion`, `codeartifact:PutPackageMetadata`, `codeartifact:ReadFromRepository` on the repository
- `sts:GetServiceBearerToken` for CodeArtifact service
