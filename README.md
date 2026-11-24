# k8s-flightsim-gitops

The desired state of the cluster from
[k8s-flightsim](https://github.com/s3n-ops/k8s-flightsim). Argo CD reads this
repository and keeps the cluster in line with it.

This repository holds only manifests. Infrastructure, scripts, scenarios
and runbooks live in the lab repository.

## Layout

```
bootstrap/
  argocd/          Argo CD itself (applied once with kubectl)
  root.yaml        App of apps: manages everything in applications/
applications/      One Argo CD Application per component
platform/          Cluster services (storage, operators)
apps/              Applications
```

| Application | Source | Sync wave |
|---|---|---|
| local-path-provisioner | `platform/local-path-provisioner` (v0.0.37) | -10 |
| cloudnative-pg | Helm chart 0.29.1 | -5 |
| miniflux | `apps/miniflux` (Miniflux 2.3.5, PostgreSQL with 3 instances) | 0 |

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
