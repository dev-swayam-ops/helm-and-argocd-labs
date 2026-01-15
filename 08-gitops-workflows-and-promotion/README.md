# 08-gitops-workflows-and-promotion

## What You'll Learn
- GitOps principles and best practices
- Multi-environment promotion workflows
- Infrastructure as Code (IaC) patterns
- Automated deployment pipelines with ArgoCD
- Approval and gating mechanisms
- Progressive delivery strategies
- GitOps security and governance

## Prerequisites
- Completed modules 00-07
- Basic Git workflow knowledge (branches, commits, pull requests)
- Understanding of ArgoCD applications
- Helm chart knowledge
- kubectl basics
- Docker and Kubernetes fundamentals

## Key Concepts

**GitOps Workflow:** Version control drives infrastructure state. All changes tracked in Git. ArgoCD continuously reconciles cluster with Git source.

**Promotion Pipeline:** dev → staging → production. Changes tested in lower environments before production. Approvals gate transitions between environments.

**Repository Structure:** Separate branches or paths for each environment. Source of truth in Git. All deployments via Git updates.

**Deployment Strategies:**
- Blue-Green: Two identical production environments
- Canary: Gradual rollout with traffic splitting
- Rolling: Sequential pod replacement
- Shadow: Test version runs parallel to production

**Image Updates:** Automated or manual updates to image tags. GitOps tools update manifest. Source control triggers deployment.

**Application Synchronization:** ArgoCD watches Git. Manual or automatic deployment on changes. Drift detection and remediation.

## Hands-on Lab: Complete GitOps Promotion Workflow

**Objective:** Set up dev → staging → prod promotion with ArgoCD applications

**Steps:**

1. **Create promotion repository structure:**
```bash
# Create Git repo with environment structure
mkdir -p gitops-repo/{dev,staging,prod}
cd gitops-repo

# Create app manifests for dev
mkdir -p dev/myapp
cat > dev/myapp/deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.19
        ports:
        - containerPort: 80
EOF

# Create staging manifests
mkdir -p staging/myapp
cp dev/myapp/deployment.yaml staging/myapp/
sed -i 's/namespace: dev/namespace: staging/g' staging/myapp/deployment.yaml
sed -i 's/replicas: 1/replicas: 2/g' staging/myapp/deployment.yaml

# Create prod manifests
mkdir -p prod/myapp
cp dev/myapp/deployment.yaml prod/myapp/
sed -i 's/namespace: dev/namespace: prod/g' prod/myapp/deployment.yaml
sed -i 's/replicas: 1/replicas: 3/g' prod/myapp/deployment.yaml
sed -i 's/image: nginx:1.19/image: nginx:latest/g' prod/myapp/deployment.yaml

git init
git add .
git commit -m "Initial GitOps structure"
```

2. **Create ArgoCD applications for each environment:**
```bash
# Dev application (auto-sync)
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/gitops-repo
    targetRevision: main
    path: dev/myapp
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF

# Staging application
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-staging
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/gitops-repo
    targetRevision: main
    path: staging/myapp
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  syncPolicy:
    manual: {}
    syncOptions:
      - CreateNamespace=true
EOF

# Production application (manual)
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/gitops-repo
    targetRevision: main
    path: prod/myapp
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    manual: {}
    syncOptions:
      - CreateNamespace=true
EOF
```

3. **Verify all applications:**
```bash
argocd app list
# Output: Shows myapp-dev, myapp-staging, myapp-prod

argocd app get myapp-dev --refresh
# Output: Health: Healthy, Sync Status: Synced

kubectl get pods -n dev -n staging -n prod
# Output: Pods running in all three namespaces
```

## Validation

- [ ] Dev, staging, prod namespaces created
- [ ] All three applications synced successfully
- [ ] Pods running in each environment
- [ ] Image versions differ by environment
- [ ] Auto-sync works for dev
- [ ] Manual sync works for staging/prod
- [ ] ArgoCD shows healthy status

## Cleanup

```bash
# Delete applications
argocd app delete myapp-dev myapp-staging myapp-prod

# Delete namespaces
kubectl delete ns dev staging prod

# Remove Git repo
rm -rf gitops-repo
```

## Common Mistakes

1. **Same image version in all environments** - Use different versions for testing
2. **Forgetting to sync after Git change** - Enable automated sync for lower environments
3. **Mixing environments in one Git branch** - Keep separate paths or branches
4. **No approval gates** - Manual sync prevents accidental production changes
5. **Lost Git history** - Commit every change, never force-push
6. **No rollback strategy** - Tag releases, maintain previous manifests
7. **Credentials in Git** - Use sealed secrets or external secret managers

## Troubleshooting

**Applications show OutOfSync:** Check Git connectivity and repo URL. Use `argocd app get <app> --refresh`.

**Sync fails in production:** Verify permissions. Check ArgoCD RBAC. Review controller logs: `kubectl logs -n argocd statefulset/argocd-application-controller`.

**Image not updating:** Verify image tag in Git. Rebuild and push image. Trigger refresh: `argocd app get <app> --hard-refresh`.

**Different behavior between environments:** Check values/overrides. Verify namespace differences. Review ConfigMaps and secrets.

## Next Steps

- Implement approval workflows with GitHub Actions or GitLab CI
- Add image scanning and security gates
- Set up automatic rollback on failure
- Implement Helm chart promotion patterns
- Explore app-of-apps pattern for complex promotion
- Add monitoring and observability for deployments
