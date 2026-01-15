# 07-argocd-applications-and-sync

## What You'll Learn
- Create and configure ArgoCD applications
- Manage sync policies and strategies
- Implement automatic and manual synchronization
- Use application destinations and sources
- Configure notification and webhook integrations
- Manage application lifecycle (create, update, delete)
- Monitor and troubleshoot application syncs

## Prerequisites
- Completed 06-argocd-basics module
- ArgoCD installed and running
- Git repository set up and connected
- Helm charts prepared for deployment
- Understanding of Kubernetes manifests

## Key Concepts

### Application Resource
ArgoCD Custom Resource defining what to deploy, where, and how to keep it synced.

### Sync Policy
Rules governing how and when ArgoCD synchronizes application state with Git.

### Automated Sync
ArgoCD automatically applies changes from Git without manual intervention.

### Manual Sync
Admin-controlled sync allowing review before applying changes.

### Self-Healing
Automatic sync when cluster state drifts from Git even without Git changes.

## Hands-on Lab: Application Sync Strategies

### Step 1: Create Base Application
```bash
# Create application with basic settings
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 15.0.0
    helm:
      values: |
        replicaCount: 2
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx-app
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
EOF

# Verify application
argocd app get nginx-app
kubectl get application -n argocd
```

### Step 2: Implement Automated Sync
```bash
# Update with automated sync policy
kubectl patch application nginx-app -n argocd \
  -p '{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}' \
  --type merge

# Verify automation enabled
argocd app get nginx-app | grep -A5 "Sync Policy"
```

**Expected Output:**
```
Sync Policy:     Automated
  Prune:         true
  Self Heal:     true
```

### Step 3: Monitor Sync Status
```bash
# Check application status
argocd app get nginx-app
argocd app info nginx-app

# Watch real-time status
argocd app wait nginx-app --health

# Check pod health
kubectl get pods -n nginx-app
```

### Step 4: Manual Sync
```bash
# Perform manual sync
argocd app sync nginx-app

# Wait for sync to complete
argocd app wait nginx-app --sync

# View sync history
argocd app history nginx-app
```

**Expected Output:**
```
ID     DATE                           REVISION
0      2024-01-15 10:30:45 +0000      15.0.0
```

### Step 5: Configure Notifications
```bash
# Create notification configuration
cat > argocd-notifications.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  trigger.on-health-degraded: |
    - when: app.status.health.status == 'Degraded'
      send: [app-health-degraded]
  trigger.on-sync-failed: |
    - when: app.status.operationState.phase in ['Error', 'Failed']
      send: [app-sync-failed]
EOF

kubectl apply -f argocd-notifications.yaml
```

## Validation
- Application is created and visible in ArgoCD
- Sync status shows Synced or Syncing
- Health status indicates Healthy
- Automated policies are configured correctly
- Manual sync completes successfully

## Cleanup
```bash
# Delete application
kubectl delete application nginx-app -n argocd

# Remove namespace
kubectl delete namespace nginx-app

# Remove notifications config
kubectl delete configmap argocd-notifications-cm -n argocd
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Application won't sync | Check source path and Git repository access |
| Health status unknown | Wait for resources to stabilize |
| Manual sync not working | Verify RBAC permissions |
| Notifications not firing | Check trigger conditions and destinations |
| Prune deletes critical resources | Use syncOptions.RespectIgnoreDifferences |

## Troubleshooting

**Problem:** "Unable to compare source and destination state"
```bash
# Solution: Check application logs
argocd app logs nginx-app
kubectl logs -n argocd deployment/argocd-repo-server
```

**Problem:** Sync fails with permission errors
```bash
# Solution: Verify RBAC
argocd account list
argocd account get-user-info
```

**Problem:** Application shows OutOfSync but nothing changed
```bash
# Solution: Refresh and hard sync
argocd app get nginx-app --refresh
argocd app sync nginx-app --force
```

## Next Steps
- Explore **08-gitops-workflows-and-promotion** for advanced patterns
- Implement multi-environment deployments
- Set up progressive delivery with Argo Rollouts
- Configure SSO and advanced RBAC for ArgoCD
