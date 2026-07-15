# Utilisation des reusable workflows `pivot-cicd`

## Convention d'appel

Un repo consommateur remplace le contenu de son `ci.yml` par un **stub mince** qui délègue au
reusable, épinglé par SHA :

```yaml
name: CI

on:
  push:
    branches: [main, develop]
    paths-ignore: ['docs/**', '**.md', '.github/ISSUE_TEMPLATE/**']
  pull_request:
    paths-ignore: ['docs/**', '**.md', '.github/ISSUE_TEMPLATE/**']
  workflow_dispatch:

# Plafond de permissions accordé au reusable (un reusable ne peut pas le dépasser).
permissions:
  contents: read
  actions: read
  security-events: write

concurrency:
  group: ${{ github.workflow }}-${{ github.head_ref || github.ref_name }}
  cancel-in-progress: true

jobs:
  ci:
    uses: PIVOT-PLATFORM/pivot-cicd/.github/workflows/ci-core.yml@<SHA>  # vX.Y.Z
    with:
      sonar-project-key: PIVOT-PLATFORM_<repo>
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

> Ne jamais utiliser `secrets: inherit` (interdit par le gate Plumber
> `reusableWorkflowsMustNotInheritSecrets`) — passer chaque secret explicitement.
> `GITHUB_TOKEN` est fourni automatiquement au reusable.

## `ci-core.yml` — inputs

| Input | Type | Défaut | Rôle |
|-------|------|--------|------|
| `java-version` | string | `25` | Version Temurin à installer |
| `maven-compiler-release` | string | `24` | Valeur de `-Dmaven.compiler.release` |
| `sonar-project-key` | string | *(requis)* | Clé projet SonarCloud (`PIVOT-PLATFORM_<repo>`) |
| `run-sonar` | boolean | `true` | Activer le job SonarCloud |
| `runs-on` | string | `ubuntu-latest` | Runner |

**Secrets** : `SONAR_TOKEN` (requis si `run-sonar=true`). Jobs : `quality-backend`,
`tests-backend` (service Redis provisionné), `sonar` (conditionné), `sca` (Trivy).

## `ci-ui.yml` — inputs

| Input | Type | Défaut | Rôle |
|-------|------|--------|------|
| `node-version` | string | `24` | Version Node.js à installer |
| `sonar-project-key` | string | *(requis)* | Clé projet SonarCloud (`PIVOT-PLATFORM_<repo>`) |
| `run-sonar` | boolean | `true` | Activer le job SonarCloud |
| `runs-on` | string | `ubuntu-latest` | Runner |

**Secrets** : `SONAR_TOKEN`. Jobs : `quality` (tsc + ESLint), `tests` (Vitest), `build`
(prod), `sonar` (conditionné), `sca` (Trivy + `npm audit`). Les gates a11y/e2e/mutation
(Lighthouse, Playwright, Stryker) restent dans leurs workflows dédiés, hors `ci-ui.yml`.

## Mise à jour du SHA épinglé

Dependabot (`ecosystem: github-actions`) ouvre automatiquement une PR de bump dans chaque repo
consommateur à chaque nouvelle release de `pivot-cicd`. Ne jamais repasser à un tag mobile.
