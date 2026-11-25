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
| local-path-provisioner | platform | upstream manifest via `platform/local-path-provisioner` | -10 |
| cert-manager | platform | Jetstack Helm chart | -8 |
| cert-manager-issuers | platform | `platform/cert-manager-issuers` (ClusterIssuer `lab-ca`) | -7 |
| envoy-gateway | platform | Envoy Gateway Helm chart (OCI), includes the Gateway API CRDs | -6 |
| cloudnative-pg | platform | CloudNativePG Helm chart | -5 |
| gateway | platform | `platform/gateway` (GatewayClass, Gateway `lab`, certificates) | -4 |
| miniflux | apps | `apps/miniflux` (Miniflux, PostgreSQL with 3 instances) | 0 |

Every version is pinned in exactly one place: `targetRevision` in
`applications/*.yaml` for Helm charts, the URL or image tag in the
manifests under `platform/` and `apps/`. This README names no versions.

## Projects

`bootstrap/argocd/projects.yaml` defines two Argo CD projects. Every
Application must name one of them; the `default` project allows nothing.

| Project | Sources | Destinations | Cluster-scoped resources |
|---|---|---|---|
| platform | this repository, CloudNativePG, Jetstack and Envoy Gateway charts | `argocd`, `local-path-storage`, `cnpg-system`, `cert-manager`, `envoy-gateway-system`, `gateway` | Namespace, CRDs, webhooks, ClusterRoles, StorageClass, ClusterIssuer, GatewayClass |
| apps | this repository | `miniflux` | Namespace only |

The projects are applied with Argo CD itself, not by the app of apps. A
change to them takes effect only after `just argocd` in the lab repository,
so a push here cannot widen its own rights. A new application namespace
needs an entry under `destinations` of `apps`.

## Gateway

All web traffic enters through one Gateway API `Gateway` (`gateway/lab`,
Envoy Gateway) on `10.17.3.100`. The lab DNS resolves every name under
`k8s-flightsim.internal` to that address.

- One HTTPS listener per hostname, each with its own certificate from the
  ClusterIssuer `lab-ca`. No wildcard certificate.
- Each listener names the namespaces that may attach routes; any other
  namespace is refused.
- Port 80 only redirects to HTTPS.
- A new application needs a listener, a certificate and an `HTTPRoute`.

## Secrets

No secrets in this repository, not even encrypted ones. The lab repository
creates them in the cluster (`just secrets`); the manifests reference them
by name only.

| Secret | Namespace | Keys | Source |
|---|---|---|---|
| `miniflux-admin` | miniflux | `ADMIN_USERNAME`, `ADMIN_PASSWORD` | `just secrets` |
| `miniflux-db-app` | miniflux | `uri` and others | created by CloudNativePG |
| `lab-intermediate-ca` | cert-manager | `tls.crt` (intermediate and root), `tls.key` | `just secrets`; the root key never enters the cluster |

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
