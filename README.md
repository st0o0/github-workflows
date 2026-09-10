# github-workflows

Reusable GitHub Actions workflows for st0o0 repositories.

## Workflows

| Workflow | Description | Language |
|---|---|---|
| `commitlint.yml` | Commit message validation | Any |
| `go-ci.yml` | Go lint, test, Docker build, e2e | Go |
| `go-docker-release.yml` | Release-please + multi-arch Docker + cosign | Go |
| `dotnet-ci.yml` | Build, test, format check, hadolint | .NET |
| `security-docker.yml` | Trivy image scan + SARIF upload | Any (Docker) |
| `security-dotnet.yml` | Trivy + NuGet vulnerability audit | .NET |
| `dev-build.yml` | Label-gated PR dev image build | Any (Docker) |
| `docs-vitepress.yml` | VitePress build + GitHub Pages deploy | Node.js |
| `codeql.yml` | GitHub CodeQL analysis | Any |

## Usage

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: st0o0/github-workflows/.github/workflows/go-ci.yml@main
    with:
      has-e2e: true
      smoke-test-command: docker run --rm app:ci --version

  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@main

  security:
    uses: st0o0/github-workflows/.github/workflows/security-docker.yml@main
```

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]

concurrency:
  group: release
  cancel-in-progress: false

jobs:
  release:
    uses: st0o0/github-workflows/.github/workflows/go-docker-release.yml@main
    with:
      image-name: ghcr.io/st0o0/my-app
      image-description: My application
    permissions:
      contents: write
      pull-requests: write
      packages: write
      id-token: write
```
