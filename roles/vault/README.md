# vault

Installs **HashiCorp Vault** and **External Secrets Operator** on a MicroK8s cluster via Helm, and exposes the Vault UI through the nginx ingress at a configurable sub-path (default `/vault`).

## What this role does

1. Adds the HashiCorp Helm repo and installs/upgrades the `vault` chart.
2. In **standalone mode** (default): initialises Vault on first boot, seals unseal keys and the root token into a Kubernetes Secret (`vault-init-keys`), and unseals automatically.
3. On subsequent runs where Vault is sealed (e.g. after a pod restart): fetches the stored keys and unseals.
4. Adds the External Secrets Helm repo and installs/upgrades the `external-secrets` chart (including CRDs).
5. Creates an nginx Ingress that maps `<host>/vault` → `vault-ui:8200`.

## Requirements

- `microk8s` with `helm3`, `ingress`, and `cert-manager` addons enabled (handled by the `microk8s` role).
- `microk8s helm3` available on the target host.

## Role variables

| Variable | Default | Description |
|---|---|---|
| `vault_namespace` | `vault` | Kubernetes namespace for Vault |
| `vault_helm_release` | `vault` | Helm release name |
| `vault_chart_version` | `""` | Pin a specific chart version (empty = latest) |
| `vault_dev_mode` | `false` | Run Vault in dev mode (ephemeral; skips init/unseal) |
| `vault_key_shares` | `1` | Number of unseal key shares |
| `vault_key_threshold` | `1` | Unseal key threshold |
| `vault_init_secret_name` | `vault-init-keys` | Name of the Secret storing unseal keys + root token |
| `vault_ingress_enabled` | `true` | Whether to create the Vault UI Ingress |
| `vault_ingress_host` | `""` | Ingress host (required when enabled) |
| `vault_ingress_path` | `/vault` | Sub-path prefix for the Vault UI |
| `vault_ingress_class` | `nginx` | Ingress class name |
| `vault_ingress_tls_secret` | `vault-tls` | TLS Secret name |
| `vault_ingress_use_cert_manager` | `true` | Annotate with `cert-manager.io/cluster-issuer` |
| `vault_ingress_cluster_issuer` | `letsencrypt-prod` | ClusterIssuer name |
| `eso_namespace` | `external-secrets` | Namespace for External Secrets Operator |
| `eso_helm_release` | `external-secrets` | Helm release name |
| `eso_chart_version` | `""` | Pin a specific chart version (empty = latest) |

## Example group_vars

```yaml
# inventories/production/group_vars/all/vault.yml
vault_ingress_enabled: true
vault_ingress_host: "example.tplinkdns.com"
```

## Accessing Vault after deployment

The **unseal keys** and **root token** are stored in a Kubernetes Secret inside the cluster:

```bash
microk8s kubectl get secret vault-init-keys -n vault -o jsonpath='{.data.root_token}' | base64 -d
```

The Vault UI is available at `https://<vault_ingress_host>/vault/ui`.

## Notes

- `vault_dev_mode: true` is useful for local testing. Data is stored in memory and lost on pod restart; no init/unseal step is needed.
- For production, consider using Vault's auto-unseal with a cloud KMS instead of stored unseal keys.
