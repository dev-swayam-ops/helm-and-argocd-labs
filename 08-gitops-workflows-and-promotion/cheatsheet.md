# 08-gitops-workflows-and-promotion: Cheatsheet

## Git Repository Structure

```
gitops-repo/
├── dev/
│   └── myapp/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── kustomization.yaml
├── staging/
│   └── myapp/
│       ├── deployment.yaml
│       └── service.yaml
└── prod/
    └── myapp/
        ├── deployment.yaml
        └── service.yaml
```

## Application Configuration

| Component | Dev | Staging | Prod |
|-----------|-----|---------|------|
| Sync Policy | Auto | Manual | Manual |
| Replicas | 1 | 2 | 3 |
| Image | latest | rc | stable |
| Approval | None | Review | Strict |

## ArgoCD Sync Policy Examples

### Auto-Sync (Development)
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
```

### Manual-Sync (Production)
```yaml
syncPolicy:
  manual: {}
  syncOptions:
    - CreateNamespace=true
    - PruneLast=true
```

## Promotion Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `git tag <name>` | Mark release | `git tag v1.0` |
| `git checkout -b <branch>` | Create env branch | `git checkout -b staging` |
| `argocd app sync <app>` | Manual sync | `argocd app sync myapp-staging` |
| `argocd app wait <app> --sync` | Wait for sync | Blocks until complete |
| `argocd app history <app>` | Sync history | Shows operation timeline |
| `argocd app diff <app>` | Git vs cluster diff | Identifies changes |

## Image Promotion Pattern

```
Dev:     nginx:latest
↓
Staging: nginx:1.20-rc1
↓
Prod:    nginx:1.20
```

## Rollback Commands

| Command | Purpose |
|---------|---------|
| `git revert HEAD` | Undo last commit |
| `git checkout <tag>` | Go to tagged version |
| `git merge <branch>` | Merge rollback changes |
| `argocd app sync <app>` | Sync rolled-back manifest |

## Multi-Environment Workflow

```
1. Dev (Auto-sync)
   ↓ (Testing & Validation)
2. Staging (Manual, Review)
   ↓ (Performance & Integration Testing)
3. Prod (Manual, Strict)
```

## Branch Protection Rules

```bash
# Main branch: require approval before merge
# Staging branch: require CI passes
# Prod branch: require 2 approvals + CI passes
```

## Common Sync Policies

| Type | Auto-Sync | Prune | SelfHeal | Use Case |
|------|-----------|-------|----------|----------|
| Dev | Yes | Yes | Yes | Rapid iteration |
| Staging | No | Yes | No | Controlled testing |
| Prod | No | No | No | Maximum safety |

## GitOps Best Practices

| Practice | Description |
|----------|-------------|
| Single Source of Truth | All state in Git |
| Declarative Manifests | YAML files in repo |
| Commit History | Audit trail of changes |
| Pull Requests | Code review before merge |
| Automated Tests | CI pipeline validation |
| Promotion Gates | Manual approval for prod |

## Monitoring Commands

```bash
# Watch all applications
argocd app list --watch

# Check specific app status
argocd app get myapp-dev --refresh

# View sync operation logs
argocd app logs myapp-staging

# Watch deployment progress
kubectl get deployment -w -n staging

# Check resource health
kubectl get all -n prod
```

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| OutOfSync | `argocd app get <app> --refresh` |
| Sync fails | `argocd app logs <app>` |
| Permissions | `kubectl auth can-i create deployments --as <sa>` |
| Manifest errors | `argocd app manifests <app> \| kubectl apply --dry-run=client` |

## Environment-Specific Overrides

```yaml
# Base (dev/myapp/deployment.yaml)
spec:
  replicas: 1

# Staging (staging/myapp/kustomization.yaml)
replicas:
  - name: myapp
    count: 2

# Prod (prod/myapp/kustomization.yaml)
replicas:
  - name: myapp
    count: 3
```

## Promotion Checklist

- [ ] Changes tested in dev
- [ ] Staging: Code review approved
- [ ] Staging: Tests pass
- [ ] Prod: Security scan passed
- [ ] Prod: Performance validated
- [ ] Prod: Approval from tech lead
- [ ] Git: Release tagged
- [ ] Monitoring: Dashboards ready
- [ ] Runbook: Rollback plan documented
