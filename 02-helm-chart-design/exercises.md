# 02-helm-chart-design: Exercises

## Exercise 1: Plan Chart Directory Structure
Design a production chart layout following Helm conventions.

**Task:**
- Create a new chart: `helm create mybank-app`
- Examine the directory structure
- List all files and their purposes
- Identify optional directories (crds/, _test/, etc.)
- Document the purpose of each file

## Exercise 2: Implement Hierarchical Values
Structure values.yaml for clarity and reusability.

**Task:**
- Edit values.yaml with nested sections:
  - global, image, deployment, service, persistence, ingress
- Create environment-specific value files:
  - values-dev.yaml
  - values-prod.yaml
- Show different configurations for each environment
- Deploy with both: `helm install --values values-dev.yaml`

## Exercise 3: Create Chart Dependencies
Add and manage dependency charts.

**Task:**
- Modify Chart.yaml to include PostgreSQL dependency
- Run `helm dependency update`
- Verify charts/ directory contains dependency
- Create values for dependency: `postgresql.enabled: true`
- Show dependency hierarchy

## Exercise 4: Implement Helper Templates
Create reusable template blocks in _helpers.tpl.

**Task:**
- Open _helpers.tpl file
- Create these helpers:
  - chart.name: returns chart name
  - chart.fullname: full resource name
  - chart.labels: common labels
- Use helpers in templates: `{{ include "chart.name" . }}`
- Validate with `helm template`

## Exercise 5: Design for RBAC
Plan role-based access control resources.

**Task:**
- Create ServiceAccount template
- Create ClusterRole template
- Create ClusterRoleBinding template
- Add RBAC condition: `rbac.create: true`
- Show proper role permissions for read-only access

## Exercise 6: Plan Persistence Strategy
Design storage requirements for applications.

**Task:**
- Create PersistentVolumeClaim template
- Add values for storage:
  - enabled, storageClassName, size, accessModes
- Implement conditional persistence: `if .Values.persistence.enabled`
- Show three storage classes: local, managed-csi, nfs

## Exercise 7: Design Multi-Cloud Compatibility
Structure chart for cloud agnostic deployment.

**Task:**
- Create separate ServiceAccount and RBAC files
- Use generic resource names
- Add cloud-specific values but keep templates neutral
- Show how same chart works on EKS, GKE, AKS
- Document platform-specific overrides

## Exercise 8: Create Chart Tests
Design test manifests to validate chart.

**Task:**
- Create tests/ directory in templates/
- Create a test deployment manifest
- Design a test that verifies pod connectivity
- Use `helm test <release>` command
- Document test success criteria

## Exercise 9: Plan Post-Installation Hooks
Design hooks for initialization tasks.

**Task:**
- Create pre-install hook for database migration
- Create post-install hook for validation
- Design pre-delete hook for cleanup
- Add hooks to templates with proper annotations
- Show hook weight and delete policy

## Exercise 10: Validate Complete Chart Design
Ensure all design decisions are properly implemented.

**Task:**
- Run `helm lint` with strict mode
- Generate templates: `helm template`
- Perform dry-run install: `kubectl apply --dry-run=server`
- Create test namespace and install release
- Verify all components deployed
- Check pod logs and resource status
- Uninstall and cleanup completely
