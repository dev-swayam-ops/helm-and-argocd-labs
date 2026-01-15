# 09-app-of-apps-pattern

## What You'll Learn
- App-of-Apps architecture pattern
- Managing complex multi-tier applications
- Hierarchical application structures
- Dependency management between applications
- Application ordering and sequencing
- Scaling GitOps for large deployments
- Cluster bootstrapping patterns

## Prerequisites
- Completed modules 00-08
- ArgoCD applications knowledge
- Helm charts or Kustomize understanding
- Git repository structure knowledge
- Multi-environment deployment experience
- Understanding of application dependencies

## Key Concepts

**App-of-Apps Pattern:** Parent application manages child applications. Enables single source of truth. Scales to thousands of applications. Parent app references child app manifests.

**Hierarchical Architecture:** Parent → children → resources. Simplifies management of complex systems. Clear dependency structure. Easier to reason about deployments.

**Bootstrap Application:** First app deployed manually. Bootstraps all other applications. Single entry point to entire system. Applied to fresh cluster.

**Application Dependencies:** Child apps may depend on other apps. Ordering ensures resources created in correct sequence. Namespace dependencies. Health checks before child sync.

**Wave Metadata:** Sync waves control deployment order. Lower numbers deploy first. Same wave syncs in parallel. RBAC and secrets before applications.

**Templating:** Parent app uses Helm/Kustomize to generate child app manifests. Dynamic configuration. Environment-specific children. Reduced duplication.

## Hands-on Lab: Bootstrap System with App-of-Apps

**Objective:** Create parent app managing multiple child applications in hierarchical structure

**Steps:**

1. **Create app-of-apps repository structure:**
```bash
mkdir -p app-of-apps/{infrastructure,platforms,applications}

# Infrastructure apps (namespaces, RBAC, secrets)
mkdir -p app-of-apps/infrastructure

cat > app-of-apps/infrastructure/namespace.yaml << 'EOF'
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
EOF

# Platform apps (databases, logging)
mkdir -p app-of-apps/platforms

cat > app-of-apps/platforms/postgres.yaml << 'EOF'
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
EOF

# Application apps (business logic)
mkdir -p app-of-apps/applications

cat > app-of-apps/applications/backend.yaml << 'EOF'
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
      containers:
      - name: backend
        image: myapp:backend-1.0
        ports:
        - containerPort: 8080
EOF

git init
git add .
git commit -m "App-of-Apps structure"
```

2. **Create child applications for each layer:**
```bash
cat > app-of-apps/infrastructure-app.yaml << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: infrastructure
  namespace: argocd
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
EOF

cat > app-of-apps/platforms-app.yaml << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platforms
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/example/app-of-apps
    targetRevision: main
    path: platforms/
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF

cat > app-of-apps/applications-app.yaml << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: applications
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/example/app-of-apps
    targetRevision: main
    path: applications/
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
EOF
```

3. **Create parent app-of-apps:**
```bash
cat > app-of-apps/root-app.yaml << 'EOF'
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
EOF
```

4. **Deploy root application (bootstrap):**
```bash
kubectl apply -f app-of-apps/root-app.yaml

# Verify parent created
argocd app get root
# Output: Shows infrastructure, platforms, applications as children

# Wait for sync
argocd app wait root --sync

# Check namespaces created
kubectl get ns
# Output: myapp, monitoring, data created
```

5. **Verify hierarchical structure:**
```bash
# Check parent health
argocd app get root --refresh

# List all applications
argocd app list
# Output: root (parent), infrastructure, platforms, applications (children)

# Check specific child status
argocd app get infrastructure

# Verify resources created
kubectl get all -n myapp
kubectl get all -n data
```

## Validation

- [ ] Root application deployed successfully
- [ ] All child applications created and synced
- [ ] Infrastructure layer deployed first
- [ ] Platform layer deployed second
- [ ] Application layer deployed last
- [ ] All resources healthy and running
- [ ] Hierarchical structure visible in ArgoCD
- [ ] Deletion cascades properly

## Cleanup

```bash
# Delete root application (cascades to children)
argocd app delete root --cascade=true

# Wait for cleanup
kubectl wait --for=delete ns/myapp --timeout=60s

# Remove repository
rm -rf app-of-apps
```

## Common Mistakes

1. **Wrong directory structure** - Follow clear naming (infrastructure/platforms/applications)
2. **Missing finalizers** - Add `resources-finalizer.argocd.argoproj.io` for cleanup
3. **Circular dependencies** - Design acyclic dependency graph
4. **No sync waves** - Use metadata to control ordering
5. **Ignoring namespaces** - Plan namespace structure upfront
6. **Tight coupling** - Keep apps loosely coupled via ConfigMaps
7. **Forgotten cleanup** - Test cascading deletion before production

## Troubleshooting

**Child apps not appearing:** Check application YAML syntax. Verify directory structure. Ensure source path correct.

**Partial sync failures:** Check child logs: `argocd app logs root`. Verify namespace permissions. Check resource dependencies.

**Cascading delete not working:** Verify finalizers present. Check RBAC permissions. Remove orphaned application CRDs.

**Resources not in expected order:** Add sync waves to control ordering. Use health checks between waves.

**Parent app stuck:** Check for infinite loops or conflicts. Review controller logs. Delete and recreate if necessary.

## Next Steps

- Implement sync waves for complex ordering
- Add health checks and readiness gates
- Implement notification webhooks for parent app
- Create templated app-of-apps for multi-tenant
- Add image update automation
- Implement policy enforcement with ArgoCD projects
