# 07-argocd-applications-and-sync: Cheatsheet

## Application Creation

### Git Repository Source
```yaml
spec:
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
    directory:
      recurse: true
```

### Helm Chart Source
```yaml
spec:
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 15.0.0
    helm:
      values: |
        replicaCount: 3
      valuesFiles:
        - values.yaml
        - values-prod.yaml
```

### Kustomize Source
```yaml
spec:
  source:
    repoURL: https://github.com/example/repo
    path: kustomize/
    kustomize:
      version: v4.5.0
```

## Sync Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| Automated | Auto-sync on Git change | Development, low-risk apps |
| Manual | Require approval | Production, high-risk apps |
| None | OutOfSync by default | Manual control only |

## Sync Policy Options

| Option | Purpose | Example |
|--------|---------|---------|
| `automated.prune` | Delete resources not in Git | `prune: true` |
| `automated.selfHeal` | Sync on cluster drift | `selfHeal: true` |
| `syncOptions.CreateNamespace` | Create namespace if missing | `CreateNamespace=true` |
| `syncOptions.PruneLast` | Delete resources last | `PruneLast=true` |
| `syncOptions.RespectIgnoreDifferences` | Allow cluster-only changes | `RespectIgnoreDifferences=true` |

## Application Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `argocd app create <name>` | Create application | With --repo, --path, --dest flags |
| `argocd app get <app>` | Get app details | Shows status and health |
| `argocd app list` | List applications | All apps in system |
| `argocd app info <app>` | Detailed info | Includes resources |
| `argocd app delete <app>` | Delete application | `--cascade=true` to delete resources |
| `argocd app sync <app>` | Manual sync | `--force` to ignore errors |
| `argocd app wait <app>` | Wait for sync | `--health` to wait for healthy |
| `argocd app logs <app>` | View app logs | Shows reconciliation logs |
| `argocd app history <app>` | Sync history | Shows past operations |
| `argocd app diff <app>` | Compare Git vs cluster | Shows differences |
| `argocd app manifests <app>` | Show manifests | Generated from source |

## Source Types

| Type | Command | Purpose |
|------|---------|---------|
| Helm | `chart: <name>` | Helm package repository |
| Git | `path: <path>` | Git repository directory |
| Kustomize | `path: <path>` | Kustomize overlay directory |
| Plugin | `plugin: <name>` | Custom config management |

## Destination Configuration

```yaml
destination:
  server: https://kubernetes.default.svc  # Local cluster
  # OR
  server: https://remote-cluster:6443     # Remote cluster
  name: my-cluster                         # Cluster name
  namespace: default                       # Namespace
```

## Sync Status Values

| Status | Meaning |
|--------|---------|
| Synced | Cluster matches Git |
| OutOfSync | Cluster differs from Git |
| Syncing | Sync in progress |
| Unknown | Cannot determine status |

## Health Status Values

| Status | Meaning |
|--------|---------|
| Healthy | All resources healthy |
| Progressing | Resources being created/updated |
| Degraded | Some resources unhealthy |
| Unknown | Cannot determine health |
| Missing | Resource not found |

## Common Sync Options

```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true           # Create namespace
    - PrunePropagationPolicy=background  # Background deletion
    - RespectIgnoreDifferences=true   # Ignore specified fields
    - Validate=false                  # Skip validation
    - PruneLast=true                  # Prune after other operations
```

## Retry Configuration

```yaml
syncPolicy:
  retry:
    limit: 5                          # Max retry attempts
    backoff:
      duration: 5s                    # Initial backoff
      factor: 2                       # Exponential multiplier
      maxDuration: 3m                 # Maximum backoff
```

## Ignore Differences

```yaml
ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
      - /spec/replicas
      - /metadata/annotations/deployment-time
```

## RBAC for Applications

| Permission | Resource | Use |
|-----------|----------|-----|
| create | applications | Create apps |
| get | applications | View apps |
| list | applications | List apps |
| delete | applications | Delete apps |
| sync | applications | Sync apps |

## Webhook Integration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: argocd-repo-webhook
  namespace: argocd
spec:
  ports:
  - port: 8000
    targetPort: 8000
  selector:
    app: argocd-repo-server
```

## Application Template

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
    path: apps/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Cluster Management

| Command | Purpose |
|---------|---------|
| `argocd cluster add <context>` | Register cluster |
| `argocd cluster list` | List clusters |
| `argocd cluster get <server>` | Get cluster info |
| `argocd cluster rm <server>` | Remove cluster |

## Source Refresh

| Command | Purpose |
|---------|---------|
| `argocd app get <app> --refresh` | Refresh source |
| `argocd app get <app> --hard-refresh` | Full refresh |
| `argocd repo update <url>` | Update repo cache |

## Troubleshooting Commands

| Command | Purpose |
|---------|---------|
| `argocd app diff <app>` | Show differences |
| `argocd app logs <app>` | View logs |
| `argocd app info <app>` | Detailed info |
| `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-repo-server` | Repo-server logs |
| `kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller` | Controller logs |

## Sync Strategies

### Progressive Sync (Manual)
```bash
argocd app sync my-app
argocd app wait my-app --sync
# Monitor: argocd app info my-app
```

### Blue-Green (App of Apps)
```yaml
spec:
  project: default
  source:
    path: blue/  # or green/
  destination:
    namespace: production
```

### Canary Release
```bash
# Deploy to canary namespace first
# Monitor metrics
# Gradually shift traffic
# Full production deployment
```
