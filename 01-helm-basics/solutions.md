# 01-helm-basics: Solutions

## Exercise 1: Create Your First Helm Chart

**Solution:**
```bash
# Create new chart
helm create my-nginx-chart

# View structure (Windows)
tree my-nginx-chart
# OR
dir my-nginx-chart /s /b

# Explain structure:
# Chart.yaml - chart metadata (name, version, description)
# values.yaml - default configuration values
# templates/ - Kubernetes manifest templates
# charts/ - dependency charts (if any)
# .helmignore - files to exclude when packaging
```

**Explanation:** `helm create` generates a production-ready chart template with all necessary files. The templates folder contains Go-based templates that merge with values.yaml to generate Kubernetes manifests.

---

## Exercise 2: Customize Chart Metadata

**Solution:**
```bash
# Edit Chart.yaml
# Change these values:
# name: webapp
# description: "A simple web application deployment"
# version: 2.0.0
# appVersion: "1.0"

# Validate the chart
helm lint my-nginx-chart
# Output: [OK] Chart is valid

# Display Chart.yaml
cat my-nginx-chart/Chart.yaml
```

**Explanation:** Chart.yaml is the chart metadata file. Version is incremented when chart changes. appVersion tracks the application version being deployed.

---

## Exercise 3: Explore Default Values

**Solution:**
```bash
# Display values file
cat my-nginx-chart/values.yaml

# Key sections include:
# - replicaCount: 1 (number of pod replicas)
# - image: repository and tag for container image
# - service: type and port configuration
# - ingress: ingress configuration
# - resources: CPU and memory limits

# Create custom values file
cat > custom-values.yaml << EOF
replicaCount: 2
image:
  repository: nginx
  tag: "1.21"
service:
  type: LoadBalancer
  port: 8080
EOF
```

**Explanation:** values.yaml is the single source of truth for chart configuration. Templates reference these values using the `{{ .Values.xxx }}` syntax.

---

## Exercise 4: Validate Chart Syntax

**Solution:**
```bash
# Lint the chart
helm lint my-nginx-chart
# Output: [OK] all templates validate

# Preview generated manifests
helm template my-nginx-chart --namespace chart-testing

# Save template output to file
helm template my-nginx-chart > manifest.yaml

# Dry-run with kubectl
kubectl apply --dry-run=client -f manifest.yaml

# Check for YAML syntax errors
helm lint --strict my-nginx-chart
```

**Explanation:** Validation catches errors before deployment. `helm template` shows what manifests will be generated. Dry-run validates without creating resources.

---

## Exercise 5: Install Chart with Default Values

**Solution:**
```bash
# Create namespace
kubectl create namespace chart-testing

# Install with defaults
helm install my-app my-nginx-chart -n chart-testing

# List releases
helm list -n chart-testing
# Output: my-app | chart-testing | 1 | deployed | my-nginx-chart

# Check pods
kubectl get pods -n chart-testing
# Output: 1 nginx pod running

# Get values
helm get values my-app -n chart-testing
# Output: shows default values from values.yaml
```

**Explanation:** `helm install` creates a release from a chart. Each release is independent and can be upgraded, rolled back, or deleted separately.

---

## Exercise 6: Deploy with Custom Values

**Solution:**
```bash
# Install with custom values
helm install web-app my-nginx-chart -n chart-testing \
  --set replicaCount=3 \
  --set image.tag=latest

# Verify 3 pods
kubectl get pods -n chart-testing
# Output: 3 nginx pods

# Get values
helm get values web-app -n chart-testing
# Output: replicaCount: 3, image.tag: latest

# View manifest
helm get manifest web-app -n chart-testing
# Output: Shows deployment with 3 replicas
```

**Explanation:** `--set` flag overrides specific values. Multiple `--set` flags can be combined. This is useful for environment-specific configurations.

---

## Exercise 7: Upgrade a Release

**Solution:**
```bash
# Upgrade with new values
helm upgrade my-app my-nginx-chart -n chart-testing \
  --set replicaCount=5

# Check history
helm history my-app -n chart-testing
# Output:
# REVISION | STATUS    | CHART              | APP VERSION
# 1        | superseded| my-nginx-chart-1.0 | 1.0
# 2        | deployed  | my-nginx-chart-1.0 | 1.0

# Verify 5 pods
kubectl get pods -n chart-testing
# Output: 5 nginx pods

# Get full info
helm get all my-app -n chart-testing
```

**Explanation:** Upgrade creates a new revision while keeping previous versions. Kubernetes gradually scales pods to avoid downtime.

---

## Exercise 8: Rollback Release

**Solution:**
```bash
# View history
helm history my-app -n chart-testing

# Rollback to revision 1
helm rollback my-app 1 -n chart-testing
# Output: Rollback to 1

# Check history (new revision created)
helm history my-app -n chart-testing
# Output: Revision 3 created as rollback to revision 1

# Verify pod count reverted
kubectl get pods -n chart-testing
# Output: Back to original replica count

# Check current values
helm get values my-app -n chart-testing
```

**Explanation:** Rollback creates a new revision (not revert). Kubernetes handles pod cleanup/creation. Useful for quick recovery from bad deployments.

---

## Exercise 9: Compare Values Between Releases

**Solution:**
```bash
# Get values from both releases
helm get values my-app -n chart-testing > release1-values.yaml
helm get values web-app -n chart-testing > release2-values.yaml

# Display and compare
cat release1-values.yaml
cat release2-values.yaml

# Manual comparison shows differences:
# my-app: replicaCount: 5, image.tag: default
# web-app: replicaCount: 3, image.tag: latest

# Use diff tool (optional)
# diff release1-values.yaml release2-values.yaml
```

**Explanation:** Exporting values helps understand configuration drift. Different releases can have different settings even from the same chart.

---

## Exercise 10: Uninstall and Cleanup

**Solution:**
```bash
# List all releases
helm list -n chart-testing
# Output: my-app and web-app

# Uninstall both
helm uninstall my-app -n chart-testing
helm uninstall web-app -n chart-testing

# Verify empty
helm list -n chart-testing
# Output: no rows in table

# Check pods gone
kubectl get pods -n chart-testing
# Output: no resources found

# Delete namespace
kubectl delete namespace chart-testing

# Cleanup files
rm -f custom-values.yaml release1-values.yaml release2-values.yaml manifest.yaml
rm -rf my-nginx-chart

# Verify cleanup complete
helm list -A
kubectl get namespaces
```

**Explanation:** Proper cleanup prevents resource waste. Uninstall removes all Kubernetes resources. Deleting namespace removes namespace-scoped data.
