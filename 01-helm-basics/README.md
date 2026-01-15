# 01-helm-basics

## What You'll Learn
- Understand Helm architecture and concepts
- Create and structure a basic Helm chart
- Use Helm values to customize deployments
- Deploy applications using Helm
- Manage releases and rollbacks

## Prerequisites
- Completed 00-setup-and-prerequisites module
- Docker and Minikube running
- Helm installed and repositories configured
- Basic understanding of Kubernetes manifests

## Key Concepts

### Helm Chart
Package containing Kubernetes manifests, templates, and configuration files.

### Release
Deployed instance of a chart in your Kubernetes cluster.

### Values
YAML file(s) containing configuration data passed to chart templates.

### Template
Go template files that generate Kubernetes manifests dynamically.

### Repository
Remote storage containing Helm charts (e.g., Bitnami, Stable).

## Hands-on Lab: Create and Deploy a Simple Chart

### Step 1: Create Chart Directory Structure
```bash
# Create a new chart
helm create my-app-chart

# Examine the structure
tree my-app-chart
# OR
dir my-app-chart /s
```

**Expected Output:**
```
my-app-chart/
├── .helmignore
├── Chart.yaml
├── charts/
├── templates/
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests/
└── values.yaml
```

### Step 2: Review Chart.yaml
```bash
cat my-app-chart/Chart.yaml
```

**Expected Output:**
```yaml
apiVersion: v2
name: my-app-chart
description: A Helm chart for Kubernetes
type: application
version: 1.0.0
appVersion: "1.0"
```

### Step 3: Review Values File
```bash
cat my-app-chart/values.yaml
# Shows default configuration for the chart
```

### Step 4: Lint the Chart
```bash
helm lint my-app-chart
```

**Expected Output:**
```
==> Linting my-app-chart
[OK] Chart.yaml is valid
[OK] templates/ directory is present
[OK] Chart.yaml file is well-formed (yaml)
[OK] all templates validate against the schema
```

### Step 5: Deploy the Chart
```bash
# Create namespace
kubectl create namespace helm-demo

# Install release
helm install my-release my-app-chart -n helm-demo

# Check deployment
helm list -n helm-demo
kubectl get pods -n helm-demo
kubectl get svc -n helm-demo
```

**Expected Output:**
```
NAME       NAMESPACE   REVISION   STATUS    CHART
my-release helm-demo   1          deployed  my-app-chart-1.0.0

NAME                     READY   STATUS    RESTARTS   AGE
my-release-my-app-chart  1/1     Running   0          10s
```

### Step 6: Upgrade and Rollback
```bash
# Upgrade with custom values
helm upgrade my-release my-app-chart -n helm-demo --set replicaCount=3

# Check history
helm history my-release -n helm-demo

# Rollback to previous release
helm rollback my-release 1 -n helm-demo

# Verify
kubectl get pods -n helm-demo
```

## Validation
- `helm list` shows deployed releases
- `kubectl get pods` shows running pods
- `helm lint` passes without errors
- `helm get values` shows correct configuration

## Cleanup
```bash
# Uninstall release
helm uninstall my-release -n helm-demo

# Delete namespace
kubectl delete namespace helm-demo

# Remove chart directory
rm -r my-app-chart
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Chart has invalid YAML | Run `helm lint` to validate |
| Template syntax errors | Check `helm template` output before installing |
| Wrong values applied | Use `helm get values <release>` to verify |
| Image pull issues | Ensure image name is correct in values |
| Resource not found | Check namespace with `-n flag` |

## Troubleshooting

**Problem:** "Release not deployed" error
```bash
# Solution: Check release status
helm status my-release -n helm-demo
helm get manifest my-release -n helm-demo
```

**Problem:** Pods stuck in pending state
```bash
# Solution: Check pod details
kubectl describe pod <pod-name> -n helm-demo
kubectl logs <pod-name> -n helm-demo
```

**Problem:** Upgrade fails
```bash
# Solution: Rollback to previous version
helm rollback my-release -n helm-demo
```

## Next Steps
- Explore **02-helm-chart-design** for advanced chart structure
- Learn template functions and values management
- Understand Helm best practices and patterns
