# k8s-flightsim-gitops

The desired state of the cluster from
[k8s-flightsim](https://github.com/s3n-ops/k8s-flightsim). Argo CD reads this
repository and keeps the cluster in line with it.

This repository holds only manifests. Infrastructure, scripts, scenarios
and runbooks live in the lab repository.

## Layout

```
bootstrap/
  argocd/          Argo CD itself and its projects (applied with kubectl)
  root.yaml        App of apps: manages everything in applications/
applications/      One Argo CD Application per component
platform/          Cluster services (storage, operators)
apps/              Applications
```

| Application | Project | Source | Sync wave |
|---|---|---|---|
| local-path-provisioner | platform | `platform/local-path-provisioner` (v0.0.37) | -10 |
| cloudnative-pg | platform | Helm chart 0.29.1 | -5 |
| miniflux | apps | `apps/miniflux` (Miniflux 2.3.5, PostgreSQL with 3 instances) | 0 |

## Projects

`bootstrap/argocd/projects.yaml` defines two Argo CD projects. Every
Application must name one of them; the `default` project allows nothing.

| Project | Sources | Destinations | Cluster-scoped resources |
|---|---|---|---|
| platform | this repository, CloudNativePG charts | `argocd`, `local-path-storage`, `cnpg-system` | Namespace, CRDs, webhooks, ClusterRoles, StorageClass |
| apps | this repository | `miniflux` | Namespace only |

The projects are applied with Argo CD itself, not by the app of apps. A
change to them takes effect only after `just argocd` in the lab repository,
so a push here cannot widen its own rights. A new application namespace
needs an entry under `destinations` of `apps`.

## Secrets

No secrets in this repository, not even encrypted ones. The lab repository
creates them in the cluster (`just secrets`); the manifests reference them
by name only.

| Secret | Namespace | Keys | Source |
|---|---|---|---|
| `miniflux-admin` | miniflux | `ADMIN_USERNAME`, `ADMIN_PASSWORD` | `just secrets` |
| `miniflux-db-app` | miniflux | `uri` and others | created by CloudNativePG |

## Bootstrap

From the lab repository:

```bash
just secrets
just argocd
```

## License

MIT, see [LICENSE](LICENSE).

## AI Disclosure

Development was assisted by AI tools (Claude).
