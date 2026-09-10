# github-workflows

Reusable GitHub Actions workflows for st0o0 repositories.

## Workflows

### General
| Workflow | Description |
|---|---|
| `commitlint.yml` | Commit message validation via wagoid/commitlint |
| `codeql.yml` | GitHub CodeQL analysis (any language) |

### Go
| Workflow | Description |
|---|---|
| `go-ci.yml` | Lint (golangci-lint), test, Docker build, optional e2e |
| `go-docker-release.yml` | Release-please + multi-arch Docker build + cosign signing |

### .NET
| Workflow | Description |
|---|---|
| `dotnet-ci.yml` | Build, test, format check, hadolint |
| `dotnet-nuget-release.yml` | GitVersion + test + pack + NuGet push + GitHub release |
| `dotnet-docker-release.yml` | Release-please + per-RID publish + multi-arch Docker + cosign |
| `dotnet-dev-build.yml` | Label-gated PR dev image (per-RID, multi-arch manifest) |

### Docker & Security
| Workflow | Description |
|---|---|
| `dev-build.yml` | Label-gated PR dev image (single-stage Docker, e.g. Go) |
| `security-docker.yml` | Trivy image scan + SARIF upload |
| `security-dotnet.yml` | Trivy image scan + NuGet vulnerability audit |

### Docs
| Workflow | Description |
|---|---|
| `docs-vitepress.yml` | VitePress build + GitHub Pages deploy |

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
    uses: st0o0/github-workflows/.github/workflows/go-ci.yml@main
    with:
      has-e2e: true
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@main
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
      image-name: ghcr.io/st0o0/bifrost
      image-description: WireGuard client for DDNS endpoint changes
    permissions:
      contents: write
      pull-requests: write
      packages: write
      id-token: write
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
    uses: st0o0/github-workflows/.github/workflows/dotnet-ci.yml@main
    with:
      solution-file: Njord.slnx
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@main
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
    uses: st0o0/github-workflows/.github/workflows/dotnet-docker-release.yml@main
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
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  build:
    uses: st0o0/github-workflows/.github/workflows/dotnet-nuget-release.yml@main
    with:
      solution-file: Flickr.Net.sln
      package-name: Flickr.Net
    secrets:
      NUGET_API_KEY: ${{ secrets.NUGET_API_KEY }}
    permissions:
      contents: write
      deployments: write
      checks: write
      pull-requests: write
```
