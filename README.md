# Service templates

Owned by the platform. Teams may open PRs for templates they need.

## profiles/

What a service may be, and how ArgoCD manages it. A subscription in the GitOps
repo names a profile; the GitOps repo's `scripts/gen-appsets.sh` reads the
profiles at the tag its registry pins (`profilesRef`) and builds each
subscription's ApplicationSet from one.

| Profile | What it is |
|---------|------------|
| `web-service` | A stateless service, released through the canary. What is removed from Git is removed from the cluster. |
| `db-service` | A database. Nothing is ever deleted automatically. |

Each profile lists the settings a team may override, per environment, in its
own `env.yaml`. A change here reaches the platform only when the GitOps repo
moves `profilesRef` to a new tag and regenerates, in one reviewed PR. The
platform reviews every change to this directory (`.github/CODEOWNERS`).

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

Starting points, one per profile in `profiles/`. `scripts/new-service.sh --profile <profile>` copies the matching scaffold
into a service's `base/` in its config repo and fills in the `__PLACEHOLDERS__`.
After that the manifests belong to the team; they are copied once, not
referenced.

| Scaffold | Profile | What it is |
|----------|---------|------------|
| `web-service` | `web-service` | A Rollout and a ClusterIP Service. |
| `db-service` | `db-service` | A Postgres StatefulSet with a volume, and a headless Service. |

## Versions

Every change that environments should pick up is a new tag (`v1.0.1`). Nothing
moves until a team changes the `?ref=` in its own repo, or, for a profile, until
the GitOps repo moves `profilesRef`.
