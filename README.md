# pivot-cicd

Source unique des pipelines CI/CD de l'organisation **PIVOT-PLATFORM** : reusable workflows
GitHub Actions et composite actions, appelés par les repos `pivot-*-core` / `pivot-*-ui`.

Objectif : **corriger un pipeline à un seul endroit**, uniformiser, supprimer la duplication
(~11–16 workflows quasi identiques par repo). Tracé au backlog : **EN05.17** (sous E05 —
CI/CD & Supply-chain), voir `pivot-docs`.

## Contenu

| Chemin | Rôle |
|--------|------|
| `.github/workflows/ci-core.yml` | Reusable — CI backend Java/Maven (quality, tests, Sonar, SCA) |
| `.github/workflows/ci-ui.yml` | Reusable — CI frontend Angular/Node (quality, tests, build, Sonar, SCA) |
| `.github/workflows/_self-ci.yml` | CI interne (actionlint + yamllint) |
| `.github/workflows/plumber.yml` | Conformité supply-chain Plumber (seuil ≥ 90 %) |
| `.github/workflows/scorecard.yml` | OpenSSF Scorecard |
| `.plumber.yaml` | Gates Plumber (SHA pinning, permissions, branche protégée…) |

## Utilisation

Voir **[`docs/USAGE.md`](docs/USAGE.md)** — convention d'appel et tableau des inputs par reusable.

Règle absolue : les appelants **épinglent par SHA** (jamais par tag — gate Plumber
`actionsMustBePinnedByCommitSha`) et laissent **Dependabot** (`ecosystem: github-actions`)
bumper le SHA à chaque release de ce repo.

## Versionnage

Releases en semver (`vX.Y.Z`) pour la traçabilité ; l'appel se fait par le **SHA** du tag.

## Périmètre à venir (roadmap EN05.17)

Vagues suivantes : migration `security.yml` depuis `.github`, reusables `scorecard`/`sbom`,
`deploy*`/`release`, `dast`/`lighthouse`/`e2e`/`mutation`, rapatriement de la composite `setup`
depuis `pivot-core`, puis rollout sur les 14 repos.
