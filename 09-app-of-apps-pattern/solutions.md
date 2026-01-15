# 09-app-of-apps-pattern: Solutions

## Exercise 1: Create Multi-Tier Directory Structure

**Solution:**
```bash
# Create repository structure
mkdir -p app-of-apps/{infrastructure,platforms,applications}
cd app-of-apps

# Infrastructure layer
mkdir -p infrastructure/{namespaces,rbac,secrets}

cat > infrastructure/namespaces/namespace.yaml << 'EOF'
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

cat > infrastructure/rbac/role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
  namespace: myapp
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list"]
EOF

# Platforms layer
mkdir -p platforms/{postgres,redis,logging}

cat > platforms/postgres/postgres.yaml << 'EOF'
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

# Applications layer
mkdir -p applications/{frontend,backend,api}

cat > applications/backend/backend.yaml << 'EOF'
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

# Initialize Git
git init
git add .
git commit -m "Initial app-of-apps structure with three layers"

# Verify structure
tree
# Shows: infrastructure, platforms, applications with subdirectories
```

**Explanation:** Three-tier structure separates concerns. Infrastructure first, then platforms, then applications. Clear naming conventions enable discovery.

---

## Exercise 2: Create Infrastructure Applications

**Solution:**
```bash
# Create infrastructure application
cat > infrastructure/kustomization.yaml << 'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespaces/namespace.yaml
  - rbac/role.yaml
EOF

# Create child application
cat > infrastructure-app.yaml << 'EOF'
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
      - Validate=false
EOF

# Deploy infrastructure application
kubectl apply -f infrastructure-app.yaml

# Verify infrastructure created
argocd app get infrastructure
# Output: Health: Healthy, Sync: Synced

# Check namespaces created
kubectl get ns
# Output: myapp, monitoring namespaces exist

# Check RBAC created
kubectl get role -n myapp
# Output: app-reader role exists
```

**Explanation:** Infrastructure layer deploys first. Creates namespaces and RBAC. Foundation for all other applications. Automated sync ensures consistent state.

---

## Exercise 3: Create Platform Applications

**Solution:**
```bash
# Create platforms kustomization
cat > platforms/kustomization.yaml << 'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - postgres/postgres.yaml
EOF

# Create platforms application
cat > platforms-app.yaml << 'EOF'
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

# Deploy platforms application
kubectl apply -f platforms-app.yaml

# Verify deployment
argocd app get platforms
# Output: Health: Healthy, Sync: Synced

# Check PostgreSQL running
kubectl get pods -n data -l app=postgres
# Output: postgres pod running

# Verify service accessible
kubectl get svc -n data
# Output: postgres service exists with ClusterIP
```

**Explanation:** Platform layer deploys shared services. Depends on infrastructure layer completing first. Databases and caches available to applications.

---

## Exercise 4: Create Business Application Layer

**Solution:**
```bash
# Create applications kustomization
cat > applications/kustomization.yaml << 'EOF'
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - backend/backend.yaml
EOF

# Create backend service
cat > applications/backend/service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: myapp
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
EOF

# Create applications application
cat > applications-app.yaml << 'EOF'
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

# Deploy applications
kubectl apply -f applications-app.yaml

# Verify deployment
argocd app get applications

# Check backend deployment
kubectl get pods -n myapp -l app=backend
# Output: 2 backend pods running

# Verify service
kubectl get svc -n myapp
# Output: backend service available
```

**Explanation:** Application layer deployed last after infrastructure and platforms. Applications connect to databases and services. Complete system operational.

---

## Exercise 5: Implement Parent Application (Bootstrap)

**Solution:**
```bash
# Create parent app-of-apps
cat > root-app.yaml << 'EOF'
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
      include: "*-app.yaml"
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

# Bootstrap entire system with single command
kubectl apply -f root-app.yaml

# Watch parent creation (shows child apps being created)
argocd app get root --refresh
# Output: infrastructure, platforms, applications as children

# Wait for all to sync
argocd app wait root --sync

# Verify hierarchical structure
argocd app list
# Output: root (parent), infrastructure, platforms, applications (children)

# Check ArgoCD UI shows tree structure
# UI shows: root -> [infrastructure, platforms, applications]
```

**Explanation:** Root application bootstraps entire system. Single apply creates all layers. Hierarchical structure visible in ArgoCD. Clean separation of concerns.

---

## Exercise 6: Configure Sync Waves

**Solution:**
```bash
# Add sync waves to control ordering
cat > infrastructure-app.yaml << 'EOF'
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
EOF

cat > platforms-app.yaml << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platforms
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "1"
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
EOF

cat > applications-app.yaml << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: applications
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "2"
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
EOF

# Deploy with waves
kubectl apply -f *-app.yaml

# Watch sync order
argocd app get root --refresh
# Shows: Infrastructure (wave 0) → Platforms (wave 1) → Applications (wave 2)

# Verify ordering
kubectl get events -n argocd | grep sync
# Shows operations in sequential order
```

**Explanation:** Sync waves enforce deployment order. Wave 0 (infrastructure) deploys first. Wave 1 (platforms) waits for wave 0 complete. Wave 2 (applications) waits for wave 1 complete. Prevents resource conflicts.

---

## Exercise 7: Add Health Checks and Dependencies

**Solution:**
```bash
# Add readiness checks to services
cat > platforms/postgres/postgres.yaml << 'EOF'
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
            command:
            - /bin/sh
            - -c
            - "pg_isready -U postgres"
          initialDelaySeconds: 10
          periodSeconds: 5
EOF

# Add wait condition to applications
cat > applications/backend/backend.yaml << 'EOF'
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
        command: ['sh', '-c', 'until nc -z postgres.data 5432; do echo waiting for postgres; sleep 2; done']
      containers:
      - name: backend
        image: myapp:backend-1.0
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
EOF

# Deploy with health checks
kubectl apply -f root-app.yaml

# Watch deployment order
argocd app wait root --sync

# Verify health checks work
kubectl get pods -n data
# Shows postgres pod with ready status

kubectl describe pod -n myapp -l app=backend
# Shows init container: wait-for-db completed
# Shows readiness probe: passing
```

**Explanation:** Readiness probes determine when service is ready. Init containers wait for dependencies. Ensures correct startup order. Applications only start when dependencies healthy.

---

## Exercise 8: Implement Cascading Deletion

**Solution:**
```bash
# Verify finalizers set for cascading deletion
kubectl get application root -n argocd -o yaml | grep finalizers
# Output: resources-finalizer.argocd.argoproj.io

# Delete root application with cascading delete
argocd app delete root --cascade=true

# Watch deletion cascade through hierarchy
watch 'kubectl get application -n argocd'
# Shows: root deleted → infrastructure, platforms, applications deleted

# Verify all resources cleaned up
kubectl get ns
# Output: Infrastructure namespaces removed

# Check all child applications gone
argocd app list
# Output: No applications remaining

# Verify cleanup
kubectl get all -A
# Output: Only system namespaces remain
```

**Explanation:** Finalizers control deletion order. Cascade=true deletes child resources. Proper cleanup prevents orphaned resources. Test before production use.

---

## Exercise 9: Create Templated App-of-Apps

**Solution:**
```bash
# Create Helm chart for app-of-apps
mkdir -p app-of-apps-helm/{templates,charts}
cd app-of-apps-helm

cat > Chart.yaml << 'EOF'
apiVersion: v2
name: app-of-apps
description: Templated app-of-apps pattern
version: 1.0.0
EOF

cat > values.yaml << 'EOF'
environment: dev
children:
  - name: infrastructure
    wave: 0
  - name: platforms
    wave: 1
  - name: applications
    wave: 2
repoURL: https://github.com/example/app-of-apps
targetRevision: main
EOF

cat > values-prod.yaml << 'EOF'
environment: prod
children:
  - name: infrastructure
    wave: 0
  - name: platforms
    wave: 1
  - name: applications
    wave: 2
  - name: monitoring
    wave: 3
repoURL: https://github.com/example/app-of-apps
targetRevision: release-1.0
EOF

# Create template for child applications
cat > templates/child-app.yaml << 'EOF'
{{- range .Values.children }}
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: {{ .name }}
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "{{ .wave }}"
spec:
  project: default
  source:
    repoURL: {{ .Values.repoURL }}
    targetRevision: {{ .Values.targetRevision }}
    path: {{ .name }}/
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
{{- end }}
EOF

# Generate manifests
helm template app-of-apps . > manifests-dev.yaml
helm template app-of-apps . -f values-prod.yaml > manifests-prod.yaml

# Deploy templated apps
kubectl apply -f manifests-dev.yaml

# Verify templated applications created
argocd app list
# Output: infrastructure, platforms, applications with correct waves
```

**Explanation:** Helm templates reduce duplication. Values per environment. Easy to generate variations. Scales to many applications.

---

## Exercise 10: Troubleshoot App-of-Apps Issues

**Solution:**
```bash
# Check parent application status
argocd app get root
# Shows overall health and sync status

# Check specific child status
argocd app get infrastructure
# Shows if OutOfSync or errors

# Review application logs
argocd app logs root
# Shows reconciliation details

# Check child application logs
argocd app logs infrastructure

# Check application controller logs
kubectl logs -n argocd statefulset/argocd-application-controller -f | grep -i "app-of-apps"

# Identify sync failures
argocd app get root
# Shows which children are unhealthy

# Fix sync issues
# 1. Check manifest syntax
kubectl apply -f infrastructure-app.yaml --dry-run=client

# 2. Verify RBAC
kubectl auth can-i create applications --as system:serviceaccount:argocd:argocd-server -n argocd

# 3. Check directory structure
argocd app manifests infrastructure | head -20

# 4. Force sync if stuck
argocd app sync root --force

# 5. Restart controller if needed
kubectl rollout restart statefulset/argocd-application-controller -n argocd

# Monitor cascading operations
watch 'argocd app list'
```

**Explanation:** Logs show detailed errors. Status checks identify problems. RBAC affects operations. Force sync clears stuck states. Monitoring enables debugging.
