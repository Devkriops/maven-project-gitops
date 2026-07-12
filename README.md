# harness-lab-gitops

GitOps config repo for the `harness-lab` app, managed by Harness GitOps + Argo Rollouts.

## Structure

- `base/` - shared Helm chart (Rollout + Service). Do not put env-specific values here.
- `environments/<env>/values.yaml` - per-env overrides (image tag, replicas, resources).

## Environments

| Env   | Values file                     | Sync policy              |
|-------|---------------------------------|--------------------------|
| dev   | environments/dev/values.yaml    | auto-sync + self-heal    |
| stage | environments/stage/values.yaml  | auto-sync (PR-gated)     |
| prod  | environments/prod/values.yaml   | manual / approval gate   |

## Harness GitOps Application (one per env)

- Source: this repo, path `base/`, Helm values file `environments/<env>/values.yaml`
- Destination: the env's cluster + namespace

## Promotion flow

1. CI builds & pushes a new image tag.
2. Bump `image:` in `environments/dev/values.yaml` -> dev deploys.
3. Open a PR copying that tag into `environments/stage/values.yaml` -> merge -> stage.
4. Open a PR copying that tag into `environments/prod/values.yaml` -> approve -> prod.

## Rollback

- Mid-rollout failure: Argo Rollouts auto-aborts to the stable version.
- After full deploy: revert the commit in Git (or roll back via Harness History & Rollback tab).
