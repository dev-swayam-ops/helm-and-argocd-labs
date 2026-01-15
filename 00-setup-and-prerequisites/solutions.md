# 00-setup-and-prerequisites: Solutions

## Exercise 1: Verify Docker Installation

**Solution:**
```bash
# Check Docker version
docker --version

# Run nginx container
docker run -d -p 8080:80 --name test-nginx nginx

# Access browser at http://localhost:8080 - you should see nginx welcome page

# Stop container
docker stop test-nginx

# Remove container
docker rm test-nginx

# Verify container is removed
docker ps -a
```

**Explanation:** This verifies Docker is properly installed, can download images, and containers run correctly.

---

## Exercise 2: Configure Minikube with Custom Resources

**Solution:**
```bash
# Delete existing Minikube cluster
minikube delete

# Start Minikube with custom resources
minikube start --driver=docker --cpus=4 --memory=8192 --addons=ingress

# Verify status
minikube status

# Check enabled addons
minikube addons list
```

**Explanation:** Proper resource allocation prevents performance issues. Ingress addon enables traffic routing to services.

---

## Exercise 3: Explore kubectl Contexts

**Solution:**
```bash
# Display current context
kubectl config current-context
# Output: minikube

# List all contexts
kubectl config get-contexts
# Output shows NAME, CLUSTER, AUTHINFO, NAMESPACE columns

# View complete kubeconfig
kubectl config view

# Show only cluster info
kubectl config view | grep cluster
```

**Explanation:** Contexts allow switching between multiple Kubernetes clusters from a single machine.

---

## Exercise 4: Create and Inspect Kubernetes Namespaces

**Solution:**
```bash
# Create namespaces
kubectl create namespace my-app
kubectl create namespace testing

# List all namespaces
kubectl get namespaces
# Output: default, kube-node-lease, kube-public, kube-system, my-app, testing

# Get detailed namespace info
kubectl describe namespace default

# Delete a namespace
kubectl delete namespace testing

# Verify deletion
kubectl get namespaces
```

**Explanation:** Namespaces logically divide cluster resources and provide multi-tenancy isolation.

---

## Exercise 5: Initialize Helm and Add Repositories

**Solution:**
```bash
# Display Helm version
helm version

# Add repositories
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add jetstack https://charts.jetstack.io

# List repositories
helm repo list

# Search for nginx chart
helm search repo nginx
# Shows multiple nginx charts from different repos

# Show chart details
helm show chart bitnami/nginx

# Pull chart values for inspection
helm show values bitnami/nginx | head -50
```

**Explanation:** Repositories contain curated Helm charts. Multiple repos provide choice and flexibility.

---

## Exercise 6: Deploy Your First Helm Chart

**Solution:**
```bash
# Create namespace
kubectl create namespace my-app-ns

# Deploy chart
helm install web-server bitnami/nginx --namespace my-app-ns

# List releases
helm list --all-namespaces
# Or: helm list -A

# Check pods in namespace
kubectl get pods -n my-app-ns
# Output: 1 nginx pod running

# View release history
helm history web-server -n my-app-ns

# Get release info
helm get all web-server -n my-app-ns
```

**Explanation:** Helm abstracts Kubernetes manifests into reusable charts. Release manages deployments.

---

## Exercise 7: Manage Helm Values

**Solution:**
```bash
# Deploy with custom values
helm install my-release bitnami/nginx --set replicaCount=3

# Verify pods
kubectl get pods
# Output: 3 nginx pods

# Get current values
helm get values my-release

# Upgrade with new values
helm upgrade my-release bitnami/nginx --set replicaCount=5

# Verify upgrade
kubectl get pods
# Output: 5 nginx pods

# Check release revision history
helm history my-release
```

**Explanation:** Values override chart defaults. Helm tracks revisions and allows rollback.

---

## Exercise 8: Examine Kubernetes Resources

**Solution:**
```bash
# List all resource types
kubectl api-resources
# Output: 100+ resource types (pods, services, deployments, etc.)

# Get all resources across namespaces
kubectl get all -A

# Get pods in all namespaces
kubectl get pods -A

# Describe a specific pod
kubectl describe pod web-server-xxxxx -n my-app-ns

# View pod logs
kubectl logs web-server-xxxxx -n my-app-ns

# Follow logs in real-time
kubectl logs web-server-xxxxx -n my-app-ns -f
```

**Explanation:** kubectl provides insight into cluster state and deployed resources.

---

## Exercise 9: Helm Chart Search and Inspection

**Solution:**
```bash
# Search for MySQL
helm search repo mysql

# Show MySQL chart values
helm show values bitnami/mysql | head -30
# Shows: replicaCount, auth, primary, secondary, etc.

# View README
helm show readme bitnami/mysql

# See chart information
helm show chart bitnami/mysql

# List all chart versions
helm search repo mysql --versions
# Output: multiple versions with timestamps

# Get specific version
helm show chart bitnami/mysql --version 9.4.0
```

**Explanation:** Reviewing charts before deployment prevents configuration issues.

---

## Exercise 10: Cleanup and Verification

**Solution:**
```bash
# List all releases in all namespaces
helm list -A

# Delete all releases
helm uninstall web-server -n my-app-ns
helm uninstall my-release

# List to verify
helm list -A
# Output: No releases

# Delete namespaces
kubectl delete namespace my-app-ns
kubectl delete namespace my-app

# Verify namespaces deleted
kubectl get namespaces
# Output: Only system namespaces remain

# Stop Minikube
minikube stop

# Verify cluster stopped
minikube status
# Output: Stopped
```

**Explanation:** Proper cleanup prevents resource waste and ensures clean state for future labs.
