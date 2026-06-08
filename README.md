# helm-charts

Helm charts authored for the **ShopHub** platform. This is the IaC repository
required by spec section 5.3 — every chart we write lives here (third-party
charts such as CNPG, kube-prometheus-stack, Loki and Tempo are *not* vendored
here; they are referenced from the [`kube-state`](https://github.com/shophub-devoops/kube-state)
repository).

## Structure

```
helm-charts
├── README.md
└── charts/
    ├── shop/            # thin chart that renders a single Shop custom resource
    ├── shop-operator/   # the operator Deployment + RBAC + the 3 CRDs
    └── shophub/         # the ShopHub backend + DB + RBAC + JWT + ServiceMonitor
```

## Charts

| Chart | What it installs | Notes |
|-------|------------------|-------|
| **shop-operator** | Operator Deployment, ClusterRole/Binding, Service, ServiceAccount, the three CRDs (`Shop`, `DiscordChannel`, `Wallet`), and a `PrometheusRule` with Shop + cluster alerts | Image is published separately as `shop-operator-controller` to avoid an OCI name collision with this chart |
| **shophub** | ShopHub backend Deployment, cluster-wide RBAC (manage `Shop` CRs across tenant namespaces), CNPG `users` database, JWT signing Secret, Service and ServiceMonitor | Depends on the operator's CRDs and the prometheus-operator CRDs (ServiceMonitor) being present in the cluster |
| **shop** | A single `Shop` CR plus an optional Discord webhook Secret | Used for quick manual provisioning / testing without going through the ShopHub UI |

## Publishing (OCI)

Charts are packaged and pushed to DockerHub as OCI artifacts by the
`helm-publish` CI workflow:

```
oci://registry-1.docker.io/urospetraskovic/shop-operator
oci://registry-1.docker.io/urospetraskovic/shophub
```

The published version is taken from each chart's `Chart.yaml` `version` field
(not the git tag). `kube-state` pins the exact versions it deploys.

## Local development

```bash
# lint every chart (same as the helm-lint CI workflow)
helm lint charts/shop charts/shop-operator charts/shophub

# render a chart to inspect the generated manifests
helm template charts/shop-operator

# install the operator into a local cluster
helm install shop-operator charts/shop-operator -n shop-operator-system --create-namespace
```

## CI

- **helm-lint** — runs `helm lint` on every chart on each PR.
- **helm-publish** — packages and pushes charts to the OCI registry on `main` / tags.
- **commit-lint** — enforces Conventional Commits on PR titles/commits.
