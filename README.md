# Service templates

Owned by the platform. Teams may open PRs for templates they need.

## components/

Behaviour a service opts into from its own environment kustomization, as a
remote component pinned to a tag of this repository:

    components:
      - https://github.com/<owner>/<this repo>//components/replicas-1?ref=v1.0.0

| Component | What it does |
|-----------|--------------|
| `replicas-1` | Every Rollout runs one replica (dev). |
| `canary-manual-gate` | Every Rollout canaries at 50 percent and waits for a person. |
| `storage-gp3` | Every StatefulSet uses the `gp3` storage class (EKS). |

A component cannot take a parameter. Anything that differs per service or per
environment, such as an Ingress host, stays in the team's own kustomization.

## scaffolds/

Starting points, one per platform profile (`platform/profiles/` in the GitOps
repo). `scripts/new-service.sh --profile <profile>` copies the matching scaffold
into a service's `base/` in its config repo and fills in the `__PLACEHOLDERS__`.
After that the manifests belong to the team; they are copied once, not
referenced.

| Scaffold | Profile | What it is |
|----------|---------|------------|
| `web-service` | `web-service` | A Rollout and a ClusterIP Service. |
| `db-service` | `db-service` | A Postgres StatefulSet with a volume, and a headless Service. |

## Versions

Every change that environments should pick up is a new tag (`v1.0.1`). Nothing
moves until a team changes the `?ref=` in its own repo.
