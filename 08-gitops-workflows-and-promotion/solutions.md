# 08-gitops-workflows-and-promotion: Solutions

## Exercise 1: Create Multi-Environment Git Repository Structure

**Solution:**
```bash
# Create directory structure
mkdir -p gitops-repo/{dev,staging,prod}
cd gitops-repo

# Create dev manifests
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
        image: myapp:1.0
        ports:
        - containerPort: 8080
EOF

# Create staging (2 replicas)
mkdir -p staging/myapp
cp dev/myapp/deployment.yaml staging/myapp/deployment.yaml
sed -i 's/namespace: dev/namespace: staging/g' staging/myapp/deployment.yaml
sed -i 's/replicas: 1/replicas: 2/g' staging/myapp/deployment.yaml

# Create prod (3 replicas, prod image)
mkdir -p prod/myapp
cp dev/myapp/deployment.yaml prod/myapp/deployment.yaml
sed -i 's/namespace: dev/namespace: prod/g' prod/myapp/deployment.yaml
sed -i 's/replicas: 1/replicas: 3/g' prod/myapp/deployment.yaml
sed -i 's/image: myapp:1.0/image: myapp:1.0-prod/g' prod/myapp/deployment.yaml

# Initialize Git
git init
git add .
git commit -m "Initial GitOps structure: dev, staging, prod environments"
```

**Explanation:** Separate paths allow environment-specific configurations. Different replicas for load handling. Image tags differ to ensure controlled rollout.

---

## Exercise 2: Configure Dev Environment with Auto-Sync

**Solution:**
```bash
# Create dev application with auto-sync
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
      prune: true      # Remove resources deleted from Git
      selfHeal: true   # Sync when cluster drifts
    syncOptions:
      - CreateNamespace=true
EOF

# Verify auto-sync enabled
argocd app get myapp-dev
# Output shows: Sync Policy: Automated, Prune: true, SelfHeal: true

# Test auto-sync by modifying Git
cd gitops-repo
sed -i 's/replicas: 1/replicas: 2/g' dev/myapp/deployment.yaml
git add .
git commit -m "Scale dev to 2 replicas"

# Wait for ArgoCD to detect (default 3 min), or refresh
argocd app get myapp-dev --refresh

# Verify pods scaled
kubectl get pods -n dev
# Output: 2 pods running
```

**Explanation:** Automated sync watches Git every 3 minutes by default. Prune removes resources not in Git. Self-healing immediately syncs cluster drift.

---

## Exercise 3: Configure Staging with Manual Sync

**Solution:**
```bash
# Create staging application with manual sync
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
    manual: {}    # No automatic sync
    syncOptions:
      - CreateNamespace=true
EOF

# Check status (OutOfSync initially)
argocd app get myapp-staging
# Output: Sync Status: OutOfSync

# Review changes before sync
argocd app diff myapp-staging
# Shows differences between Git and cluster

# Manually sync when ready
argocd app sync myapp-staging

# Wait for completion
argocd app wait myapp-staging --sync

# Check history
argocd app history myapp-staging
# Output: Shows manual sync operation with timestamp
```

**Explanation:** Manual sync requires explicit approval. Allows testing and review before production. Prevents accidental deployments.

---

## Exercise 4: Configure Production with Strict Controls

**Solution:**
```bash
# Create production application with strict controls
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
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
    manual: {}  # Require approval
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true      # Delete resources last
      - RespectIgnoreDifferences=true
EOF

# Verify finalizers
kubectl get application myapp-prod -n argocd -o yaml | grep finalizers

# Check manual sync only
argocd app get myapp-prod
# Output: Sync Policy: Manual

# Require policy review before sync
argocd app sync myapp-prod --dry-run
# Shows what would be deployed

# Verify actual sync
argocd app sync myapp-prod
argocd app wait myapp-prod --sync

# Check resource limits
kubectl describe ns prod
# Output: Resource quotas and limits
```

**Explanation:** Finalizers ensure controlled cleanup. Manual sync gates high-risk changes. PruneLast prevents accidental resource deletion. Dry-run validates before sync.

---

## Exercise 5: Implement Image Version Promotion

**Solution:**
```bash
# Set up image versions
cd gitops-repo

# Dev: latest development version
sed -i 's/image: myapp:1.0/image: myapp:latest/g' dev/myapp/deployment.yaml

# Staging: release candidate
sed -i 's/image: myapp:1.0/image: myapp:1.1-rc1/g' staging/myapp/deployment.yaml

# Prod: stable release
sed -i 's/image: myapp:1.0-prod/image: myapp:1.0/g' prod/myapp/deployment.yaml

git add .
git commit -m "Update image versions per environment"

# Verify versions
argocd app get myapp-dev --refresh
argocd app get myapp-staging --refresh
argocd app get myapp-prod --refresh

# Check deployed images
kubectl get deployment -n dev -o jsonpath='{.items[0].spec.template.spec.containers[0].image}'
# Output: myapp:latest

kubectl get deployment -n staging -o jsonpath='{.items[0].spec.template.spec.containers[0].image}'
# Output: myapp:1.1-rc1

kubectl get deployment -n prod -o jsonpath='{.items[0].spec.template.spec.containers[0].image}'
# Output: myapp:1.0
```

**Explanation:** Different image versions per environment. Dev uses latest for testing. Staging uses RC (release candidate). Prod uses stable version. Promotes quality through pipeline.

---

## Exercise 6: Create GitOps Promotion Pipeline

**Solution:**
```bash
# Set up branch protection (GitHub example)
# Requires: GitHub CLI installed

# Create promotion branch structure
cd gitops-repo
git branch staging
git branch prod

# Create promotion workflow (GitHub Actions)
mkdir -p .github/workflows

cat > .github/workflows/promote.yaml << 'EOF'
name: Promote to Environment
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - staging
          - prod

jobs:
  promote:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Merge to ${{ github.event.inputs.environment }}
        run: |
          git config user.name "GitOps Bot"
          git config user.email "gitops@example.com"
          git checkout ${{ github.event.inputs.environment }}
          git merge main --no-edit
          git push origin ${{ github.event.inputs.environment }}
EOF

git add .
git commit -m "Add promotion workflow"

# Verify workflow
git log --oneline
# Shows commits with promotion metadata
```

**Explanation:** Promotion workflow enforces review before merge. Separate branches for each environment. Automated merge ensures consistency. Audit trail for compliance.

---

## Exercise 7: Implement Rollback Strategy

**Solution:**
```bash
# Create release tags
cd gitops-repo

# Tag current version as release
git tag -a v1.0 -m "Production release v1.0"
git push origin v1.0

# Create rollback branch with previous version
git checkout -b rollback/v0.9
git log --oneline

# Checkout previous version
git revert HEAD
git push origin rollback/v0.9

# To rollback to v0.9
git checkout main
git merge rollback/v0.9
git push origin main

# ArgoCD will auto-detect (if auto-sync enabled) or manual sync
argocd app sync myapp-prod
argocd app wait myapp-prod --sync

# Verify rollback
kubectl get deployment -n prod -o jsonpath='{.items[0].spec.template.spec.containers[0].image}'
# Output: myapp:0.9 (rolled back version)

# Check history
argocd app history myapp-prod
# Shows all operations including rollback
```

**Explanation:** Git tags mark releases for easy reference. Rollback branches restore previous state. Commits provide audit trail. Quick revert on failures.

---

## Exercise 8: Monitor Promotion Progress

**Solution:**
```bash
# Check all environments
argocd app list
# Output: Shows myapp-dev, myapp-staging, myapp-prod status

# Detailed status per environment
for env in dev staging prod; do
  echo "=== $env ==="
  argocd app get myapp-$env
  # Shows: Health, Sync Status, Last Sync
done

# View operation history
argocd app history myapp-dev
argocd app history myapp-staging
argocd app history myapp-prod

# Watch resource health during promotion
kubectl get deployment -n dev -w
kubectl get deployment -n staging -w
kubectl get deployment -n prod -w

# Check logs during sync
argocd app logs myapp-staging
# Shows real-time reconciliation logs

# Create monitoring dashboard (Prometheus metrics)
kubectl exec -n argocd deployment/argocd-server -- \
  curl -s http://localhost:8082/metrics | grep argocd_app
```

**Explanation:** List shows overall status. History tracks operations. Watch commands monitor real-time. Logs provide detailed troubleshooting. Metrics enable monitoring.

---

## Exercise 9: Implement Approval Gates

**Solution:**
```bash
# Manual approval workflow
# Set manual sync to require review

# GitHub Code Owners file (CODEOWNERS)
mkdir -p .github
cat > .github/CODEOWNERS << 'EOF'
# Production changes require approval
prod/ @devops-team

# Staging changes require single approval
staging/ @team
EOF

git add .
git commit -m "Add code owners for approval gates"

# Create pull request template
cat > .github/PULL_REQUEST_TEMPLATE.md << 'EOF'
## Description
Describe the changes being promoted

## Environment
- [ ] Dev
- [ ] Staging
- [ ] Production

## Testing
- [ ] Tested in lower environment
- [ ] No regressions
- [ ] Performance verified

## Approval
- Requires approval from code owners
- Wait for CI/CD pipeline
- Manual sync confirmation
EOF

git add .
git commit -m "Add PR template for promotion tracking"

# Set up notification webhook (Slack example)
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  template.promotion-pending: |
    message: "Promotion to {{.app.metadata.name}} pending approval"
EOF

# Require manual sync for prod
# Already configured with manual syncPolicy
```

**Explanation:** CODEOWNERS enforces review requirements. PR templates standardize promotion info. Notifications alert stakeholders. Manual sync gates prevent auto-deployment.

---

## Exercise 10: Troubleshoot Promotion Failures

**Solution:**
```bash
# Common issue: OutOfSync status
argocd app get myapp-staging
# Check if repo is accessible
argocd repo get https://github.com/example/gitops-repo

# Fix: Update repo cache
argocd repo update https://github.com/example/gitops-repo

# Issue: Sync fails
argocd app info myapp-staging
# Check detailed error messages

# Review logs
kubectl logs -n argocd statefulset/argocd-application-controller | tail -100
kubectl logs -n argocd deployment/argocd-repo-server | tail -100

# Check Git connectivity
argocd app get myapp-staging --refresh

# Verify RBAC permissions
kubectl auth can-i create deployments \
  --as system:serviceaccount:argocd:argocd-server \
  -n staging

# Dry-run to validate manifests
argocd app manifests myapp-staging | kubectl apply -f - --dry-run=client

# If stuck, force sync
argocd app sync myapp-staging --force

# Clear repo cache if needed
kubectl rollout restart deployment/argocd-repo-server -n argocd
```

**Explanation:** OutOfSync typically means connectivity issues. Logs show detailed errors. RBAC affects sync ability. Dry-run validates before sync. Force sync clears stuck states.
