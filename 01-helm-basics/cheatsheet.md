# 01-helm-basics: Cheatsheet

## Chart Creation & Structure

| Command | Purpose | Example |
|---------|---------|---------|
| `helm create <chart>` | Create new chart | `helm create my-app` |
| `helm lint <chart>` | Validate chart syntax | `helm lint my-app` |
| `helm template <chart>` | Preview manifests | `helm template my-app` |
| `helm package <chart>` | Package chart to tgz | `helm package my-app` |
| `helm show chart <chart>` | Display Chart.yaml | `helm show chart bitnami/nginx` |
| `helm show values <chart>` | Display values.yaml | `helm show values bitnami/nginx` |
| `helm show readme <chart>` | Display README | `helm show readme bitnami/nginx` |

## Release Management

| Command | Purpose | Example |
|---------|---------|---------|
| `helm install <release> <chart>` | Deploy chart | `helm install web bitnami/nginx -n prod` |
| `helm list` | List releases | `helm list -A` |
| `helm list -n <ns>` | List releases in namespace | `helm list -n production` |
| `helm status <release>` | Release status | `helm status web -n prod` |
| `helm history <release>` | Release history | `helm history web -n prod` |
| `helm get values <release>` | Get release values | `helm get values web -n prod` |
| `helm get manifest <release>` | Get release manifest | `helm get manifest web -n prod` |
| `helm get all <release>` | Get complete release info | `helm get all web -n prod` |

## Installing & Upgrading

| Command | Purpose | Example |
|---------|---------|---------|
| `helm install -f values.yaml` | Install with values file | `helm install web bitnami/nginx -f custom.yaml` |
| `helm install --set key=value` | Override single value | `helm install web bitnami/nginx --set replicaCount=3` |
| `helm install --set-string key=val` | Set as string (not interpreted) | `helm install web bitnami/nginx --set-string image.tag=v1` |
| `helm upgrade <release> <chart>` | Update release | `helm upgrade web bitnami/nginx` |
| `helm upgrade --set key=value` | Upgrade with values | `helm upgrade web bitnami/nginx --set replicaCount=5` |
| `helm upgrade --reuse-values` | Keep previous values | `helm upgrade web bitnami/nginx --reuse-values --set replica=2` |

## Release Control

| Command | Purpose | Example |
|---------|---------|---------|
| `helm rollback <release> <rev>` | Revert to revision | `helm rollback web 1 -n prod` |
| `helm delete <release>` | Delete release (same as uninstall) | `helm delete web -n prod` |
| `helm uninstall <release>` | Uninstall release | `helm uninstall web -n prod` |
| `helm uninstall --keep-history` | Uninstall but keep history | `helm uninstall web --keep-history` |

## Values & Configuration

| File | Purpose |
|------|---------|
| `Chart.yaml` | Chart metadata (name, version, description) |
| `values.yaml` | Default configuration values |
| `values-prod.yaml` | Production overrides |
| `values-dev.yaml` | Development overrides |
| `templates/` | Kubernetes manifest templates |

## Common Value Override Patterns

```bash
# Set single value
--set replicaCount=3

# Set nested value
--set image.repository=my-image
--set image.tag=v1.2.0

# Set multiple values
--set replicaCount=3 --set image.tag=latest

# Use values file
--values custom-values.yaml
-f custom-values.yaml

# Combine file and overrides
-f values.yaml --set replicaCount=3

# Set as string (prevents parsing)
--set-string nodeSelector.zone=us-west-2a
```

## Template Syntax (in chart templates)

| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{ .Values.xxx }}` | Access value | `{{ .Values.replicaCount }}` |
| `{{ .Chart.Name }}` | Chart name | `{{ .Chart.Name }}` |
| `{{ .Release.Name }}` | Release name | `{{ .Release.Name }}` |
| `{{ .Namespace }}` | Namespace | `{{ .Namespace }}` |
| `{{ if .Values.xxx }}` | Conditional | `{{ if .Values.ingress.enabled }}` |
| `{{ range .Values.xxx }}` | Loop | `{{ range .Values.hosts }}` |
| `{{ .Labels | toYaml }}` | Format as YAML | `{{ .Values.labels | toYaml }}` |

## Namespace Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-n <namespace>` | Target namespace | `helm install web nginx -n production` |
| `--namespace <ns>` | Long form | `helm install web nginx --namespace production` |
| `-A` | All namespaces | `helm list -A` |
| `--all-namespaces` | All namespaces | `helm list --all-namespaces` |

## Useful Patterns

### Install with Multiple Value Files
```bash
helm install web bitnami/nginx -f values.yaml -f values-prod.yaml
```

### Use Values from Different Files for Different Envs
```bash
# Development
helm install web bitnami/nginx -f values-dev.yaml

# Production
helm install web bitnami/nginx -f values-prod.yaml
```

### Upgrade with Specific Values Only
```bash
helm upgrade web bitnami/nginx --reuse-values --set replicaCount=5
```

### Debug Chart Before Install
```bash
helm template web bitnami/nginx | kubectl apply --dry-run=client -f -
```

### Export Release Values
```bash
helm get values <release> > current-values.yaml
```

## Troubleshooting Commands

| Command | Purpose |
|---------|---------|
| `helm lint <chart>` | Validate chart |
| `helm template <chart>` | Check manifest generation |
| `helm status <release>` | Check release status |
| `helm history <release>` | View deployment history |
| `helm get manifest <release>` | View deployed manifest |
| `kubectl describe pod <pod>` | Debug pod issues |
| `kubectl logs <pod>` | View pod logs |

## Helm Repositories

| Command | Purpose | Example |
|---------|---------|---------|
| `helm repo add <name> <url>` | Add repository | `helm repo add bitnami https://charts.bitnami.com/bitnami` |
| `helm repo list` | List repositories | `helm repo list` |
| `helm repo update` | Update repo cache | `helm repo update` |
| `helm repo remove <name>` | Remove repository | `helm repo remove bitnami` |
| `helm search repo <keyword>` | Search charts | `helm search repo nginx` |
