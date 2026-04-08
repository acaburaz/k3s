# k3s Cluster GitOps

Managed with **ArgoCD** — all app configs live here, synced automatically to the cluster.

## Cluster
- **Node**: m710q (`192.168.0.134`) | External IP: `85.194.34.199`
- **Ingress**: Traefik | **TLS**: cert-manager + Let's Encrypt
- **ArgoCD**: `https://192.168.0.134:32742` (next to Rancher on 32740)
- **Image Registry**: `ghcr.io/acaburaz` (GitHub Container Registry)

## Structure

```
apps/
  mynforkort/     PHP app (xn--myndighetsfrkortningar-4hc.nu)
  mailcow/        Mail server — DMS (Postfix+Dovecot) + Roundcube
  logiplan/       LogiPlan WebAPI (.NET)
  mysql/          MySQL database
  registry/       Local Docker registry
argocd/
  applications.yaml        ArgoCD Application CRDs (one per app)
  argocd-server-nodeport.yaml
infra/
  cert-manager/   Let's Encrypt ClusterIssuer
```

## Secrets
`apps/mailcow/secrets.yaml` contains `<BASE64_CHANGE_ME>` placeholders for sensitive values.
Apply manually: `kubectl apply -f apps/mailcow/secrets.yaml` after filling in real values.

## Adding an app
1. Create `apps/<name>/` with Kubernetes manifests
2. Add an `Application` entry to `argocd/applications.yaml`
3. Push — ArgoCD syncs automatically
