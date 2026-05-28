# Usage Guide

Detailed reference for all workflows and actions in this pipeline library.

## Prerequisites

- Repository must be in the `Leberkas-org` GitHub organization (or fork this repo for your own org)
- Copy starter files from `starter/` directory to your repo root
- Commit messages must follow [Conventional Commits](https://www.conventionalcommits.org/) format

## Workflow Reference

### build-test.yml

Build and test a .NET solution with optional code coverage.

**Usage:**
```yaml
jobs:
  build:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/build-test.yml@v1
    with:
      solution-path: ./src/MyProject.slnx
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dotnet-version-file` | no | `./src/global.json` | Path to global.json |
| `solution-path` | **yes** | — | Path to .slnx or .sln |
| `build-configuration` | no | `Release` | Build configuration |
| `coverage-enabled` | no | `true` | Generate coverage reports |
| `test-result-directory` | no | `testresults` | Test output directory |
| `extra-apt-packages` | no | `""` | Extra apt packages (e.g. `libmsquic`) |
| `sonar-enabled` | no | `false` | Enable SonarCloud analysis |
| `sonar-project-key` | no | `""` | SonarCloud project key |
| `sonar-organization` | no | `""` | SonarCloud organization |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `sonar-token` | if sonar-enabled | SonarCloud authentication token |

**Usage with SonarCloud:**
```yaml
jobs:
  build:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/build-test.yml@v1
    with:
      solution-path: ./src/MyProject.slnx
      sonar-enabled: true
      sonar-project-key: my-org_my-project
      sonar-organization: my-org
    secrets:
      sonar-token: ${{ secrets.SONAR_TOKEN }}
```

---

### nuget-publish.yml

Pack and publish NuGet packages. Typically called by `release.yml`, not directly.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dotnet-version-file` | no | `./src/global.json` | Path to global.json |
| `solution-path` | **yes** | — | Path to .slnx or .sln |
| `version` | **yes** | — | SemVer version |
| `tag-name` | **yes** | — | Git tag for release attachment |
| `package-output-directory` | no | `./packages` | Output directory |
| `nuget-source` | no | `https://api.nuget.org/v3/index.json` | NuGet feed URL |
| `trusted-publishing` | no | `false` | Use OIDC trusted publishing instead of API key |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `nuget-api-key` | if not trusted-publishing | NuGet.org API key |
| `nuget-username` | if trusted-publishing | NuGet.org username |

---

### docker-build-push.yml

Build and push Docker images. Typically called by `release.yml`, not directly.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dockerfile-path` | no | `./Dockerfile` | Path to Dockerfile |
| `context` | no | `.` | Docker build context |
| `image-name` | **yes** | — | Image name |
| `registry` | no | `ghcr.io` | Registry URL |
| `version` | **yes** | — | SemVer version |
| `platforms` | no | `linux/amd64` | Target platforms |
| `build-args` | no | `""` | Build arguments |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `registry-username` | **yes** | Registry username |
| `registry-password` | **yes** | Registry password/token |

---

### release.yml

Orchestrates release-please, NuGet publishing, and Docker publishing.

**Usage (NuGet with API key):**
```yaml
jobs:
  release:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    uses: Leberkas-org/template.pipeline.github/.github/workflows/release.yml@v1
    with:
      nuget-publish: true
      solution-path: ./src/MyProject.slnx
    secrets:
      nuget-api-key: ${{ secrets.NUGET_SECRET }}
```

**Usage (NuGet with trusted publishing):**
```yaml
jobs:
  release:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    uses: Leberkas-org/template.pipeline.github/.github/workflows/release.yml@v1
    with:
      nuget-publish: true
      nuget-trusted-publishing: true
      solution-path: ./src/MyProject.slnx
    secrets:
      nuget-username: ${{ secrets.NUGET_USER }}
```

**Usage (Docker only):**
```yaml
jobs:
  release:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    uses: Leberkas-org/template.pipeline.github/.github/workflows/release.yml@v1
    with:
      docker-publish: true
      docker-image-name: my-service
      docker-registry: ghcr.io
      dockerfile-path: ./src/MyService/Dockerfile
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

**Usage (Both):**
```yaml
jobs:
  release:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    uses: Leberkas-org/template.pipeline.github/.github/workflows/release.yml@v1
    with:
      nuget-publish: true
      docker-publish: true
      solution-path: ./src/MyProject.slnx
      docker-image-name: my-service
    secrets:
      nuget-api-key: ${{ secrets.NUGET_SECRET }}
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `nuget-publish` | no | `false` | Enable NuGet publishing |
| `nuget-trusted-publishing` | no | `false` | Use OIDC trusted publishing |
| `docker-publish` | no | `false` | Enable Docker publishing |
| `dotnet-version-file` | no | `./src/global.json` | Path to global.json |
| `solution-path` | no | `""` | Path to .slnx/.sln |
| `dockerfile-path` | no | `./Dockerfile` | Path to Dockerfile |
| `docker-registry` | no | `ghcr.io` | Container registry |
| `docker-image-name` | no | `""` | Docker image name |
| `docker-platforms` | no | `linux/amd64` | Target platforms |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `nuget-api-key` | if nuget-publish without trusted publishing | NuGet API key |
| `nuget-username` | if nuget-trusted-publishing | NuGet.org username |
| `registry-username` | if docker-publish | Registry username |
| `registry-password` | if docker-publish | Registry password |

---

### sonar.yml

Run SonarCloud analysis with code coverage integration.

**Usage:**
```yaml
jobs:
  sonar:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/sonar.yml@v1
    with:
      solution-path: ./src/MyProject.slnx
      sonar-project-key: my-org_my-project
      sonar-organization: my-org
    secrets:
      sonar-token: ${{ secrets.SONAR_TOKEN }}
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dotnet-version-file` | no | `./src/global.json` | Path to global.json |
| `solution-path` | **yes** | — | Path to .slnx or .sln |
| `sonar-project-key` | **yes** | — | SonarCloud project key |
| `sonar-organization` | **yes** | — | SonarCloud organization |
| `build-configuration` | no | `Release` | Build configuration |
| `test-result-directory` | no | `testresults` | Test output directory |
| `extra-apt-packages` | no | `""` | Extra apt packages (e.g. `libmsquic`) |

**Secrets:**
| Secret | Required | Description |
|--------|----------|-------------|
| `sonar-token` | **yes** | SonarCloud authentication token |

---

### commitlint.yml

Validate PR titles and commit messages.

**Usage:**
```yaml
jobs:
  lint:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/commitlint.yml@v1
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `commitlint-config-file` | no | `commitlint.config.mjs` | Path to commitlint config |

---

### docs-build.yml

Build a VitePress (or other Node.js-based) docs site on PRs for validation.

**Usage:**
```yaml
jobs:
  docs:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/docs-build.yml@v1
    with:
      docs-directory: docs
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `docs-directory` | no | `docs` | Working directory containing package.json |
| `node-version` | no | `22` | Node.js version |
| `build-command` | no | `npm run build` | Build script |

---

### docs-deploy.yml

Build a docs site and deploy to GitHub Pages. Typically called by `release.yml`, not directly.

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `docs-directory` | no | `docs` | Working directory containing package.json |
| `node-version` | no | `22` | Node.js version |
| `build-command` | no | `npm run build` | Build script |
| `output-directory` | no | `docs/.vitepress/dist` | Path to built site output |

---

### release.yml — Docs Deploy Option

Add `docs-deploy: true` to the release workflow to deploy docs on each release:

```yaml
jobs:
  release:
    uses: Leberkas-org/template.pipeline.github/.github/workflows/release.yml@v1
    with:
      nuget-publish: true
      docs-deploy: true
      docs-directory: docs
      solution-path: ./src/MyProject.slnx
    secrets:
      nuget-api-key: ${{ secrets.NUGET_SECRET }}
```

Additional release.yml inputs for docs:

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `docs-deploy` | no | `false` | Enable docs deploy on release |
| `docs-directory` | no | `docs` | Docs working directory |
| `docs-build-command` | no | `npm run build` | Build script |
| `docs-output-directory` | no | `docs/.vitepress/dist` | Built site path |

## Starter Files

Copy these from `starter/` to your repo root:

| File | Purpose |
|------|---------|
| `commitlint.config.mjs` | Enforces Conventional Commits format |
| `release-please-config.json` | Configures release-please changelog sections |
| `.release-please-manifest.json` | Tracks current version (start at `0.1.0`) |

## Versioning

This library uses Git tags for versioning (`v1`, `v1.0.0`).

- Pin to **major tag** for stability: `@v1`
- Pin to **exact tag** for reproducibility: `@v1.0.0`
- Pin to **branch** for development: `@main`

## Commit Convention

Consumer repos must use Conventional Commits:

| Prefix | Version Bump | Example |
|--------|-------------|---------|
| `feat:` | Minor | `feat: add user export endpoint` |
| `fix:` | Patch | `fix: correct date parsing in reports` |
| `perf:` | Patch | `perf: cache database queries` |
| `feat!:` or `BREAKING CHANGE:` | Major | `feat!: redesign auth API` |
| `docs:`, `chore:`, `test:`, `ci:`, `build:` | None | `docs: update API reference` |
| `deps:` | None | `deps: bump Akka to 1.5.70` |

## Constraints

- **Nesting limit:** GitHub allows max 4 levels of `workflow_call`. Consumer → release.yml → nuget/docker is 3 levels. Consumers have 1 more level available.
- **Secrets:** Must be explicitly passed at each level. Use `secrets: inherit` only within the same organization.
- **Path filters:** Add `paths-ignore` in your consumer workflow, not in the reusable workflows.
