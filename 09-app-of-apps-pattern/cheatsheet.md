# 09-app-of-apps-pattern: Cheatsheet

## App-of-Apps Structure

```
root (parent)
├── infrastructure (wave 0)
│   ├── namespaces
│   ├── rbac
│   └── secrets
├── platforms (wave 1)
│   ├── postgres
│   ├── redis
│   └── logging
└── applications (wave 2)
    ├── frontend
    ├── backend
    └── api
```

## Sync Waves Ordering

| Wave | Layer | Purpose | Example |
|------|-------|---------|---------|
| 0 | Infrastructure | Namespaces, RBAC, secrets | Create cluster foundation |
| 1 | Platforms | Data stores, message queues | Shared services |
| 2 | Applications | Microservices | Business logic |
| 3+ | Monitoring | Observability | Optional |

## Parent Application Template

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/example/app-of-apps
    targetRevision: main
    path: app-of-apps
    directory:
      recurse: true
      include: '*-app.yaml'
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Child Application Template

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infrastructure
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "0"
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/example/app-of-apps
    targetRevision: main
    path: infrastructure/
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Deployment Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl apply -f root-app.yaml` | Bootstrap system | Creates all layers |
| `argocd app list` | Show all applications | List parent + children |
| `argocd app get root` | Parent status | Shows health and sync |
| `argocd app get infrastructure` | Child status | Layer-specific status |
| `argocd app wait root --sync` | Wait for complete | Blocks until all synced |
| `argocd app logs root` | Parent logs | Reconciliation details |
| `argocd app delete root --cascade=true` | Delete with cascade | Cleans up all children |

## Dependency Order

```
kubectl apply root-app.yaml
    ↓
    Wave 0: Infrastructure (namespaces, RBAC)
        ↓ (when healthy)
    Wave 1: Platforms (databases, caches)
        ↓ (when healthy)
    Wave 2: Applications (microservices)
        ↓ (complete)
```

## Health Status Propagation

| Status | Infrastructure | Platforms | Applications |
|--------|-----------------|-----------|--------------|
| Healthy | ✓ | ✓ | ✓ |
| Degraded | ✗ | Blocked | Blocked |
| Unknown | ✗ | Blocked | Blocked |

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| OutOfSync | `argocd app get <app> --refresh` |
| Sync stuck | `argocd app sync <app> --force` |
| Child missing | `kubectl get application -n argocd` |
| Resource ordering | Check sync-wave annotations |
| Cascading delete fails | Verify finalizers present |
| Logs | `argocd app logs <app>` |

## Kubernetes Resources Needed

```bash
# Namespace with RBAC
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
  namespace: myapp
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

## Database Service Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: data
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        ports:
        - containerPort: 5432
        readinessProbe:
          exec:
            command: ["pg_isready", "-U", "postgres"]
          initialDelaySeconds: 10
          periodSeconds: 5
```

## Application Dependency Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      initContainers:
      - name: wait-for-db
        image: busybox
        command: ['sh', '-c', 'until nc -z postgres.data 5432; do echo waiting; sleep 2; done']
      containers:
      - name: backend
        image: myapp:backend-1.0
        ports:
        - containerPort: 8080
```

## Finalizer Configuration

```yaml
metadata:
  finalizers:
    - resources-finalizer.argocd.argoproj.io
```

## Directory Structure Best Practices

- Infrastructure: Foundational resources (namespaces, RBAC, secrets)
- Platforms: Shared services (databases, caches, logging)
- Applications: Business logic (microservices, APIs)
- Monitoring: Observability (Prometheus, Grafana, logging)

## Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| Child app not created | Incorrect path or directory | Check source.path and directory.include |
| Sync order wrong | Missing sync-wave annotation | Add annotation with correct wave number |
| Resources in wrong order | No init containers | Add initContainers for dependencies |
| Delete fails | Missing finalizers | Add resources-finalizer annotation |
| OutOfSync after delete | Orphaned resources | Verify cascading delete |

## Monitoring Commands

```bash
# Watch all applications
watch 'argocd app list'

# Watch parent application
argocd app get root --refresh

# Check sync waves
kubectl get application -n argocd -o json | jq '.items[].metadata.annotations'

# View all resources
kubectl get all -A --sort-by=.metadata.namespace
```

## Cleanup Checklist

- [ ] Delete root application: `argocd app delete root --cascade=true`
- [ ] Verify children deleted: `argocd app list`
- [ ] Check namespaces removed: `kubectl get ns`
- [ ] Confirm all resources gone: `kubectl get all -A`
- [ ] Remove git repository: `rm -rf app-of-apps`
