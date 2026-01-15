# 00-setup-and-prerequisites

## What You'll Learn
- Install Docker for container runtime
- Install Kubernetes locally using Minikube or Kind
- Install kubectl CLI tool
- Install Helm package manager
- Verify all installations are working correctly
- Understand the prerequisite architecture

## Prerequisites
- Windows 10/11 or Linux or macOS
- At least 8GB RAM (16GB recommended)
- 50GB free disk space
- Administrator access
- Internet connection

## Key Concepts

### Kubernetes (K8s)
Container orchestration platform that manages containerized applications across clusters.

### Docker
Containerization platform that packages applications with dependencies into containers.

### kubectl
Command-line interface to communicate with Kubernetes clusters.

### Helm
Package manager for Kubernetes that simplifies application deployment.

### Minikube
Local Kubernetes cluster for development and testing.

## Hands-on Lab: Complete Setup

### Step 1: Install Docker
**For Windows:**
```bash
# Download from: https://www.docker.com/products/docker-desktop
# Install Docker Desktop
# Verify installation
docker --version
docker run hello-world
```

**Expected Output:**
```
Docker version 24.0.0, build xxxxxxx
Hello from Docker!
```

### Step 2: Install Minikube
**For Windows (using PowerShell as Admin):**
```bash
# Download and install Minikube
choco install minikube
# OR download from: https://minikube.sigs.k8s.io/docs/start/

# Verify installation
minikube version

# Start Minikube cluster
minikube start --driver=docker --cpus=4 --memory=8192

# Check cluster status
minikube status
```

**Expected Output:**
```
minikube: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

### Step 3: Install kubectl
```bash
# Verify kubectl (comes with Minikube)
kubectl version --client

# Verify cluster connection
kubectl cluster-info
kubectl get nodes
```

**Expected Output:**
```
Client Version: v1.28.0
Cluster "minikube" set as active context

NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   5m    v1.28.0
```

### Step 4: Install Helm
**For Windows:**
```bash
# Using Chocolatey
choco install kubernetes-helm

# OR download from: https://github.com/helm/helm/releases

# Verify installation
helm version

# Add Helm repositories
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

**Expected Output:**
```
version.BuildInfo{Version:"v3.13.0", GitCommit:"xxx", ...}
```

### Step 5: Verify Complete Setup
```bash
# Create test namespace
kubectl create namespace test-setup

# Deploy Nginx using Helm
helm install my-nginx bitnami/nginx --namespace test-setup

# Check deployment
kubectl get pods -n test-setup
kubectl get svc -n test-setup

# Cleanup
helm uninstall my-nginx -n test-setup
kubectl delete namespace test-setup
```

## Validation
- Docker: `docker ps` shows no errors
- Kubernetes: `kubectl get nodes` shows minikube as Ready
- Helm: `helm list` command works
- Integration: Can deploy a simple chart successfully

## Cleanup
```bash
# Stop Minikube (keeps data)
minikube stop

# Delete Minikube cluster (if needed)
minikube delete

# Remove Helm repositories
helm repo remove stable
helm repo remove bitnami
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Docker daemon not running | Start Docker Desktop or docker service |
| Minikube driver issues | Ensure Docker is running before starting Minikube |
| kubectl can't connect | Run `kubectl config view` and check context |
| Helm command not found | Ensure Helm is in system PATH |
| Insufficient resources | Allocate more CPU/memory: `minikube start --cpus=4 --memory=8192` |

## Troubleshooting

**Problem:** "Connection refused" when running kubectl
```bash
# Solution: Check cluster status
minikube status
minikube restart
```

**Problem:** Docker daemon issues
```bash
# Solution: Restart Docker service
# Windows: Restart Docker Desktop
# Linux: sudo systemctl restart docker
```

**Problem:** Minikube slow or stuck
```bash
# Solution: Delete and recreate
minikube delete
minikube start --driver=docker
```

## Next Steps
- Proceed to **01-helm-basics** to learn Helm fundamentals
- Explore kubectl basic commands
- Understand YAML syntax for Kubernetes manifests
