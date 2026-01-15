# 05-helm-testing-and-linting: Solutions

## Exercise 1: Lint Chart for Errors and Warnings

**Solution:**
```bash
# Basic lint
helm lint myapp
# Output: [OK] Chart.yaml is valid

# Strict mode
helm lint myapp --strict
# Output: [WARNING] values.yaml should not contain...

# Generate report
helm lint myapp > lint-report.txt

# Fix issues identified:
# - Missing Chart description
# - Icon URL invalid
# - No maintainers listed
# - Template spacing issues

# Verify fixes
helm lint myapp
# All [OK] messages
```

**Explanation:** Linting catches errors early. Strict mode enforces best practices. Fix all issues before production deployment.

---

## Exercise 2: Validate Chart Metadata

**Solution:**
```bash
cat > Chart.yaml << 'EOF'
apiVersion: v2
name: myapp
description: A production-ready application
type: application
version: 1.0.0
appVersion: "2.5.1"
maintainers:
  - name: DevOps Team
    email: devops@example.com
    url: https://example.com
keywords:
  - web
  - api
home: https://github.com/example/myapp
sources:
  - https://github.com/example/myapp
dependencies:
  - name: postgresql
    version: "12.1.5"
    repository: "https://charts.bitnami.com/bitnami"
EOF

# Validate
helm lint .
# All metadata checks pass
helm chart metadata .
```

**Explanation:** Complete metadata helps users understand chart. Proper versioning enables dependency management.

---

## Exercise 3: Template Rendering with Dry-Run

**Solution:**
```bash
# Basic template rendering
helm template myapp
# Shows all rendered manifests

# Debug mode shows processing
helm template myapp --debug
# Shows variable substitution steps

# Specific template only
helm template myapp -s templates/deployment.yaml
# Only deployment manifest

# Custom values
helm template myapp -f values-prod.yaml
# Renders with prod configuration

# Save to file
helm template myapp > manifests.yaml

# Compare renderings
diff <(helm template myapp -f values-dev.yaml) \
     <(helm template myapp -f values-prod.yaml)
```

**Explanation:** Template rendering shows final manifests. Debug mode helps identify template issues. Comparison reveals environment differences.

---

## Exercise 4: Validate Kubernetes Manifests

**Solution:**
```bash
# Generate manifests
helm template myapp > manifests.yaml

# Client-side validation
kubectl apply -f manifests.yaml --dry-run=client
# Checks basic YAML and CRD schemas

# Server-side validation
kubectl apply -f manifests.yaml --dry-run=server
# Validates against running API server

# Check specific resource
helm template myapp -s templates/deployment.yaml | \
  kubectl apply -f - --dry-run=server

# Validate with kubeval
kubeval manifests.yaml
# Shows schema errors

# Fix issues and revalidate
helm lint myapp
helm template myapp > manifests.yaml
kubectl apply -f manifests.yaml --dry-run=server
```

**Explanation:** Server-side dry-run catches most errors. Client-side sufficient for basic validation. kubeval provides strict schema checking.

---

## Exercise 5: Create and Run Test Pods

**Solution:**
```bash
# Create test pod template
cat > templates/tests/test-connection.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-test"
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  containers:
    - name: connectivity
      image: busybox
      command: ['wget']
      args: ['http://{{ include "myapp.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
EOF

# Deploy chart
helm install myapp ./myapp

# Run tests
helm test myapp
# Output: PASSED: Pod myapp-test

# Check test pod logs
kubectl logs myapp-test
# Shows test execution details

# Verify pod was cleaned up
kubectl get pods | grep test
# Should be removed after test
```

**Explanation:** Test pods validate deployment. Hook lifecycle controls when tests run. Delete policy cleans up after completion.

---

## Exercise 6: Implement Smoke Tests

**Solution:**
```bash
# Create health check test
cat > templates/tests/test-health.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-health-test"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: health
      image: curlimages/curl
      command: ['curl']
      args: ['http://{{ include "myapp.fullname" . }}:{{ .Values.service.port }}/health']
  restartPolicy: Never
EOF

# Create env var test
cat > templates/tests/test-env.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-env-test"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: env
      image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
      command: ['sh', '-c']
      args: ['echo $APP_NAME && test $APP_NAME = "myapp"']
      env:
        - name: APP_NAME
          value: "myapp"
  restartPolicy: Never
EOF

# Run multiple tests
helm test myapp
# All smoke tests pass
```

**Explanation:** Smoke tests validate critical functionality. Multiple tests cover different aspects. Quick validation before full testing.

---

## Exercise 7: Create Pre-Install Validation Hooks

**Solution:**
```bash
# Pre-install validation job
cat > templates/pre-install-job.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-pre-install"
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
      - name: pre-install
        image: busybox
        command: ['sh']
        args:
          - -c
          - |
            echo "Validating prerequisites..."
            # Check namespace exists
            kubectl get namespace {{ .Release.Namespace }}
            if [ $? -ne 0 ]; then
              echo "ERROR: Namespace not found"
              exit 1
            fi
            echo "All checks passed"
      restartPolicy: Never
EOF

# Deploy with validation
helm install myapp ./myapp
# Pre-install job runs first
# Install proceeds only if job succeeds
```

**Explanation:** Pre-install hooks validate prerequisites. Hook weight controls execution order. Prevents invalid deployments.

---

## Exercise 8: Implement Chart Schema Validation

**Solution:**
```bash
# Create schema file
cat > values.schema.json << 'EOF'
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["image", "service"],
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1,
      "maximum": 10,
      "default": 1
    },
    "image": {
      "type": "object",
      "required": ["repository", "tag"],
      "properties": {
        "repository": {
          "type": "string",
          "minLength": 1
        },
        "tag": {
          "type": "string",
          "minLength": 1
        }
      }
    },
    "environment": {
      "type": "string",
      "enum": ["dev", "staging", "prod"]
    }
  }
}
EOF

# Validate against schema
helm lint myapp
# Checks schema during linting

# Test with invalid values
cat > values-invalid.yaml << 'EOF'
replicaCount: 20  # Exceeds maximum
environment: invalid  # Not in enum
EOF

helm template myapp -f values-invalid.yaml
# Fails schema validation
```

**Explanation:** Schema defines valid values. Validation catches configuration errors early. Prevents invalid deployments.

---

## Exercise 9: Automate Testing in CI/CD

**Solution:**
```bash
# Create test script
cat > test-chart.sh << 'EOF'
#!/bin/bash
set -e

echo "=== Linting ===" 
helm lint ./myapp --strict

echo "=== Template Validation ===" 
helm template myapp ./myapp > /tmp/manifests.yaml
kubectl apply -f /tmp/manifests.yaml --dry-run=server

echo "=== Schema Validation ===" 
helm template myapp -f values-dev.yaml > /dev/null
helm template myapp -f values-prod.yaml > /dev/null

echo "=== Testing ===" 
helm install myapp ./myapp --dry-run --debug

echo "All tests passed!"
EOF

chmod +x test-chart.sh

# Run in CI pipeline
./test-chart.sh

# GitHub Actions example
cat > .github/workflows/test-helm.yml << 'EOF'
name: Helm Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: azure/setup-helm@v1
      - run: ./test-chart.sh
EOF
```

**Explanation:** Automated testing catches errors before merge. CI/CD integration ensures quality. Script can be customized per project.

---

## Exercise 10: Create Complete Test Suite

**Solution:**
```bash
# Create comprehensive test
cat > test-full-suite.sh << 'EOF'
#!/bin/bash
set -e

CHART="./myapp"
NAMESPACE="test-helm"

echo "=== STEP 1: Lint ===" 
helm lint $CHART --strict

echo "=== STEP 2: Template Validation ===" 
helm template test-release $CHART --debug > /tmp/manifests.yaml

echo "=== STEP 3: Kubernetes Validation ===" 
kubectl apply -f /tmp/manifests.yaml --dry-run=server --namespace=$NAMESPACE

echo "=== STEP 4: Multi-Environment ===" 
helm template $CHART -f values-dev.yaml > /dev/null
helm template $CHART -f values-prod.yaml > /dev/null

echo "=== STEP 5: Deploy and Test ===" 
kubectl create namespace $NAMESPACE
helm install test-release $CHART -n $NAMESPACE
helm test test-release -n $NAMESPACE
sleep 5

echo "=== STEP 6: Verify Deployment ===" 
kubectl get all -n $NAMESPACE
kubectl logs -n $NAMESPACE -l app=myapp | head -20

echo "=== STEP 7: Upgrade Test ===" 
helm upgrade test-release $CHART -n $NAMESPACE

echo "=== STEP 8: Rollback Test ===" 
helm rollback test-release -n $NAMESPACE

echo "=== STEP 9: Cleanup ===" 
helm uninstall test-release -n $NAMESPACE
kubectl delete namespace $NAMESPACE

echo "✓ All tests passed!"
EOF

chmod +x test-full-suite.sh
./test-full-suite.sh
```

**Explanation:** Comprehensive suite validates all aspects. Includes deploy, test, upgrade, rollback. Catches regressions.
