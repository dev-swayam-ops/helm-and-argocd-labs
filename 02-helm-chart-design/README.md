# 02-helm-chart-design

## What You'll Learn
- Structure a production-ready Helm chart
- Design charts with proper naming conventions
- Implement best practices for chart organization
- Create reusable chart dependencies
- Design for multi-environment deployments
- Follow Helm chart security best practices

## Prerequisites
- Completed 01-helm-basics module
- Understanding of Kubernetes manifests
- Basic YAML syntax knowledge
- Helm installed and running locally

## Key Concepts

### Chart Architecture
Well-organized directory structure following Helm conventions and best practices.

### Chart Dependencies
Ability to reference and bundle other charts as dependencies.

### Values Structure
Hierarchical organization of configuration values for clarity and reusability.

### Template Organization
Logical grouping and modularization of Kubernetes manifest templates.

### Naming Conventions
Consistent, predictable naming patterns for resources and charts.

## Hands-on Lab: Design a Production Chart

### Step 1: Create Chart with Proper Structure
```bash
# Create a production chart
helm create production-app

# Review structure
tree production-app
# Key directories:
# - templates/ for Kubernetes manifests
# - charts/ for dependencies
# - crds/ for custom resource definitions
```

### Step 2: Organize Values File
```bash
# Edit values.yaml with proper structure
cat > production-app/values.yaml << 'EOF'
# Global values
global:
  environment: production
  domain: example.com

# Image configuration
image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

# Deployment settings
deployment:
  replicaCount: 3
  strategy: RollingUpdate

# Service configuration
service:
  type: ClusterIP
  port: 80

# Resource limits
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

# Persistence
persistence:
  enabled: false
  size: 10Gi

# Ingress
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix

# Node affinity
affinity: {}

# Tolerations
tolerations: []
EOF
```

### Step 3: Create Helper Template
```bash
# Create _helpers.tpl for reusable templates
cat > production-app/templates/_helpers.tpl << 'EOF'
{{/*
Expand the name of the chart.
*/}}
{{- define "production-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a fully qualified app name.
*/}}
{{- define "production-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
EOF
```

### Step 4: Define Dependencies
```bash
# Create Chart.yaml with dependencies
cat > production-app/Chart.yaml << 'EOF'
apiVersion: v2
name: production-app
description: A production-ready Helm chart
type: application
version: 1.0.0
appVersion: "1.0"
maintainers:
  - name: DevOps Team
    email: devops@example.com
dependencies:
  - name: postgresql
    version: "12.1.5"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
EOF

# Download dependencies
helm dependency update production-app/
```

### Step 5: Validate Chart Structure
```bash
# Lint chart
helm lint production-app

# Template with dry-run
helm template production-app

# Install in test environment
kubectl create namespace chart-test
helm install prod-release production-app -n chart-test

# Verify
kubectl get all -n chart-test
helm get values prod-release -n chart-test

# Cleanup
helm uninstall prod-release -n chart-test
kubectl delete namespace chart-test
```

**Expected Output:**
```
==> Linting production-app
[OK] Chart.yaml is valid
[OK] all templates validate
[OK] no issues found

Dependencies found:
postgresql: 12.1.5
```

## Validation
- `helm lint` passes without warnings
- Chart includes values.yaml with proper hierarchy
- Templates use helper functions
- Dependencies properly declared and downloaded
- Chart can be installed successfully

## Cleanup
```bash
# Remove test releases
helm uninstall prod-release -n chart-test

# Delete test namespace
kubectl delete namespace chart-test

# Remove chart directory
rm -rf production-app
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Inconsistent value naming | Use hierarchical structure (image.repository, not image_repository) |
| No helper templates | Create _helpers.tpl for common functions |
| Hardcoded values in templates | Move all to values.yaml |
| Missing Chart.yaml fields | Include description, type, and appVersion |
| Poor resource organization | Group related resources (deployment, service, etc.) |

## Troubleshooting

**Problem:** "Dependency not found"
```bash
# Solution: Update dependencies
helm dependency update <chart>
helm dependency list <chart>
```

**Problem:** Template rendering errors
```bash
# Solution: Debug with template command
helm template <chart> --debug
helm lint <chart>
```

**Problem:** Resource naming conflicts
```bash
# Solution: Use proper naming conventions
# Use: {{ include "chart.fullname" . }}
# Not: {{ .Release.Name }}-resource
```

## Next Steps
- Explore **03-helm-templating-and-functions** for advanced templating
- Implement conditional logic and loops in templates
- Create multi-environment value files
- Learn about post-installation hooks and tests
