# github-actions-argocd-image-updater

Reusable GitHub workflow that updates an image tag in a GlueOps `deployment-configurations` repo and either opens a pull request or pushes directly to the default branch.

This is the workflow that runs after a release in an application repo to roll the new container tag into ArgoCD's source of truth.

## Usage

```yaml
name: prod CD

on:
  release:
    types: [created]

jobs:
  update-prod-api:
    uses: GlueOps/github-actions-argocd-image-updater/.github/workflows/argocd-tags-ci.yml@v0.0.1
    secrets:
      GLUEOPS_DEPLOYMENT_CONFIGS_REPO_TOKEN: ${{ secrets.GLUEOPS_DEPLOYMENT_CONFIGS_REPO_TOKEN }}
    with:
      ENV: prod
      CREATE_PR: true
      DEPLOYMENT_CONFIGS_APP_NAME: api

  update-prod-ui:
    needs: update-prod-api
    uses: GlueOps/github-actions-argocd-image-updater/.github/workflows/argocd-tags-ci.yml@v0.0.1
    secrets:
      GLUEOPS_DEPLOYMENT_CONFIGS_REPO_TOKEN: ${{ secrets.GLUEOPS_DEPLOYMENT_CONFIGS_REPO_TOKEN }}
    with:
      ENV: prod
      CREATE_PR: true
      DEPLOYMENT_CONFIGS_APP_NAME: ui
```

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `ENV` | yes | — | Application environment, matching the directory name under `apps/<app>/envs/` in the deployment-configs repo. |
| `CREATE_PR` | yes | — | `true` opens a PR against the default branch. `false` commits and pushes directly. |
| `DEPLOYMENT_CONFIGS_APP_NAME` | no | `''` | App name in the deployment-configs repo. Falls back to the calling repo's name when empty. |
| `DEPLOYMENT_CONFIGS_REPO` | no | `deployment-configurations` | Name of the deployment-configs repo (assumed in the same org as the caller). |
| `DEPLOYMENT_CONFIGS_REPO_DEFAULT_BRANCH` | no | `main` | Default branch of the deployment-configs repo. |

## Secrets

| Name | Description |
|---|---|
| `GLUEOPS_DEPLOYMENT_CONFIGS_REPO_TOKEN` | GitHub PAT with **Contents: read/write** and **Pull requests: read/write** on the deployment-configs repo. |

## What it does

1. Checks out the deployment-configs repo using the provided token.
2. Resolves the image tag — release tag for `release` events, short SHA otherwise.
3. Updates `apps/<DEPLOYMENT_CONFIGS_APP_NAME>/envs/<ENV>/values.yaml` so that `image.tag` matches the resolved value.
4. Either opens a PR (`CREATE_PR: true`) or pushes directly to the default branch (`CREATE_PR: false`).

## Versioning

Pin to a tag (e.g. `@v0.0.1`) for stability. `@main` always reflects the latest commit and may change.
