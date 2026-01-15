# 06-argocd-basics: Solutions

## Exercise 1: Install ArgoCD in Kubernetes

**Solution:**
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify pods running
kubectl get pods -n argocd
# Output: 4 pods running (server, repo-server, controller-manager, dex-server)

# Get initial password
PASSWORD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo "Password: $PASSWORD"

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443 &

# Access at https://localhost:8080
# Username: admin
# Password: (from above)

# Change password
argocd account update-password --account admin --new-password <NEW_PASSWORD>
```

**Explanation:** ArgoCD installs 4 core components. Initial password is temporary and should be changed immediately for security.

---

## Exercise 2: Install and Configure ArgoCD CLI

**Solution:**
```bash
# Download latest CLI
VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep tag_name | sed -E 's/.*"v([^"]+)".*/\1/')
curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/

# Or using package manager
brew install argocd  # macOS
choco install argocd  # Windows

# Verify installation
argocd version
# Output: argocd: v2.x.x, server: v2.x.x

# Login to server
argocd login localhost:8080 --insecure --username admin --password <PASSWORD>

# List accounts
argocd account list

# Test CLI
argocd app list
```

**Explanation:** CLI provides programmatic access to ArgoCD. Login credentials are stored securely. Multiple commands available for management.

---

## Exercise 3: Connect Git Repository

**Solution:**
```bash
# Add public repository (no auth needed)
argocd repo add https://github.com/example/helm-charts

# Add private repository with token
argocd repo add https://github.com/example/private-repo \
  --username git \
  --password <GITHUB_TOKEN>

# Add with SSH key
argocd repo add git@github.com:example/charts.git \
  --ssh-private-key-path ~/.ssh/id_rsa

# List repositories
argocd repo list
# Output: URL and connection status

# Get detailed info
argocd repo get https://github.com/example/helm-charts

# Update repository
argocd repo update https://github.com/example/helm-charts

# Test SSH connection
ssh -T git@github.com
```

**Explanation:** Multiple authentication methods supported. Public repos need no credentials. Private repos require tokens or SSH keys.

---

## Exercise 4: Create First ArgoCD Application

**Solution:**
```bash
# Create simple application
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hello-world
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: nginx
    targetRevision: 15.0.0
  destination:
    server: https://kubernetes.default.svc
    namespace: hello-world
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
EOF

# Verify creation
kubectl get application -n argocd
argocd app list

# Get application details
argocd app get hello-world
# Shows: Status, Health, Sync Status

# View application info
argocd app info hello-world
argocd app info hello-world --refresh

# Check pods deployed
kubectl get pods -n hello-world
```

**Explanation:** Application CRD defines deployment intent. ArgoCD controller watches and reconciles state. Initial status may be OutOfSync.

---

## Exercise 5: Understand ArgoCD Architecture

**Solution:**
```bash
# Check all components
kubectl get deployments,statefulsets -n argocd

# 1. argocd-server - REST API and UI
kubectl logs -n argocd deployment/argocd-server | head -20

# 2. argocd-repo-server - Git/Helm processing
kubectl logs -n argocd deployment/argocd-repo-server | head -20

# 3. argocd-application-controller - Reconciliation
kubectl logs -n argocd statefulset/argocd-application-controller | head -20

# 4. argocd-dex-server - SSO/Authentication
kubectl logs -n argocd deployment/argocd-dex-server | head -20

# View architecture
argocd admin stats
argocd version

# Check API resources
kubectl api-resources | grep argoproj
# Output: applications, appprojects, etc.
```

**Explanation:** Each component has specific role. Server handles API/UI. Repo-server processes sources. Controller maintains desired state.

---

## Exercise 6: Explore ArgoCD UI Features

**Solution:**
```bash
# Access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Login with admin credentials
# Navigate to https://localhost:8080

# Key features:
# 1. Applications view - see all apps and status
# 2. Application detail - view resources and topology
# 3. Sync status - see if in sync with Git
# 4. Health status - check resource health
# 5. Timeline - view operation history
# 6. Resource view - see all deployed Kubernetes resources
# 7. Log viewer - see application logs

# Via CLI equivalent
argocd app list
argocd app get <app-name>
argocd app logs <app-name>
argocd app wait <app-name> --health
```

**Explanation:** UI provides visual status overview. Useful for monitoring and debugging. CLI provides equivalent functionality programmatically.

---

## Exercise 7: Configure ArgoCD RBAC

**Solution:**
```bash
# Create new account
argocd account create deploy-user

# Create role with permissions
kubectl patch configmap argocd-rbac-cm -n argocd -p \
  '{"data":{"policy.csv":"p, deploy-user, applications, get, */*, allow\np, deploy-user, applications, sync, */*, allow"}}'

# Assign role to user
argocd account update-password --account deploy-user --new-password newpass

# Test new user login
argocd login localhost:8080 --username deploy-user --password newpass

# List roles
kubectl get -n argocd configmap argocd-rbac-cm -o yaml

# Verify permissions
argocd app list  # Should work for deploy-user
```

**Explanation:** RBAC uses ConfigMap for policies. Users can be restricted to specific applications or actions. Useful for team permissions.

---

## Exercise 8: Set Up Notification Webhooks

**Solution:**
```bash
# Create notification service account
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: argocd-notifications
  namespace: argocd
EOF

# Configure webhook (example with generic webhook)
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
EOF

# Install notifications controller
kubectl apply -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/manifests/install.yaml

# Verify
kubectl get pods -n argocd-notifications
```

**Explanation:** Notifications require separate controller. Triggers define events. Multiple notification backends supported (Slack, email, webhook).

---

## Exercise 9: Monitor Application Status

**Solution:**
```bash
# Get current status
argocd app get hello-world

# Check health
argocd app info hello-world
# Shows: health status, sync status, repo info

# Watch real-time changes
argocd app wait hello-world --health

# View operation history
argocd app history hello-world
# Shows: revision, timestamp, status

# Check pod status
kubectl get pods -n hello-world -o wide

# Monitor logs
kubectl logs -f -n hello-world <pod-name>

# Check events
kubectl describe pod -n hello-world <pod-name>
kubectl get events -n hello-world
```

**Explanation:** Multiple ways to monitor. Health status tracks resource readiness. Sync status shows Git vs cluster state alignment.

---

## Exercise 10: Troubleshoot ArgoCD Issues

**Solution:**
```bash
# Check repo-server logs
kubectl logs -n argocd deployment/argocd-repo-server --tail=50
# Look for: connection errors, credential issues, parsing errors

# Check controller logs
kubectl logs -n argocd statefulset/argocd-application-controller --tail=50
# Look for: reconciliation errors, sync issues

# Check server logs
kubectl logs -n argocd deployment/argocd-server --tail=50
# Look for: API errors, authentication issues

# Debug application
argocd app get <app> --refresh
argocd app logs <app>

# Check repository connectivity
argocd repo get https://github.com/example/repo
argocd repo update https://github.com/example/repo

# Verify RBAC
kubectl get -n argocd configmap argocd-rbac-cm -o yaml

# Check cluster connectivity
kubectl cluster-info
kubectl auth can-i create applications --as system:serviceaccount:argocd:argocd-controller-manager -n argocd
```

**Explanation:** Logs are primary troubleshooting tool. Check each component's logs. RBAC and credentials are common issues.
