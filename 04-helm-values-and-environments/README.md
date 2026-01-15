# 04-helm-values-and-environments

## What You'll Learn
- Manage values across multiple environments
- Create environment-specific value files
- Override values at deployment time
- Use value templates and defaults
- Handle secrets securely
- Implement GitOps patterns with values
- Scale configurations for different contexts

## Prerequisites
- Completed 03-helm-templating-and-functions module
- Understanding of Helm chart structure
- Knowledge of template syntax and functions
- Basic understanding of YAML and environment concepts

## Key Concepts

### Values File
YAML files containing configuration that templates reference for dynamic manifest generation.

### Environment Isolation
Separate value files (dev, staging, prod) allowing different configurations per environment.

### Value Override
Method of replacing default values using -f flag, --set, or value files.

### Secrets Management
Secure handling of sensitive data (passwords, tokens, API keys) in Helm deployments.

### Value Inheritance
Base values overridden by environment-specific values in hierarchical manner.

## Hands-on Lab: Multi-Environment Setup

### Step 1: Create Base Chart
```bash
# Create chart
helm create multi-env-app

# Review current values
cat multi-env-app/values.yaml
```

### Step 2: Create Environment Value Files
```bash
# Base development values
cat > multi-env-app/values-dev.yaml << 'EOF'
environment: development
replicaCount: 1
image:
  repository: myapp
  tag: "dev"
  pullPolicy: Always
resources:
  limits:
    memory: "256Mi"
    cpu: "200m"
  requests:
    memory: "128Mi"
    cpu: "100m"
database:
  host: postgres-dev.local
  port: 5432
  name: app_dev
logging:
  level: debug
EOF

# Staging values
cat > multi-env-app/values-staging.yaml << 'EOF'
environment: staging
replicaCount: 2
image:
  repository: myapp
  tag: "staging"
  pullPolicy: IfNotPresent
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"
  requests:
    memory: "256Mi"
    cpu: "250m"
database:
  host: postgres-staging.local
  port: 5432
  name: app_staging
logging:
  level: info
EOF

# Production values
cat > multi-env-app/values-prod.yaml << 'EOF'
environment: production
replicaCount: 3
image:
  repository: myapp
  tag: "latest"
  pullPolicy: IfNotPresent
resources:
  limits:
    memory: "1Gi"
    cpu: "1000m"
  requests:
    memory: "512Mi"
    cpu: "500m"
database:
  host: postgres-prod.local
  port: 5432
  name: app_prod
logging:
  level: warn
persistence:
  enabled: true
  size: 50Gi
EOF
```

### Step 3: Deploy to Multiple Environments
```bash
# Development deployment
kubectl create namespace dev
helm install app-dev multi-env-app -f multi-env-app/values-dev.yaml -n dev

# Staging deployment
kubectl create namespace staging
helm install app-staging multi-env-app -f multi-env-app/values-staging.yaml -n staging

# Production deployment
kubectl create namespace prod
helm install app-prod multi-env-app -f multi-env-app/values-prod.yaml -n prod
```

### Step 4: Verify Deployments
```bash
# Check all releases
helm list -A

# Compare configurations
helm get values app-dev -n dev
helm get values app-staging -n staging
helm get values app-prod -n prod

# Verify pods and resources
kubectl get pods -n dev
kubectl get pods -n staging
kubectl get pods -n prod
```

**Expected Output:**
```
NAMESPACE  NAME        STATUS    APP
dev        app-dev     deployed  multi-env-app
staging    app-staging deployed  multi-env-app
prod       app-prod    deployed  multi-env-app

Pod counts: dev=1, staging=2, prod=3
Resource limits scale by environment
```

### Step 5: Cleanup
```bash
# Uninstall releases
helm uninstall app-dev -n dev
helm uninstall app-staging -n staging
helm uninstall app-prod -n prod

# Delete namespaces
kubectl delete namespace dev staging prod

# Remove chart
rm -rf multi-env-app
```

## Validation
- `helm get values` shows correct environment configuration
- Pod replicas match environment specification
- Resource limits scale appropriately per environment
- Database connections use correct hosts
- Logging levels differ by environment

## Cleanup
```bash
helm uninstall app-dev -n dev
helm uninstall app-staging -n staging
helm uninstall app-prod -n prod
kubectl delete namespace dev staging prod
rm -rf multi-env-app
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Hardcoded values in templates | Move all to values.yaml |
| Wrong environment deployed | Verify values file with `-f` flag |
| Secrets in plain text | Use Kubernetes secrets, not values |
| No base values file | Create values.yaml with common defaults |
| Values file order wrong | Put base file first: `-f values.yaml -f values-prod.yaml` |

## Troubleshooting

**Problem:** Values not applying
```bash
# Solution: Verify which values file is active
helm get values <release>
helm template <chart> -f values-prod.yaml --debug
```

**Problem:** Environment variables not available in pod
```bash
# Solution: Check if env vars in values are rendered
kubectl describe pod <pod-name>
kubectl exec <pod-name> -- env | grep APP
```

**Problem:** Different config deployed than expected
```bash
# Solution: Compare released values with file
diff <(helm get values myapp) values-prod.yaml
```

## Next Steps
- Explore **05-helm-testing-and-linting** for chart validation
- Implement secret management solutions (Sealed Secrets, External Secrets)
- Design GitOps workflows for value management
- Learn advanced value override patterns
