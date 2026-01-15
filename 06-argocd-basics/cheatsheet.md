# 06-argocd-basics: Cheatsheet

## Installation & Setup

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl apply -n argocd -f install.yaml` | Install ArgoCD | `kubectl apply -n argocd -f https://raw.github.../install.yaml` |
| `kubectl get pods -n argocd` | Verify installation | See all running ArgoCD pods |
| `kubectl port-forward svc/argocd-server -n argocd 8080:443` | Access UI | Forwards local port 8080 to service |
| `kubectl -n argocd get secret argocd-initial-admin-secret` | Get password | Extract initial admin password |

## CLI Installation

| Method | Command |
|--------|---------|
| Download binary | `curl -sSL -o argocd https://github.com/.../argocd-linux-amd64` |
| macOS | `brew install argocd` |
| Windows | `choco install argocd` |
| Verify | `argocd version` |

## ArgoCD CLI Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `argocd login <server>` | Login to ArgoCD | `argocd login localhost:8080 --insecure` |
| `argocd logout` | Logout | Clears stored credentials |
| `argocd repo add <url>` | Add repository | `argocd repo add https://github.com/example/repo` |
| `argocd repo list` | List repositories | Shows all connected repos |
| `argocd repo get <url>` | Get repo details | Shows connection status |
| `argocd app list` | List applications | Shows all ArgoCD applications |
| `argocd app create` | Create application | With YAML or flags |
| `argocd app get <app>` | Get app details | Shows status, health, sync |
| `argocd app info <app>` | App information | Detailed info with refresh |
| `argocd app sync <app>` | Manual sync | Syncs app with Git |
| `argocd app wait <app>` | Wait for sync | Blocks until sync complete |
| `argocd app logs <app>` | View app logs | Shows application logs |
| `argocd app history <app>` | View history | Shows sync operations |

## Application Creation

### Helm Chart Example
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 15.0.0
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

### Git Repository Example
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
```

## Repository Management

| Command | Purpose |
|---------|---------|
| `argocd repo add <url> --username <user> --password <token>` | Add with auth |
| `argocd repo add git@github.com:repo.git --ssh-private-key-path ~/.ssh/id_rsa` | Add with SSH |
| `argocd repo list` | List all repos |
| `argocd repo get <url>` | Get repo info |
| `argocd repo update <url>` | Refresh repo |
| `argocd repo rm <url>` | Remove repo |

## Account & RBAC Management

| Command | Purpose |
|---------|---------|
| `argocd account list` | List accounts |
| `argocd account create <username>` | Create user |
| `argocd account update-password --account <user> --new-password <pwd>` | Change password |
| `argocd account delete <user>` | Delete user |
| `kubectl patch configmap argocd-rbac-cm -n argocd -p ...` | Update RBAC policies |

## Status & Monitoring

| Command | Purpose | Example |
|---------|---------|---------|
| `argocd app get <app>` | App status | Returns status, health, sync |
| `argocd app info <app>` | Detailed info | With repo, destination, resources |
| `argocd app wait <app> --health` | Wait for health | Blocks until healthy |
| `argocd app wait <app> --sync` | Wait for sync | Blocks until synced |
| `argocd admin stats` | Admin statistics | Server stats |
| `argocd version` | Version info | CLI and server version |

## Application Management

| Command | Purpose |
|---------|---------|
| `kubectl apply -f app.yaml -n argocd` | Create from YAML |
| `kubectl get application -n argocd` | List applications |
| `kubectl describe application <app> -n argocd` | Get details |
| `kubectl delete application <app> -n argocd` | Delete application |
| `kubectl patch application <app> -n argocd -p '...'` | Update application |

## Application Sync Operations

| Command | Purpose | Example |
|---------|---------|---------|
| `argocd app sync <app>` | Manual sync | `argocd app sync hello-world` |
| `argocd app sync <app> --force` | Force sync (discard local changes) | `argocd app sync hello-world --force` |
| `argocd app sync <app> --prune` | Sync and delete out-of-sync resources | `argocd app sync hello-world --prune` |
| `argocd app wait <app>` | Wait for sync completion | Blocks until done |

## Troubleshooting

| Command | Purpose |
|---------|---------|
| `kubectl logs -n argocd deployment/argocd-server` | Server logs |
| `kubectl logs -n argocd deployment/argocd-repo-server` | Repo-server logs |
| `kubectl logs -n argocd statefulset/argocd-application-controller` | Controller logs |
| `argocd app logs <app>` | Application logs |
| `kubectl get events -n argocd` | ArgoCD namespace events |
| `argocd repo get <url>` | Check repo connectivity |

## Common Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-n, --namespace` | Target namespace | `-n argocd` |
| `--insecure` | Skip TLS verification | `argocd login --insecure` |
| `--refresh` | Refresh from source | `argocd app get app --refresh` |
| `--force` | Force operation | `argocd app sync app --force` |
| `--prune` | Delete out-of-sync resources | `argocd app sync app --prune` |
| `--tail` | Follow logs | `kubectl logs --tail=50` |

## Core Components

| Component | Role | Namespace |
|-----------|------|-----------|
| argocd-server | REST API, Web UI | argocd |
| argocd-repo-server | Git/Helm processing | argocd |
| argocd-application-controller | Reconciliation engine | argocd |
| argocd-dex-server | SSO authentication | argocd |
| argocd-redis | Caching | argocd |
