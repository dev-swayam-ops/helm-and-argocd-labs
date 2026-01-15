# 06-argocd-basics

## What You'll Learn
- Understand GitOps principles and ArgoCD architecture
- Install and configure ArgoCD
- Connect Git repositories to ArgoCD
- Create and manage ArgoCD applications
- Monitor application health and sync status
- Use ArgoCD CLI for management
- Implement continuous deployment with ArgoCD

## Prerequisites
- Completed 05-helm-testing-and-linting module
- Kubernetes cluster running (Minikube, Kind, or cloud)
- Git repository access (GitHub, GitLab, Gitea)
- kubectl configured and working
- Helm installed

## Key Concepts

### GitOps
Declarative infrastructure management using Git as source of truth. All infrastructure and application changes tracked in version control.

### ArgoCD
GitOps continuous deployment tool that automatically syncs Kubernetes resources from Git repositories.

### Application
ArgoCD resource defining source repository, destination cluster, and sync policies.

### Sync
Process of reconciling actual cluster state with desired state from Git repository.

### Health Status
Indicator showing if application resources are healthy, progressing, or degraded.

## Hands-on Lab: Install and Configure ArgoCD

### Step 1: Install ArgoCD
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify installation
kubectl get pods -n argocd
kubectl get svc -n argocd
```

**Expected Output:**
```
NAME                                    READY   STATUS    RESTARTS
argocd-server-xxx                       1/1     Running   0
argocd-repo-server-xxx                  1/1     Running   0
argocd-controller-manager-xxx           1/1     Running   0
argocd-dex-server-xxx                   1/1     Running   0
```

### Step 2: Access ArgoCD UI
```bash
# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial password
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Access at https://localhost:8080
# Username: admin
# Password: (from above command)
```

### Step 3: Install ArgoCD CLI
```bash
# Download CLI
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd

# Or use package manager
brew install argocd  # macOS
choco install argocd  # Windows

# Verify
argocd version
```

### Step 4: Configure Git Repository
```bash
# Login to ArgoCD
argocd login localhost:8080 --insecure --username admin --password <PASSWORD>

# Add Git repository
argocd repo add https://github.com/example/helm-charts \
  --username git \
  --password <GIT_TOKEN>

# Verify repository
argocd repo list
```

### Step 5: Create First Application
```bash
# Create application
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hello-world
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/helm-charts
    targetRevision: main
    path: charts/hello-world
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

# Check application status
argocd app get hello-world
kubectl get application -A
```

**Expected Output:**
```
Name:               hello-world
Project:            default
Status:             Synced
Repository:         https://github.com/example/helm-charts
Sync Policy:        Automated
Health Status:      Healthy
```

## Validation
- ArgoCD server is running and accessible
- Repository is connected and accessible
- Application is created and synced
- Pod health status shows Healthy
- Git changes automatically trigger sync

## Cleanup
```bash
# Delete application
kubectl delete application hello-world -n argocd

# Delete ArgoCD
kubectl delete namespace argocd

# Remove port-forward
# Ctrl+C in terminal
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Initial password not retrieved | Check argocd-initial-admin-secret exists |
| UI not accessible | Verify port-forward and network connectivity |
| Repository not connecting | Check Git credentials and token expiration |
| Application not syncing | Verify source path exists in repository |
| Health status unknown | Wait for controller reconciliation |

## Troubleshooting

**Problem:** "Unable to connect to repository"
```bash
# Solution: Verify credentials
argocd repo list
argocd repo get https://github.com/example/repo
argocd account list
```

**Problem:** Application stuck in "OutOfSync"
```bash
# Solution: Manual sync
argocd app sync hello-world
argocd app wait hello-world
```

**Problem:** Pods not appearing after sync
```bash
# Solution: Check application logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
argocd app logs hello-world
```

## Next Steps
- Explore **07-argocd-applications-and-sync** for advanced application management
- Implement GitOps workflows with ArgoCD
- Set up multi-cluster deployments
- Implement automated rollbacks and progressive delivery
