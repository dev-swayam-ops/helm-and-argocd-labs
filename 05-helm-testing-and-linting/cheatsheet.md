# 05-helm-testing-and-linting: Cheatsheet

## Linting Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `helm lint <chart>` | Basic linting | `helm lint myapp` |
| `helm lint --strict` | Enforce all rules | `helm lint myapp --strict` |
| `helm lint --quiet` | Minimal output | `helm lint myapp --quiet` |
| `helm lint --values` | Lint with values | `helm lint -f values.yaml myapp` |

## Template Rendering

| Command | Purpose | Example |
|---------|---------|---------|
| `helm template <release> <chart>` | Render all templates | `helm template my-app myapp` |
| `helm template --debug` | Show processing steps | `helm template my-app myapp --debug` |
| `helm template -s <template>` | Specific template | `helm template -s templates/deployment.yaml` |
| `helm template -f <values>` | Custom values | `helm template -f values-prod.yaml` |
| `helm template --output-dir` | Save to files | `--output-dir /tmp/manifests` |
| `helm template --show-only` | Single resource | `--show-only templates/service.yaml` |

## Dry-Run Validation

| Command | Purpose | Example |
|---------|---------|---------|
| `helm install --dry-run` | Dry-run install | `helm install test myapp --dry-run` |
| `helm install --dry-run --debug` | Debug dry-run | `helm install test myapp --dry-run --debug` |
| `kubectl apply --dry-run=client` | Client validation | `kubectl apply -f manifest.yaml --dry-run=client` |
| `kubectl apply --dry-run=server` | Server validation | `kubectl apply -f manifest.yaml --dry-run=server` |

## Test Pod Hooks

| Hook | Timing | Example |
|------|--------|---------|
| `pre-install` | Before install | Database migrations |
| `post-install` | After install | Smoke tests |
| `pre-upgrade` | Before upgrade | Backup, validation |
| `post-upgrade` | After upgrade | Verification |
| `pre-delete` | Before delete | Cleanup, export data |
| `test` | On `helm test` | Validation tests |

## Test Pod Cleanup Policies

| Policy | Behavior |
|--------|----------|
| `before-hook-creation` | Delete before running hook |
| `hook-succeeded` | Keep if hook succeeds |
| `hook-failed` | Keep if hook fails |
| `none` | Never delete |

## Test Execution

| Command | Purpose | Example |
|---------|---------|---------|
| `helm test <release>` | Run release tests | `helm test myapp` |
| `helm test --namespace` | Test in namespace | `helm test myapp -n prod` |
| `kubectl logs <test-pod>` | View test logs | `kubectl logs myapp-test` |
| `kubectl describe pod <test>` | Test pod details | `kubectl describe pod myapp-test` |

## Schema Validation

```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["image", "service"],
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1,
      "maximum": 10
    },
    "image": {
      "type": "object",
      "required": ["repository"],
      "properties": {
        "repository": { "type": "string" },
        "tag": { "type": "string" }
      }
    }
  }
}
```

## Common Lint Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Missing description | Chart.yaml incomplete | Add description field |
| Invalid YAML | Syntax error | Check indentation and quotes |
| Unknown field | Typo in template | Verify variable names |
| Icon URL invalid | Bad URL format | Use valid HTTPS URL |
| No maintainers | Missing contact info | Add maintainers section |

## Pre-Install Hook Example

```yaml
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
        command: ['sh', '-c']
        args:
          - echo "Running pre-install validation"
      restartPolicy: Never
```

## Test Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "chart.fullname" . }}-test"
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  containers:
    - name: test
      image: busybox
      command: ['wget']
      args: ['http://myservice:80/health']
  restartPolicy: Never
```

## Debugging Linting Issues

```bash
# Show detailed lint output
helm lint ./myapp -v

# Test specific values
helm lint ./myapp -f values-prod.yaml

# Template debugging
helm template . --debug 2>&1 | head -50

# Validate YAML
kubectl create -f manifest.yaml --dry-run=client -o yaml
```

## Testing Workflow

```bash
# 1. Lint chart
helm lint ./myapp --strict

# 2. Generate manifests
helm template myapp ./myapp > manifests.yaml

# 3. Validate manifests
kubectl apply -f manifests.yaml --dry-run=server

# 4. Install
helm install myapp ./myapp -n test

# 5. Run tests
helm test myapp -n test

# 6. Check results
kubectl logs -l app=myapp -n test

# 7. Cleanup
helm uninstall myapp -n test
```

## CI/CD Integration

```bash
# Basic test script
set -e
helm lint ./chart --strict
helm template test-release ./chart > /tmp/manifest.yaml
kubectl apply -f /tmp/manifest.yaml --dry-run=server

# Multi-environment test
for env in dev staging prod; do
  helm lint ./chart -f values-$env.yaml
  helm template -f values-$env.yaml ./chart > /tmp/$env.yaml
  kubectl apply -f /tmp/$env.yaml --dry-run=server
done
```

## Validation Tools

| Tool | Purpose | Command |
|------|---------|---------|
| `helm lint` | Chart validation | `helm lint ./chart` |
| `helm template` | Manifest rendering | `helm template . ./chart` |
| `kubeval` | Schema validation | `kubeval manifest.yaml` |
| `kubelint` | Best practices | `kubelint manifest.yaml` |
| `kubectl apply --dry-run` | Kubernetes validation | `kubectl apply -f - --dry-run=server` |
