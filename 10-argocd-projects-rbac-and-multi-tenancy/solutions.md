# 10-argocd-projects-rbac-and-multi-tenancy: Solutions

## Exercise 1: Create AppProject for Team Isolation

**Solution:**
```bash
# Create AppProject
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-dev
  namespace: argocd
spec:
  description: Development team project
  sourceRepos:
    - "https://github.com/example/team-dev-repo"
  destinations:
    - namespace: "team-dev"
      server: "https://kubernetes.default.svc"
  namespaceResourceBlacklist:
    - group: ""
      kind: ResourceQuota
    - group: ""
      kind: NetworkPolicy
  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"
EOF

# Verify creation
kubectl get appproject -n argocd
# Output: team-dev project listed

# Get detailed info
kubectl get appproject team-dev -n argocd -o yaml
```

**Explanation:** AppProject defines isolation boundaries. sourceRepos restricts Git access. destinations limits K8s namespaces. Resource lists control what can be deployed.

---

## Exercise 2: Implement Namespace Isolation

**Solution:**
```bash
# Create namespaces
kubectl create namespace team-dev
kubectl create namespace team-dev-staging

# Create AppProject with namespace patterns
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-dev
  namespace: argocd
spec:
  sourceRepos:
    - "https://github.com/example/team-dev-repo"
  destinations:
    - namespace: "team-dev*"
      server: "https://kubernetes.default.svc"
    - namespace: "team-dev-staging"
      server: "https://kubernetes.default.svc"
EOF

# Create application in project
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: team-dev-app
  namespace: argocd
spec:
  project: team-dev
  source:
    repoURL: https://github.com/example/team-dev-repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: team-dev
EOF

# Test isolation - try invalid namespace (should fail)
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: invalid-app
  namespace: argocd
spec:
  project: team-dev
  source:
    repoURL: https://github.com/example/team-dev-repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: other-team
EOF
# Output: Error - namespace not in project destinations

# Verify correct namespace works
argocd app get team-dev-app
# Output: Deployed to team-dev namespace
```

**Explanation:** Wildcard patterns (team-dev*) match multiple namespaces. Each destination is validated. Invalid namespaces rejected by ArgoCD validation.

---

## Exercise 3: Create Service Accounts per Team

**Solution:**
```bash
# Create service account for team
kubectl create serviceaccount team-dev-user -n argocd

# Generate token
TOKEN=$(kubectl create token team-dev-user -n argocd --duration=8760h)
echo "Token: $TOKEN"

# Create role
kubectl create role team-dev-role \
  -n argocd \
  --verb=get,list,create,update,delete \
  --resource=applications,appprojects

# Create rolebinding
kubectl create rolebinding team-dev-binding \
  --role=team-dev-role \
  --serviceaccount=argocd:team-dev-user \
  -n argocd

# Test authentication
argocd login argocd-server --auth-token $TOKEN
argocd app list
# Output: Shows applications accessible to team-dev-user

# Verify permissions
kubectl auth can-i list applications \
  --as system:serviceaccount:argocd:team-dev-user \
  -n argocd
# Output: yes
```

**Explanation:** Service account created for team automation. Token generated with defined TTL. Role restricts to specific resources. Rolebinding connects account to role.

---

## Exercise 4: Configure RBAC for Multi-Tenancy

**Solution:**
```bash
# Create read-only role
kubectl apply -f - << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: team-readonly
  namespace: argocd
rules:
- apiGroups: ["argoproj.io"]
  resources: ["applications", "appprojects"]
  verbs: ["get", "list", "watch"]
EOF

# Create deployer role
kubectl apply -f - << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: team-deployer
  namespace: argocd
rules:
- apiGroups: ["argoproj.io"]
  resources: ["applications"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: ["argoproj.io"]
  resources: ["appprojects"]
  verbs: ["get", "list"]
EOF

# Assign roles to service accounts
kubectl create rolebinding team-readonly-binding \
  --role=team-readonly \
  --serviceaccount=argocd:team-viewer \
  -n argocd

kubectl create rolebinding team-deployer-binding \
  --role=team-deployer \
  --serviceaccount=argocd:team-deployer \
  -n argocd

# Verify permissions
kubectl auth can-i delete applications \
  --as system:serviceaccount:argocd:team-viewer \
  -n argocd
# Output: no

kubectl auth can-i delete applications \
  --as system:serviceaccount:argocd:team-deployer \
  -n argocd
# Output: yes
```

**Explanation:** Different roles grant different permissions. Read-only role for viewers. Deployer role for CI/CD. Least-privilege model enforced.

---

## Exercise 5: Manage Cluster Destinations

**Solution:**
```bash
# Register second cluster
argocd cluster add prod-cluster

# Create AppProject with cluster restrictions
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-prod
  namespace: argocd
spec:
  sourceRepos:
    - "https://github.com/example/team-prod-repo"
  destinations:
    - namespace: "team-prod"
      server: "https://prod-cluster-api:6443"
  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"
EOF

# Create application for prod cluster
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: team-prod-app
  namespace: argocd
spec:
  project: team-prod
  source:
    repoURL: https://github.com/example/team-prod-repo
    targetRevision: main
    path: apps/
  destination:
    server: "https://prod-cluster-api:6443"
    namespace: team-prod
EOF

# List registered clusters
argocd cluster list

# Verify cluster routing
argocd app get team-prod-app
# Output: Deployed to prod cluster
```

**Explanation:** Clusters registered in ArgoCD. Each AppProject restricts to specific clusters. Prevents dev apps deploying to production.

---

## Exercise 6: Enforce Source Repository Restrictions

**Solution:**
```bash
# Create AppProject with restricted repos
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-secure
  namespace: argocd
spec:
  sourceRepos:
    - "https://github.com/example/team-secure-approved-*"
    - "https://gitlab.com/example/team-secure-*"
  destinations:
    - namespace: "team-secure*"
      server: "https://kubernetes.default.svc"
  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"
EOF

# Test allowed repo (should succeed)
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: allowed-app
  namespace: argocd
spec:
  project: team-secure
  source:
    repoURL: https://github.com/example/team-secure-approved-repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: team-secure
EOF
# Output: Succeeds

# Test unauthorized repo (should fail)
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: unauthorized-app
  namespace: argocd
spec:
  project: team-secure
  source:
    repoURL: https://github.com/untrusted/repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: team-secure
EOF
# Output: Error - repository not in project allowed list
```

**Explanation:** sourceRepos whitelist approved repositories. Pattern matching allows flexibility. Unknown repos automatically rejected.

---

## Exercise 7: Implement Resource Quotas

**Solution:**
```bash
# Create resource quota
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-dev
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "100"
    services: "10"
  scopeSelector:
    matchExpressions:
    - operator: In
      scopeName: PriorityClass
      values: ["high"]
EOF

# Create deployment that respects quota
kubectl apply -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quota-aware-app
  namespace: team-dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: quota-aware
  template:
    metadata:
      labels:
        app: quota-aware
    spec:
      containers:
      - name: app
        image: nginx
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
EOF

# View quota usage
kubectl describe resourcequota team-quota -n team-dev
# Output: Shows used vs hard limits

# Test quota enforcement
kubectl set resources deployment quota-aware-app \
  -n team-dev --limits=cpu=100,memory=100Gi
# Output: Error if exceeds quota
```

**Explanation:** ResourceQuota enforces cluster-wide limits per namespace. Prevents single team from consuming all resources. Hard limits prevent excess usage.

---

## Exercise 8: Manage Application Ownership

**Solution:**
```bash
# Create application in specific project
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: team-a-app
  namespace: argocd
  labels:
    team: team-a
spec:
  project: team-a
  source:
    repoURL: https://github.com/example/team-a-repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: team-a-apps
EOF

# Create application in different project
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: team-b-app
  namespace: argocd
  labels:
    team: team-b
spec:
  project: team-b
  source:
    repoURL: https://github.com/example/team-b-repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: team-b-apps
EOF

# List applications by team
argocd app list -o json | jq '.[] | select(.metadata.labels.team=="team-a")'

# Transfer app to different project
kubectl patch application team-a-app -n argocd \
  -p '{"spec":{"project":"team-a-new"}}' \
  --type merge

# Verify ownership
argocd app get team-a-app | grep project
```

**Explanation:** Project field determines ownership. Label helps identify team apps. Patching changes project ownership. Prevents unauthorized transfers.

---

## Exercise 9: Configure Multi-Cluster RBAC

**Solution:**
```bash
# Register multiple clusters
argocd cluster add cluster-dev
argocd cluster add cluster-staging
argocd cluster add cluster-prod

# Create cluster-specific service accounts
for cluster in dev staging prod; do
  kubectl --context=$cluster create serviceaccount argocd-admin -n argocd
  kubectl --context=$cluster create clusterrolebinding argocd-admin \
    --clusterrole=cluster-admin \
    --serviceaccount=argocd:argocd-admin
done

# Create RBAC roles for each cluster
kubectl apply -f - << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argocd-dev-deployer
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "create", "update", "patch"]
  namespaces: ["dev"]
EOF

# Assign roles per cluster
kubectl create clusterrolebinding dev-deployer-binding \
  --clusterrole=argocd-dev-deployer \
  --serviceaccount=argocd:team-dev-user

# Verify cluster access
argocd cluster list
# Output: Shows all registered clusters with permissions
```

**Explanation:** Each cluster has separate RBAC configuration. Service accounts per cluster for isolation. Cluster-specific roles control what can be deployed.

---

## Exercise 10: Troubleshoot Multi-Tenancy Issues

**Solution:**
```bash
# Check permission denied error
argocd app get team-app
# Error: PERMISSION_DENIED

# Verify service account has access
kubectl auth can-i get applications \
  --as system:serviceaccount:argocd:team-user \
  -n argocd

# Check AppProject restrictions
kubectl get appproject team-project -n argocd -o yaml | grep -A5 destinations

# Fix namespace mismatch
kubectl get application team-app -n argocd -o yaml | grep namespace

# Verify rolebinding
kubectl get rolebinding -n argocd | grep team-user

# Check service account permissions
kubectl describe serviceaccount team-user -n argocd

# Review AppProject logs
kubectl logs -n argocd statefulset/argocd-application-controller | tail -50

# Test with explicit context
argocd app get team-app --kubeconfig /path/to/kubeconfig

# Debug destination access
kubectl auth can-i create deployments \
  --as system:serviceaccount:argocd:team-user \
  -n team-namespace
```

**Explanation:** Permission denied typically from missing RBAC. AppProject restrictions must match application destination. Logs show detailed validation errors.
