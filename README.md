# template.pipeline.github

Reusable GitHub Actions workflows and composite actions for .NET projects in the Leberkas-org organization.

## Quick Start

**1.** Copy starter files to your repo root:

```
starter/commitlint.config.mjs     → commitlint.config.mjs
starter/release-please-config.json → release-please-config.json
starter/.release-please-manifest.json → .release-please-manifest.json
```

**2.** Create `.github/workflows/ci.yml` in your repo:

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/commitlint.yml@v1

  build:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/build-test.yml@v1
    with:
      solution-path: ./src/MyProject.slnx

  release:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: build
    uses: Leberkas-org/template.pipeline.github/.github/workflows/release.yml@v1
    with:
      nuget-publish: true
      solution-path: ./src/MyProject.slnx
    secrets:
      nuget-api-key: ${{ secrets.NUGET_SECRET }}
```

## Available Workflows

| Workflow | Trigger | Description |
|----------|---------|-------------|
| `build-test.yml` | `workflow_call` | Build, test, and generate code coverage |
| `nuget-publish.yml` | `workflow_call` | Pack and publish NuGet packages |
| `docker-build-push.yml` | `workflow_call` | Build and push Docker images |
| `release.yml` | `workflow_call` | release-please + orchestrate publish |
| `commitlint.yml` | `workflow_call` | PR title and commit message validation |
| `docs-build.yml` | `workflow_call` | Build VitePress/docs site (PR validation) |
| `docs-deploy.yml` | `workflow_call` | Build + deploy docs to GitHub Pages |

## Composite Actions

| Action | Description |
|--------|-------------|
| `actions/dotnet-setup` | .NET SDK install + NuGet cache + restore |
| `actions/nuget-auth` | Private NuGet feed authentication |
| `actions/docker-login` | Registry-agnostic Docker login |

See [docs/usage.md](docs/usage.md) for detailed input/secret reference.
