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

### Release

| Workflow | Description |
|---|---|
| `release-please.yml` | Standalone release-please — creates PRs and GitHub Releases from conventional commits |

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
| `go-ci.yml` | Lint (golangci-lint), test, build, vulncheck, security |
| `go-release-docker.yml` | Multi-arch Docker build + push to GHCR + cosign signing |
| `go-dev-build.yml` | Label-gated PR dev image (single-stage Docker) |

### .NET

| Workflow | Description |
|---|---|
| `dotnet-ci.yml` | Build, test, security (Trivy + CodeQL) |
| `dotnet-release-docker.yml` | Per-RID publish + multi-arch Docker + cosign signing |
| `dotnet-release-nuget.yml` | Build + test + pack + NuGet push + GitHub Release asset |
| `dotnet-dev-build.yml` | Label-gated PR dev image (per-RID, multi-arch manifest) |

## Architecture: Release Flow

Release-please is decoupled from build/publish workflows. Each consumer repo has a single `release.yml` with chained jobs:

```
push main → release-please → (if release created) → build + publish
```

This avoids the `GITHUB_TOKEN` limitation where releases created by workflows don't trigger `on: release` events.

### Phase 1: Release-Please

Runs on every push to main. Creates/updates release PRs and publishes GitHub Releases with semantic version tags.

### Phase 2: Build + Publish

Only runs when release-please creates a release. Receives the clean version (no `v` prefix) directly from release-please outputs.

### Multi-component repos

For repos with multiple release targets (e.g. app + rulesets), a mapping job extracts per-component outputs into clean names:

```yaml
releases:
  needs: release-please
  runs-on: ubuntu-latest
  outputs:
    app: ${{ steps.map.outputs.app }}
    app-version: ${{ steps.map.outputs.app-version }}
    rulesets: ${{ steps.map.outputs.rulesets }}
  steps:
    - id: map
      env:
        JSON: ${{ needs.release-please.outputs.json }}
      run: |
        echo "app=$(echo "$JSON" | jq -r '.".--release_created" // "false"')" >> "$GITHUB_OUTPUT"
        echo "app-version=$(echo "$JSON" | jq -r '.".--version" // ""')" >> "$GITHUB_OUTPUT"
        echo "rulesets=$(echo "$JSON" | jq -r '."data/community--release_created" // "false"')" >> "$GITHUB_OUTPUT"
```

Adding a new component: add lines to the mapping step + a new downstream job.

## Usage

### Prerequisites

Every consumer repo needs:
- `release-please-config.json` — release-please configuration
- `.release-please-manifest.json` — version tracking

### Go Docker app (e.g. bifrost)

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
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@v1
```

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
  workflow_dispatch:
concurrency:
  group: release
  cancel-in-progress: false
jobs:
  release-please:
    uses: st0o0/github-workflows/.github/workflows/release-please.yml@v1
    permissions:
      contents: write
      pull-requests: write
  docker:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    uses: st0o0/github-workflows/.github/workflows/go-release-docker.yml@v1
    with:
      version: ${{ needs.release-please.outputs.version }}
      image-name: ghcr.io/st0o0/bifrost
      image-description: WireGuard client for DDNS endpoint changes
    permissions:
      contents: read
      packages: write
      id-token: write
```

### .NET Docker app (e.g. njord)

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
  workflow_dispatch:
concurrency:
  group: release
  cancel-in-progress: false
jobs:
  release-please:
    uses: st0o0/github-workflows/.github/workflows/release-please.yml@v1
    permissions:
      contents: write
      pull-requests: write
  docker:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    uses: st0o0/github-workflows/.github/workflows/dotnet-release-docker.yml@v1
    with:
      version: ${{ needs.release-please.outputs.version }}
      image-name: ghcr.io/st0o0/njord
      image-description: Multi-model weather intelligence for Home Assistant
      solution-file: Njord.slnx
      publish-project: Njord/Njord.csproj
    permissions:
      contents: read
      packages: write
      id-token: write
```

### NuGet library (e.g. Flickr.Net)

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
  workflow_dispatch:
concurrency:
  group: release
  cancel-in-progress: false
jobs:
  release-please:
    uses: st0o0/github-workflows/.github/workflows/release-please.yml@v1
    permissions:
      contents: write
      pull-requests: write
  nuget:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    uses: st0o0/github-workflows/.github/workflows/dotnet-release-nuget.yml@v1
    with:
      version: ${{ needs.release-please.outputs.version }}
      solution-file: Flickr.Net.slnx
      package-name: Flickr.Net
    secrets:
      NUGET_USER: ${{ secrets.NUGET_USER }}
    permissions:
      contents: write
      checks: write
      id-token: write
```

### Security scan

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

## release-please.yml Outputs

| Output | Description | Use case |
|---|---|---|
| `release_created` | `true` if a release was created | Single-component repos |
| `version` | Clean semver (e.g. `0.2.0`) | Single-component repos |
| `tag_name` | Git tag (e.g. `v0.2.0`) | Single-component repos |
| `releases_created` | `true` if any release was created | Multi-component repos |
| `paths_released` | JSON array of released paths | Multi-component repos |
| `prs_created` | `true` if any PR was created/updated | Informational |
| `json` | All release-please outputs as JSON | Multi-component per-path access |
