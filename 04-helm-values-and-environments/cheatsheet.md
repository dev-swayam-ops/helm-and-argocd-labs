# 04-helm-values-and-environments: Cheatsheet

## Values File Management

| Pattern | Purpose | Example |
|---------|---------|---------|
| `values.yaml` | Default configuration | Common settings for all envs |
| `values-dev.yaml` | Dev environment | Low resources, debug logging |
| `values-staging.yaml` | Staging environment | Medium resources, info logging |
| `values-prod.yaml` | Production environment | High resources, warn logging |
| `values-secrets.yaml` | Sensitive data | Passwords, tokens, API keys |
| `values.schema.json` | Validation rules | Type and required field validation |

## Value Override Methods

| Method | Purpose | Example | Priority |
|--------|---------|---------|----------|
| `values.yaml` | Default values | Built-in defaults | Lowest |
| `-f file.yaml` | File override | `-f values-prod.yaml` | Medium |
| `--set key=value` | Single value | `--set replicaCount=3` | High |
| `--set-string key=val` | String value | `--set-string image.tag=v1` | High |
| Multiple `-f` | Layer files | `-f base.yaml -f prod.yaml` | Stacked |

## Installation Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `helm install` | Deploy with values | `helm install app chart -f values-prod.yaml` |
| `-f, --values` | Use values file | `-f prod-values.yaml` |
| `--set` | Override value | `--set replicaCount=5` |
| `--set-string` | String override | `--set-string image.tag=v1` |
| `--values-set` | Multiple sets | `--set x=1 --set y=2` |
| `helm get values` | Show active values | `helm get values <release>` |
| `helm upgrade` | Update values | `helm upgrade app chart -f new-values.yaml` |

## Values Structure Patterns

```yaml
# Hierarchical organization
global:
  environment: prod
  domain: example.com

image:
  repository: myapp
  tag: "1.0"
  pullPolicy: IfNotPresent

resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

database:
  host: db.prod.local
  port: 5432
  name: app_prod

features:
  cache:
    enabled: true
  audit:
    enabled: true
```

## Environment-Specific Patterns

### Development
```yaml
environment: dev
replicaCount: 1
image:
  tag: "dev"
  pullPolicy: Always
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
  limits:
    memory: "128Mi"
    cpu: "100m"
logging:
  level: debug
```

### Production
```yaml
environment: prod
replicaCount: 3
image:
  tag: "latest"
  pullPolicy: IfNotPresent
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
logging:
  level: warn
persistence:
  enabled: true
```

## Validation Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `required` | Force required value | `{{ .Values.key \| required "key required" }}` |
| `fail` | Stop with error | `{{ if not .Values.x }}{{ fail "x required" }}{{ end }}` |
| `has` | Check key exists | `{{ has "key" .Values.map }}` |
| `empty` | Check if empty | `{{ if empty .Values.x }}` |
| `eq` | Value comparison | `{{ if eq .Values.env "prod" }}` |

## Multi-Environment Deployment

```bash
# Create namespaces
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod

# Deploy to dev
helm install app-dev chart/ -f values-dev.yaml -n dev

# Deploy to staging
helm install app-staging chart/ -f values-staging.yaml -n staging

# Deploy to prod
helm install app-prod chart/ -f values-prod.yaml -n prod

# List all releases
helm list -A

# Get values from each
helm get values app-dev -n dev
helm get values app-staging -n staging
helm get values app-prod -n prod
```

## Common Value Overrides

```bash
# Resource scaling
--set resources.limits.memory=2Gi

# Replica count
--set replicaCount=5

# Image version
--set-string image.tag=v2.0.0

# Database host
--set database.host=prod-db.internal

# Enable features
--set features.cache.enabled=true

# Multiple values
--set replicaCount=3 --set logging.level=debug
```

## Secrets Management

| Pattern | Usage | Security |
|---------|-------|----------|
| `values-secrets.yaml` | Store secrets in file | Not recommended (commit risk) |
| Kubernetes Secret | Reference secret in template | Better (external management) |
| Sealed Secrets | Encrypt secrets in Git | Recommended |
| External Secrets | Vault/AWS Secrets Manager | Best (central management) |

## Comparison and Validation

```bash
# Compare released values
diff <(helm get values app-dev) values-dev.yaml

# Export for analysis
helm get values app-prod > prod-current.yaml

# Show differences
diff values-prod.yaml prod-current.yaml

# Validate before install
helm template chart -f values-prod.yaml | kubectl apply --dry-run=server -f -
```

## GitOps Patterns

| Pattern | Purpose | Tool |
|---------|---------|------|
| Git-driven values | Source of truth in Git | ArgoCD, Flux |
| Kustomize overlays | Layer values per env | Kustomize |
| Helmfile | Declarative releases | Helmfile |
| Sealed Secrets | Encrypted secrets in Git | Sealed Secrets |
| External Secrets | Reference external vaults | External Secrets |

## Debugging Values

```bash
# Show all values in effect
helm get values <release>

# Template with debug
helm template chart --debug

# Compare values
helm get values <release> | diff - values.yaml

# Check specific value
helm get values <release> | grep replicas
```

## Best Practices

| Practice | Reason | Example |
|----------|--------|---------|
| Use base + override | DRY, easier maintenance | `-f base.yaml -f prod.yaml` |
| Document values | Helps users | Add comments in values.yaml |
| Validate required | Prevent bad deployments | Use `required` function |
| Separate secrets | Security | Don't commit secrets file |
| Test values | Catch errors early | `helm template --debug` |
| Version values | Track changes | Git track values files |
| Use strict lint | Catch issues | `helm lint --strict` |
