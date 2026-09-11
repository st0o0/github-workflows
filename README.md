# github-workflows

Reusable GitHub Actions workflows for st0o0 repositories.

## Versioning

This repo uses [release-please](https://github.com/googleapis/release-please) with semantic versioning.
Consumers should pin to a **major version tag**:

```yaml
uses: st0o0/github-workflows/.github/workflows/dotnet-ci.yml@v1
```

The `@v1` tag tracks the latest `v1.x.x` release. Breaking changes will bump to `@v2`.

## Workflows

### Shared

| Workflow | Description |
|---|---|
| `commitlint.yml` | Commit message validation via wagoid/commitlint |
| `codeql.yml` | GitHub CodeQL analysis (any language) |
| `security.yml` | Trivy Docker image scan + SARIF upload to Security tab |
| `docs.yml` | VitePress build + GitHub Pages deploy |

### Go

| Workflow | Description |
|---|---|
| `go-ci.yml` | Lint (golangci-lint), test, Docker build, optional e2e |
| `go-release.yml` | Release-please + multi-arch Docker build + cosign signing |
| `go-dev-build.yml` | Label-gated PR dev image (single-stage Docker) |

### .NET

| Workflow | Description |
|---|---|
| `dotnet-ci.yml` | Build, test, format check, hadolint |
| `dotnet-release-docker.yml` | Release-please + per-RID publish + multi-arch Docker + cosign |
| `dotnet-release-nuget.yml` | GitVersion + test + pack + NuGet push + GitHub release |
| `dotnet-dev-build.yml` | Label-gated PR dev image (per-RID, multi-arch manifest) |

## Usage

### Go app (e.g. bifrost)

```yaml
# .github/workflows/ci.yml
name: CI
on: [pull_request]
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
jobs:
  ci:
    uses: st0o0/github-workflows/.github/workflows/go-ci.yml@v1
    with:
      has-e2e: true
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@v1
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
    uses: st0o0/github-workflows/.github/workflows/go-release.yml@v1
    with:
      image-name: ghcr.io/st0o0/bifrost
      image-description: WireGuard client for DDNS endpoint changes
    permissions:
      contents: write
      pull-requests: write
      packages: write
      id-token: write
```

```yaml
# .github/workflows/security.yml
name: Security
on:
  pull_request:
    paths: ['Dockerfile', 'go.mod', 'go.sum']
  schedule:
    - cron: '0 6 * * 1'
  workflow_dispatch:
concurrency:
  group: security-${{ github.ref }}
  cancel-in-progress: true
jobs:
  scan:
    uses: st0o0/github-workflows/.github/workflows/security.yml@v1
    with:
      docker-image-name: bifrost
    permissions:
      contents: read
      security-events: write
```

### .NET Docker app (e.g. njord)

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
    paths-ignore: ['docs/**']
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
jobs:
  ci:
    uses: st0o0/github-workflows/.github/workflows/dotnet-ci.yml@v1
    with:
      solution-file: Njord.slnx
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@v1
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
    uses: st0o0/github-workflows/.github/workflows/dotnet-release-docker.yml@v1
    with:
      image-name: ghcr.io/st0o0/njord
      image-description: Multi-model weather intelligence for Home Assistant
      solution-file: Njord.slnx
      publish-project: Njord/Njord.csproj
    permissions:
      contents: write
      pull-requests: write
      packages: write
      id-token: write
```

### NuGet library (e.g. Flickr.Net)

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
    branches: [main]
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
jobs:
  ci:
    uses: st0o0/github-workflows/.github/workflows/dotnet-ci.yml@v1
    with:
      solution-file: Flickr.Net.sln
      has-dockerfile: false
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@v1
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
    uses: st0o0/github-workflows/.github/workflows/dotnet-release-nuget.yml@v1
    with:
      solution-file: Flickr.Net.sln
      package-name: Flickr.Net
    secrets:
      NUGET_API_KEY: ${{ secrets.NUGET_API_KEY }}
    permissions:
      contents: write
      pull-requests: write
      checks: write
```
