# 02-helm-chart-design: Cheatsheet

## Chart Structure

| Directory | Purpose |
|-----------|---------|
| `Chart.yaml` | Chart metadata and dependencies |
| `values.yaml` | Default configuration |
| `templates/` | Kubernetes manifest templates |
| `charts/` | Dependency charts (generated) |
| `crds/` | Custom Resource Definitions |
| `.helmignore` | Files to exclude from package |

## Common Template Files

| File | Purpose |
|------|---------|
| `_helpers.tpl` | Reusable template functions |
| `deployment.yaml` | Application deployment |
| `service.yaml` | Service definition |
| `configmap.yaml` | Configuration data |
| `secret.yaml` | Sensitive data |
| `ingress.yaml` | Ingress routing |
| `pvc.yaml` | Persistent volume claims |
| `serviceaccount.yaml` | Service account |
| `clusterrole.yaml` | Cluster-wide role |
| `clusterrolebinding.yaml` | Role binding |

## Dependency Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `helm dependency add` | Add dependency | `helm dependency add postgresql 12.1.5` |
| `helm dependency update` | Download dependencies | `helm dependency update my-chart` |
| `helm dependency list` | List dependencies | `helm dependency list my-chart` |
| `helm dependency build` | Build dependency archive | `helm dependency build my-chart` |

## Values File Organization

| Pattern | Purpose | Example |
|---------|---------|---------|
| `values.yaml` | Default values | image.repository, replicaCount |
| `values-dev.yaml` | Development overrides | Lower resources, replicas |
| `values-prod.yaml` | Production overrides | Higher resources, HA setup |
| `values-aws.yaml` | AWS-specific | EKS annotations, IAM roles |
| `values-gcp.yaml` | GCP-specific | GKE annotations, GSA |

## Chart.yaml Fields

| Field | Purpose | Example |
|-------|---------|---------|
| `apiVersion` | Chart version format | `v2` (current) |
| `name` | Chart name | `myapp` |
| `version` | Chart version (semver) | `1.0.0` |
| `appVersion` | App version deployed | `2.5.1` |
| `description` | Chart description | `Production app chart` |
| `type` | Chart type | `application` or `library` |
| `maintainers` | Chart maintainers | Name and email |
| `dependencies` | Sub-chart dependencies | Name, version, repository |

## Helper Template Patterns

| Pattern | Purpose | Example |
|---------|---------|---------|
| `define "chart.name"` | Short app name | `myapp` |
| `define "chart.fullname"` | Full resource name | `release-myapp` |
| `define "chart.labels"` | Common labels | app, version, instance |
| `define "chart.selector"` | Label selector | For pod selection |

## Conditional Patterns

| Pattern | Usage | Example |
|---------|-------|---------|
| `{{- if .Values.xxx }}` | Enable optional feature | `if .Values.ingress.enabled` |
| `{{- if not .Values.xxx }}` | Disable when false | `if not .Values.autoscaling` |
| `{{- if eq .Values.env "prod" }}` | Compare values | `if eq .Values.environment "production"` |
| `{{- if .Values.xxx.enabled }}` | Check nested condition | `if .Values.persistence.enabled` |

## Loop Patterns

| Pattern | Usage | Example |
|---------|-------|---------|
| `{{- range .Values.list }}` | Iterate list | `range .Values.hosts` |
| `{{- range $k, $v := .Values.map }}` | Iterate map | `range $env, $value := .Values.envVars` |
| `{{- range $i, $v := .Values.list }}` | Iterate with index | `range $index, $item := .Values.items` |

## Resource Naming

| Pattern | Purpose | Used For |
|---------|---------|----------|
| `{{ include "chart.fullname" . }}` | Full resource name | Deployments, services |
| `{{ include "chart.name" . }}` | Short name | Labels, selectors |
| `{{ .Release.Name }}` | Release name only | Dynamic parts |
| `{{ .Chart.Name }}` | Chart name only | Generic names |

## RBAC Best Practices

| Resource | When Needed | Example |
|----------|-------------|---------|
| `ServiceAccount` | App needs pod identity | API access |
| `Role` | Namespace-scoped access | Read pods in namespace |
| `ClusterRole` | Cluster-wide access | Read cluster nodes |
| `RoleBinding` | Bind role to account | Connect SA to role |
| `ClusterRoleBinding` | Cluster-wide binding | Cloud provider integration |

## Common Values Patterns

```yaml
# Global values
global:
  environment: prod
  domain: example.com

# Image configuration
image:
  repository: myapp
  tag: "1.0"
  pullPolicy: IfNotPresent

# Replica configuration
replicaCount: 3

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
  enabled: true
  size: 10Gi
  storageClassName: fast

# Ingress
ingress:
  enabled: true
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix
```

## Hook Annotations

| Hook | Timing | Common Use |
|------|--------|------------|
| `pre-install` | Before chart install | Database init, validation |
| `post-install` | After chart install | Smoke tests, setup |
| `pre-upgrade` | Before chart upgrade | Backup, migration |
| `post-upgrade` | After chart upgrade | Verification |
| `pre-delete` | Before chart delete | Cleanup, backup |
| `post-delete` | After chart delete | Final cleanup |
| `test` | On `helm test` | Validation tests |

## Multi-Environment Setup

```bash
# Install development
helm install myapp chart/ -f values-dev.yaml

# Install production
helm install myapp chart/ -f values-prod.yaml

# Stack overrides
helm install myapp chart/ -f values.yaml -f values-prod.yaml

# Command-line overrides
helm install myapp chart/ -f values.yaml --set replicaCount=5
```
