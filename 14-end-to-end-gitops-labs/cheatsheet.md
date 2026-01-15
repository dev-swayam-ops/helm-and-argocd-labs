# 14-end-to-end-gitops-labs: Cheatsheet

## Complete GitOps Workflow

```
Code Change
    ↓
Commit to develop branch
    ↓
GitHub Actions: Build image, push registry
    ↓
Update values.yaml in Git
    ↓
ArgoCD watches develop branch
    ↓
Auto-sync to dev namespace
    ↓
Manual promotion to staging
    ↓
ArgoCD syncs staging namespace
    ↓
Approval + manual promotion to main
    ↓
ArgoCD triggers canary in prod
    ↓
Metrics analysis validates
    ↓
Gradual traffic shift to 100%
    ↓
Automatic rollback on failure
    ↓
Complete deployment
```

## Multi-Environment Configuration

| Environment | Auto-Sync | Branch | Replicas | Approval |
|-------------|-----------|--------|----------|----------|
| Dev | Yes | develop | 1 | None |
| Staging | No | staging | 2 | Dev Lead |
| Prod | No | main | 3 | Lead + Manager |

## Key Git Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `git checkout -b feature/name` | Create feature branch | Feature development |
| `git push origin feature/name` | Push to remote | CI/CD triggers |
| `git merge <branch>` | Merge changes | Promote between environments |
| `git tag v1.0.0` | Tag release | Mark production version |
| `git log --graph --oneline --all` | View history | Understand flow |

## ArgoCD Application Structure

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-env
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo
    targetRevision: branch-name
    path: helm/myapp
    helm:
      releaseName: myapp
      valuesFiles:
      - values-env.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: env
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Helm Value File Pattern

```yaml
# values.yaml - Base defaults
replicaCount: 2
image:
  tag: "1.0.0"

# values-dev.yaml - Dev overrides
replicaCount: 1
image:
  tag: "latest"

# values-prod.yaml - Prod overrides
replicaCount: 3
image:
  tag: "1.0.0"  # Specific version
resources:
  limits:
    memory: "512Mi"
```

## CI/CD Integration Commands

```bash
# Trigger build on push
git push origin develop

# Update image tag in values file
sed -i 's/tag: .*/tag: "'"${NEW_TAG}"'"/' values.yaml

# Commit updated values
git add values.yaml
git commit -m "Update image tag to ${NEW_TAG}"
git push origin develop

# ArgoCD auto-detects and syncs
argocd app get myapp-dev
```

## Monitoring and Validation

```bash
# Watch application status
argocd app watch myapp-dev

# Check rollout progress
kubectl rollout status rollout/myapp -n prod

# Verify metrics
kubectl get servicemonitor -A

# Check sync history
argocd app history myapp-prod

# View deployment events
kubectl get events -n prod --sort-by='.lastTimestamp'
```

## Progressive Delivery Setup

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    canary:
      steps:
      - setWeight: 10      # 10% traffic
      - pause: {duration: 5m}
      - setWeight: 50      # 50% traffic
      - pause: {duration: 5m}
      - setWeight: 100     # Full traffic
```

## Common Troubleshooting

| Issue | Command | Solution |
|-------|---------|----------|
| App not syncing | `argocd app get myapp` | Check repo URL, branch, credentials |
| Git push rejected | `git pull origin branch` | Pull latest before pushing |
| Image not updating | `kubectl set image` | Manually set image or update values |
| Rollout stuck | `kubectl describe rollout myapp` | Check metrics, pause, restart |
| Metrics missing | `kubectl get servicemonitor` | Create ServiceMonitor, verify scrape |

## Promotion Workflow

```bash
# Development (automatic)
git push origin develop
# ArgoCD auto-syncs to dev immediately

# Staging (manual, from develop)
git checkout staging && git merge develop && git push
argocd app sync myapp-staging

# Production (manual, from staging)
git checkout main && git merge staging && git push
argocd app sync myapp-prod
# Monitors canary deployment
```

## Environment Values Examples

**Dev (Fast iteration):**
```yaml
replicas: 1
pullPolicy: Always
resources:
  requests:
    memory: 64Mi
    cpu: 10m
```

**Prod (High availability):**
```yaml
replicas: 3
pullPolicy: IfNotPresent
resources:
  limits:
    memory: 512Mi
    cpu: 500m
affinity:
  podAntiAffinity:
    required
```

## Backup and Recovery

```bash
# Backup ArgoCD configuration
kubectl get applications -A -o yaml > backup-apps.yaml
kubectl get appprojects -A -o yaml > backup-projects.yaml

# Restore from backup
kubectl apply -f backup-apps.yaml
kubectl apply -f backup-projects.yaml

# Backup application state
kubectl get all -n prod -o yaml > backup-prod-state.yaml

# Restore application state
kubectl delete all -n prod
kubectl apply -f backup-prod-state.yaml
```

## Dashboard Commands

```bash
# Port forward to Grafana
kubectl port-forward -n monitoring svc/grafana 3000:80

# Access ArgoCD dashboard
kubectl port-forward -n argocd svc/argocd-server 8080:443

# Prometheus metrics
kubectl port-forward -n monitoring svc/prometheus 9090:9090

# View deployment history
kubectl rollout history deployment/myapp -n prod
```

## Best Practices Checklist

- [ ] Git as single source of truth
- [ ] Environment-specific value files
- [ ] Automated CI/CD pipeline
- [ ] Approval gates for production
- [ ] Monitoring and alerting configured
- [ ] Rollback procedures tested
- [ ] Disaster recovery documented
- [ ] RBAC and network policies enforced
- [ ] Secrets encrypted at rest and in transit
- [ ] Regular backup testing
