# github-workflows

[![Release](https://img.shields.io/github/v/release/st0o0/github-workflows?sort=semver&style=flat-square)](https://github.com/st0o0/github-workflows/releases)

Reusable GitHub Actions workflows for st0o0 repositories.

## Versioning

This repo uses [release-please](https://github.com/googleapis/release-please) with semantic versioning.
Consumers should pin to an **exact release tag** and let Renovate keep it current:

<!-- x-release-please-start-version -->

```yaml
uses: st0o0/github-workflows/.github/workflows/dotnet-ci.yml@v1.0.0
```

With the shared [`st0o0/renovate-config`](https://github.com/st0o0/renovate-config) preset, Renovate pins the tag to its commit SHA and opens a PR for every new release. Minor and patch releases are automerged once CI passes; major releases (breaking changes) need a manual merge.

A moving `v1` tag that tracks the latest `v1.x.x` release is also published for repos without Renovate.

Dependency updates in this repo are committed as `fix(deps):` so that they produce a release and reach consumers.

## Workflows

### Release

| Workflow | Description |
|---|---|
| `release-please.yml` | Standalone release-please, creates PRs and GitHub Releases from conventional commits |

### Shared

| Workflow | Description |
|---|---|
| `commitlint.yml` | Commit message validation via wagoid/commitlint |
| `security.yml` | Trivy scan of a locally built Docker image or of the filesystem + SARIF upload to Security tab |
| `docs.yml` | VitePress build + GitHub Pages deploy |
| `labeler.yml` | Path-based PR labeling (actions/labeler) |
| `labels.yml` | Label definition sync (EndBug/label-sync) |

### Docker

| Workflow | Description |
|---|---|
| `docker-release.yml` | Multi-arch Docker build + push to GHCR with SBOM and provenance + cosign signing |
| `docker-dev.yml` | PR dev image pushed to GHCR as `pr-<number>`, cosign-signed, pull command posted as PR comment |

### Go

| Workflow | Description |
|---|---|
| `go-ci.yml` | Tests (race), go.mod tidy check, golangci-lint, hadolint, Docker build + optional smoke test |

### .NET

| Workflow | Description |
|---|---|
| `dotnet-ci.yml` | Build, test with coverage report, hadolint, format check, optional Slopwatch |
| `dotnet-build-nuget.yml` | Build + test + pack, uploads the `.nupkg` files as `nuget-packages` artifact |

`dotnet-ci.yml` and `dotnet-build-nuget.yml` run the optional composite actions `.github/actions/ci-setup` and `.github/actions/ci-post-build` from the **consumer repo** if they exist.

### Frontend

| Workflow | Description |
|---|---|
| `frontend-ci.yml` | pnpm install + lint, typecheck and optional tests |

### Python

| Workflow | Description |
|---|---|
| `python-ci.yml` | uv sync (locked) + pytest, ruff format/lint |

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
- `release-please-config.json`: release-please configuration
- `.release-please-manifest.json`: version tracking

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
    uses: st0o0/github-workflows/.github/workflows/go-ci.yml@v1.0.0
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@v1.0.0
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
    uses: st0o0/github-workflows/.github/workflows/release-please.yml@v1.0.0
    permissions:
      contents: write
      pull-requests: write
  docker:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    uses: st0o0/github-workflows/.github/workflows/docker-release.yml@v1.0.0
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
    uses: st0o0/github-workflows/.github/workflows/release-please.yml@v1.0.0
    permissions:
      contents: write
      pull-requests: write
  docker:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    uses: st0o0/github-workflows/.github/workflows/docker-release.yml@v1.0.0
    with:
      version: ${{ needs.release-please.outputs.version }}
      image-name: ghcr.io/st0o0/njord
      image-description: Multi-model weather intelligence for Home Assistant
      platforms: linux/amd64,linux/arm64,linux/arm/v7
    permissions:
      contents: read
      packages: write
      id-token: write
```

### Dev image for a PR

The reusable workflow builds on every call; gate it in the consumer repo, e.g. on a `dev-build` label:

```yaml
# .github/workflows/dev-build.yml
name: Dev Build
on:
  pull_request:
    types: [labeled, synchronize, reopened]
concurrency:
  group: dev-build-${{ github.event.pull_request.number }}
  cancel-in-progress: true
jobs:
  docker:
    if: >-
      contains(github.event.pull_request.labels.*.name, 'dev-build') &&
      github.event.pull_request.head.repo.full_name == github.repository
    uses: st0o0/github-workflows/.github/workflows/docker-dev.yml@v1.0.0
    with:
      image-name: ghcr.io/st0o0/bifrost
      image-description: Dev build of bifrost
    permissions:
      contents: read
      packages: write
      pull-requests: write
      id-token: write
```

### NuGet library (e.g. Flickr.Net)

`dotnet-build-nuget.yml` only builds, tests and packs. The publish job must live in the consumer repo, because NuGet Trusted Publishing checks the workflow that requests the OIDC token. Copy [`examples/release-nuget.yml`](examples/release-nuget.yml) as a starting point.

### Python app (e.g. ha-njord)

```yaml
# .github/workflows/ci.yml
name: CI
on: [pull_request]
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
jobs:
  python:
    uses: st0o0/github-workflows/.github/workflows/python-ci.yml@v1.0.0
  commitlint:
    uses: st0o0/github-workflows/.github/workflows/commitlint.yml@v1.0.0
```

Projects that aren't published as a package (apps, HA custom components) set
`package = false` under `[tool.uv]` in `pyproject.toml` so `uv sync` doesn't
try to build a wheel.

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
    uses: st0o0/github-workflows/.github/workflows/security.yml@v1.0.0
    with:
      docker-image-name: bifrost
    permissions:
      contents: read
      security-events: write
```

### Labels

Both jobs need `permissions:` declared **on the calling job**, not just inside the reusable workflow, since a reusable job can only be granted permissions the caller itself has, so omitting this block makes GitHub silently cap it at `pull-requests: none` and the run fails at startup with `Invalid workflow file`.

```yaml
# .github/workflows/labeler.yml
name: PR Labeler
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  label:
    uses: st0o0/github-workflows/.github/workflows/labeler.yml@v1.0.0
    permissions:
      contents: read
      pull-requests: write
```

```yaml
# .github/workflows/labels.yml
name: Sync Labels
on:
  push:
    branches: [main]
    paths: ['.github/labels.yml']
  workflow_dispatch:
jobs:
  sync:
    uses: st0o0/github-workflows/.github/workflows/labels.yml@v1.0.0
    permissions:
      contents: read
      issues: write
```

<!-- x-release-please-end -->

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
