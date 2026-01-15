# 07-argocd-applications-and-sync: Solutions

## Exercise 1: Create Application from Git Repository

**Solution:**
```bash
# Create application with Git source
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: git-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/k8s-manifests
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: git-app
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
EOF

# Verify creation
argocd app get git-app

# Check sync status
argocd app info git-app
argocd app get git-app --refresh

# View deployed resources
kubectl get all -n git-app

# Check specific paths in repo
argocd app get git-app | grep Path
```

**Explanation:** Git source requires repository URL and path. targetRevision specifies branch/tag. ArgoCD reconciles deployed resources with Git.

---

## Exercise 2: Create Application from Helm Chart

**Solution:**
```bash
# Helm chart application
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helm-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 15.0.0
    helm:
      values: |
        replicaCount: 3
        service:
          type: LoadBalancer
  destination:
    server: https://kubernetes.default.svc
    namespace: helm-app
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
EOF

# With values file from Git
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helm-git-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/helm-values
    targetRevision: main
    path: charts/
    helm:
      valuesFiles:
        - values.yaml
        - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: helm-app
EOF

# Verify
argocd app get helm-app
argocd app info helm-app

# Check rendered values
argocd app manifests helm-app | head -50
```

**Explanation:** Helm source uses chart repository. Values can be inline or from Git files. targetRevision for charts specifies semantic version.

---

## Exercise 3: Implement Automated Sync Policy

**Solution:**
```bash
# Create with automated sync
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: auto-sync-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: auto-sync
  syncPolicy:
    automated:
      prune: true      # Delete resources not in Git
      selfHeal: true   # Sync when cluster drifts
    syncOptions:
      - CreateNamespace=true
EOF

# Modify Git and observe auto-sync
# Edit manifest in Git repo
# Wait 3 minutes for ArgoCD to check (default interval)

# Or trigger refresh
argocd app get auto-sync-app --refresh
argocd app wait auto-sync-app --sync

# Check history
argocd app history auto-sync-app
# Shows automatic sync operations
```

**Explanation:** Automated sync watches Git. Prune removes resources deleted from Git. Self-healing re-syncs cluster drift immediately.

---

## Exercise 4: Configure Manual Sync Policy

**Solution:**
```bash
# Manual sync application
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: manual-sync-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: manual
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    # No automated sync = manual only
EOF

# Check status (OutOfSync initially)
argocd app get manual-sync-app
# Status: OutOfSync

# Manually sync when ready
argocd app sync manual-sync-app

# Wait for completion
argocd app wait manual-sync-app --sync

# View history
argocd app history manual-sync-app
# Shows manual sync operation

# Modify Git
# Status becomes OutOfSync again
# Requires manual sync to apply
```

**Explanation:** Manual sync requires explicit approval. Useful for production. Review changes before applying.

---

## Exercise 5: Manage Multiple Sync Policies

**Solution:**
```bash
# Application with advanced sync options
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: advanced-sync-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: advanced
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=background
      - RespectIgnoreDifferences=true
      - Validate=false
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
EOF

# With source ignore
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ignore-diff-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: ignore
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
EOF

# Verify options applied
argocd app get advanced-sync-app
```

**Explanation:** Sync options control behavior. RespectIgnoreDifferences allows cluster changes. Retry policies handle transient failures.

---

## Exercise 6: Implement Destination Cluster Configuration

**Solution:**
```bash
# Register additional cluster
argocd cluster add <CONTEXT_NAME>

# Create application for different cluster
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: multi-cluster-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
  destination:
    server: https://my-remote-cluster-api:6443
    namespace: multi-cluster
EOF

# Use local cluster shorthand
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: local-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    targetRevision: main
    path: manifests/
  destination:
    server: https://kubernetes.default.svc  # Local cluster
    namespace: local
EOF

# List clusters
argocd cluster list

# Get cluster info
argocd cluster get https://kubernetes.default.svc
```

**Explanation:** Multiple clusters can be registered. Each application targets specific cluster. Local cluster uses default service.

---

## Exercise 7: Monitor and Analyze Sync Status

**Solution:**
```bash
# Get detailed sync status
argocd app get nginx-app
# Shows: Status, Health, Sync Status, Details

# Refresh from source
argocd app get nginx-app --refresh

# View operation details
argocd app info nginx-app

# Monitor sync progress
argocd app wait nginx-app --sync

# Check resource sync status
argocd app get nginx-app
# Each resource shows: Namespace, Name, Kind, Status, Health

# View full history
argocd app history nginx-app
# Shows: ID, DATE, REVISION, AUTHOR, STATUS

# Get logs during sync
argocd app logs nginx-app
kubectl logs -n argocd statefulset/argocd-application-controller | grep nginx-app

# Compare Git vs cluster
argocd app diff nginx-app
# Shows differences between Git and deployed resources
```

**Explanation:** Sync status shows overall state. Resource view shows individual component status. Logs show detailed operations.

---

## Exercise 8: Configure Notification and Webhook Triggers

**Solution:**
```bash
# Install notifications controller
kubectl apply -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/manifests/install.yaml

# Configure triggers
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
      send: [sync-succeeded]
  trigger.on-sync-failed: |
    - when: app.status.operationState.phase in ['Error', 'Failed']
      send: [sync-failed]
  trigger.on-health-degraded: |
    - when: app.status.health.status == 'Degraded'
      send: [health-degraded]
EOF

# Configure notifications (Slack example)
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  template.sync-succeeded: |
    message: "App {{.app.metadata.name}} sync succeeded"
EOF

# Subscribe application to notifications
kubectl patch application my-app -n argocd -p \
  '{"metadata":{"annotations":{"notifications.argoproj.io/subscribe.on-sync-succeeded":"slack:my-channel"}}}' \
  --type merge
```

**Explanation:** Notifications controller watches applications. Triggers fire on events. Multiple notification backends supported.

---

## Exercise 9: Manage Application Lifecycle Operations

**Solution:**
```bash
# Create application
kubectl apply -f app.yaml -n argocd
argocd app get my-app

# Update application configuration
kubectl patch application my-app -n argocd \
  -p '{"spec":{"source":{"targetRevision":"v2.0"}}}' \
  --type merge

# Perform upgrade with sync
argocd app sync my-app
argocd app wait my-app --sync

# Check update in history
argocd app history my-app

# Manage deletion
kubectl delete application my-app -n argocd

# Cascading delete (delete app and resources)
kubectl delete application my-app -n argocd --cascade=foreground

# Cleanup resources
argocd app sync my-app --prune

# Verify deletion
kubectl get application -n argocd | grep my-app
# Should be gone
```

**Explanation:** Application CRUD operations same as Kubernetes resources. Cascading delete removes deployed resources. History tracks all changes.

---

## Exercise 10: Troubleshoot Application Sync Issues

**Solution:**
```bash
# Check application status
argocd app get my-app

# If OutOfSync, refresh and check details
argocd app get my-app --refresh

# View detailed information
argocd app info my-app

# Check logs
argocd app logs my-app

# Repo-server logs (Git/Helm issues)
kubectl logs -n argocd deployment/argocd-repo-server | tail -50

# Application controller logs (reconciliation issues)
kubectl logs -n argocd statefulset/argocd-application-controller | tail -50

# Common issues and fixes:

# 1. Source not accessible
argocd repo get https://github.com/example/repo
argocd repo update https://github.com/example/repo

# 2. Destination unreachable
argocd cluster get https://kubernetes.default.svc

# 3. Permission denied
kubectl auth can-i create deployments --as system:serviceaccount:argocd:argocd-server

# 4. Chart not found
argocd app info my-app | grep -i chart

# 5. Values syntax error
helm template -f values.yaml chart/

# Force sync if stuck
argocd app sync my-app --force

# Clear server-side cache
kubectl rollout restart deployment/argocd-repo-server -n argocd
```

**Explanation:** Logs provide detailed error information. Cluster/repo connectivity must be verified. Permissions affect sync operations.
