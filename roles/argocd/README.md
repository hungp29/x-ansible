# argocd

Installs [Argo CD](https://argo-cd.readthedocs.io/) on a MicroK8s cluster using the official install manifest.

## Requirements

- MicroK8s cluster (use the `microk8s` role first).
- Target host must have network access to GitHub for the install manifest.

## Role variables

| Variable | Default | Description |
|----------|---------|-------------|
| `argocd_version` | `stable` | Branch or tag for install manifest (e.g. `v2.9.3`). |
| `argocd_namespace` | `argocd` | Kubernetes namespace for Argo CD. |
| `argocd_wait_ready` | `true` | Wait for `argocd-server` deployment to be available. |
| `argocd_ingress_enabled` | `true` | Create Ingress at `argocd_path` (e.g. `https://host/argocd`). |
| `argocd_path` | `"/argocd"` | URL subpath for the UI (no trailing slash). |
| `argocd_ingress_host` | `""` | Ingress hostname (required when ingress enabled). |
| `argocd_ingress_class` | `"nginx"` | Ingress class name. |
| `argocd_ingress_tls_secret` | `"argocd-server-tls"` | TLS secret name (cert-manager creates when enabled). |
| `argocd_ingress_use_cert_manager` | `true` | Use cert-manager cluster issuer for TLS. |
| `argocd_ingress_cluster_issuer` | `"letsencrypt-prod"` | cert-manager ClusterIssuer name. |

When ingress is enabled, the server is configured with `server.insecure`, `server.rootpath`, and `server.basehref` so TLS is at the Ingress and the app is served under `argocd_path`.

## After install

**Initial admin password:**

```bash
microk8s kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**With Ingress:** open `https://<argocd_ingress_host><argocd_path>` (e.g. `https://example.com/argocd`). Default user is `admin`.

**Without Ingress (port-forward):**

```bash
microk8s kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open https://localhost:8080 (default user `admin`).
