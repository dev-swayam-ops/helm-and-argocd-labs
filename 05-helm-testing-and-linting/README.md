# 05-helm-testing-and-linting

## What You'll Learn
- Lint charts for syntax and best practice errors
- Validate chart structure and metadata
- Create and run test pods
- Implement chart hook tests
- Test templating with dry-run
- Use pytest for advanced testing
- Automate chart validation in CI/CD

## Prerequisites
- Completed 04-helm-values-and-environments module
- Understanding of Helm chart structure
- Basic Kubernetes testing concepts
- Familiarity with pytest or basic testing frameworks

## Key Concepts

### Linting
Static analysis tool that validates chart syntax, structure, and best practices before deployment.

### Dry-Run
Test mode that shows manifests without creating actual resources.

### Chart Tests
Pods that validate deployed application after Helm installs the release.

### Hook Tests
Test pods attached to chart lifecycle events (post-install, pre-delete, etc.).

### Test Automation
Integration of chart testing into CI/CD pipelines for quality assurance.

## Hands-on Lab: Test and Validate Chart

### Step 1: Create Chart with Tests
```bash
# Create chart
helm create test-app

# Review existing structure
ls -la test-app/templates/tests/
```

### Step 2: Create Linting Rules
```bash
# Run lint with all rules
helm lint test-app

# Lint with strict mode
helm lint test-app --strict

# Generate report
helm lint test-app > lint-report.txt
```

### Step 3: Create Test Pod
```bash
# Create test manifest
cat > test-app/templates/tests/test-connection.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "test-app.fullname" . }}-test"
  labels:
    {{- include "test-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['http://{{ include "test-app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
EOF
```

### Step 4: Validate Templates
```bash
# Generate templates
helm template test-app > manifests.yaml

# Dry-run install
helm install test-release test-app --dry-run --debug

# Validate manifests
kubectl apply -f manifests.yaml --dry-run=server

# Check for errors
helm template test-app | kubectl apply -f - --dry-run=client
```

### Step 5: Deploy and Test
```bash
# Create namespace
kubectl create namespace test-validation

# Install chart
helm install test-release test-app -n test-validation

# Run tests
helm test test-release -n test-validation

# Check test results
kubectl logs -l app.kubernetes.io/instance=test-release -n test-validation

# Verify status
helm status test-release -n test-validation
```

**Expected Output:**
```
==> Linting test-app
[OK] Chart.yaml is valid
[OK] all templates validate against schema
[OK] no issues found

TEST PASSED:
Pod test-release-test successfully connected to service
```

## Validation
- `helm lint` passes without errors
- `helm template` produces valid YAML
- `helm test` pods run successfully
- Dry-run shows expected manifests
- No issues found in schema validation

## Cleanup
```bash
# Uninstall release
helm uninstall test-release -n test-validation

# Delete namespace
kubectl delete namespace test-validation

# Remove chart
rm -rf test-app lint-report.txt manifests.yaml
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Test pod fails to run | Check image availability and pod specs |
| Lint warnings ignored | Run `--strict` to enforce all rules |
| Template syntax errors | Use `helm template --debug` to see rendered YAML |
| Missing chart metadata | Ensure Chart.yaml has all required fields |
| Test hooks not running | Verify `helm.sh/hook` annotation is correct |

## Troubleshooting

**Problem:** Test pod stuck in pending
```bash
# Solution: Check pod status and events
kubectl describe pod <test-pod> -n test-validation
kubectl logs <test-pod> -n test-validation
```

**Problem:** Lint warnings not fixed
```bash
# Solution: Review specific warning
helm lint test-app --strict
# Fix issues and re-run
```

**Problem:** Dry-run validation fails
```bash
# Solution: Check specific resource
helm template test-app --show-only templates/deployment.yaml
kubectl apply -f - --dry-run=server
```

## Next Steps
- Explore **06-argocd-basics** for GitOps implementation
- Implement CI/CD testing with GitHub Actions or GitLab CI
- Create comprehensive test suites for charts
- Integrate chart testing into development workflow
