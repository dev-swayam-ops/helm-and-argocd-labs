# 04-helm-values-and-environments: Solutions

## Exercise 1: Create Multi-Environment Values Files

**Solution:**
```bash
# Base values.yaml
cat > values.yaml << 'EOF'
replicaCount: 1
image:
  repository: myapp
  tag: "latest"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  limits:
    memory: "256Mi"
    cpu: "200m"
logging:
  level: info
EOF

# Development values
cat > values-dev.yaml << 'EOF'
replicaCount: 1
image:
  tag: "dev"
  pullPolicy: Always
resources:
  limits:
    memory: "128Mi"
    cpu: "100m"
logging:
  level: debug
EOF

# Production values
cat > values-prod.yaml << 'EOF'
replicaCount: 3
image:
  tag: "latest"
  pullPolicy: IfNotPresent
resources:
  limits:
    memory: "1Gi"
    cpu: "1000m"
logging:
  level: warn
persistence:
  enabled: true
EOF
```

**Explanation:** Base values provide common defaults. Environment files override only what differs, keeping values DRY.

---

## Exercise 2: Override Values at Installation

**Solution:**
```bash
# Single value override
helm install app myapp --set replicaCount=5

# Values file override
helm install app myapp -f values-prod.yaml

# Multiple file override
helm install app myapp -f values.yaml -f values-prod.yaml

# String override (prevents parsing)
helm install app myapp --set-string image.tag=v1.2.3

# Verify applied values
helm get values app
# Shows all merged values in effect
```

**Explanation:** -f applies entire file, --set overrides single values. Order matters: first file sets base, later files override.

---

## Exercise 3: Implement Environment-Specific Secrets

**Solution:**
```bash
# Dev secrets file
cat > values-dev-secrets.yaml << 'EOF'
secrets:
  database_password: devpassword123
  api_key: dev-key-xxx
  jwt_secret: dev-secret
EOF

# Prod secrets file (with real secrets)
cat > values-prod-secrets.yaml << 'EOF'
secrets:
  database_password: prod-secure-password
  api_key: prod-key-yyy
  jwt_secret: prod-secret
EOF

# Template reference
cat > templates/deployment.yaml << 'EOF'
env:
  - name: DB_PASSWORD
    value: {{ .Values.secrets.database_password | quote }}
  - name: API_KEY
    value: {{ .Values.secrets.api_key | quote }}
EOF

# Deploy with secrets
helm install app myapp -f values-dev-secrets.yaml -n dev
helm install app myapp -f values-prod-secrets.yaml -n prod

# Verify
kubectl get pod -o jsonpath='{.items[0].spec.containers[0].env}' -n dev
```

**Explanation:** Secrets files separate sensitive data. Store real secrets in secure vault, reference them in values. Never commit secrets.

---

## Exercise 4: Design Values Hierarchy

**Solution:**
```bash
cat > values.yaml << 'EOF'
global:
  domain: example.com
  environment: dev

image:
  repository: myapp
  tag: "latest"

database:
  host: localhost
  port: 5432
  name: appdb

cache:
  enabled: true
  redis_host: redis.local

logging:
  level: info
  format: json
EOF

# Override with prod values
cat > values-prod.yaml << 'EOF'
global:
  environment: prod

image:
  tag: "v1.0.0"

database:
  host: prod-db.cloud.local
  name: app_prod

logging:
  level: warn
EOF

# Merged result in Helm
helm template myapp -f values.yaml -f values-prod.yaml
# Shows inherited and overridden values
```

**Explanation:** Nested structure organizes related values. Child values inherit parent section, only override needed fields.

---

## Exercise 5: Implement Feature Flags

**Solution:**
```bash
# Base values with features
cat > values.yaml << 'EOF'
features:
  cache:
    enabled: true
  metrics:
    enabled: true
  audit:
    enabled: false
  backup:
    enabled: false
EOF

# Production enables all
cat > values-prod.yaml << 'EOF'
features:
  audit:
    enabled: true
  backup:
    enabled: true
EOF

# Template conditional
cat > templates/deployment.yaml << 'EOF'
{{- if .Values.features.cache.enabled }}
- name: CACHE_ENABLED
  value: "true"
{{- end }}
{{- if .Values.features.audit.enabled }}
- name: AUDIT_ENABLED
  value: "true"
{{- end }}
EOF

# Deploy with different features
helm install app myapp -f values-dev.yaml  # Fewer features
helm install app myapp -f values-prod.yaml # All features
```

**Explanation:** Feature flags allow same chart with different capabilities. Useful for gradual rollout or cost optimization.

---

## Exercise 6: Create Values Template Documentation

**Solution:**
```bash
cat > values.yaml << 'EOF'
# Global configuration
global:
  # Environment name: dev, staging, prod
  environment: dev
  # Base domain for all services
  domain: example.com

# Image configuration
image:
  # Docker image repository
  repository: myapp
  # Image tag/version
  tag: "latest"
  # Image pull policy: Always, IfNotPresent, Never
  pullPolicy: IfNotPresent

# Resource requests and limits
resources:
  # Maximum resources
  limits:
    memory: "256Mi"  # Memory limit
    cpu: "200m"      # CPU limit
  # Minimum guaranteed resources
  requests:
    memory: "128Mi"  # Memory request
    cpu: "100m"      # CPU request
EOF

# Create values schema
cat > values.schema.json << 'EOF'
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "environment": {
      "type": "string",
      "enum": ["dev", "staging", "prod"]
    },
    "replicas": {
      "type": "integer",
      "minimum": 1
    }
  },
  "required": ["environment"]
}
EOF
```

**Explanation:** Well-documented values help users. Schema validates values format. Comments explain purpose and valid options.

---

## Exercise 7: Implement Value Validation

**Solution:**
```bash
# Template with validation
cat > templates/deployment.yaml << 'EOF'
{{- if not .Values.image.repository }}
  {{ fail "image.repository is required" }}
{{- end }}

{{- if not .Values.resources.requests.memory }}
  {{ fail "resources.requests.memory is required" }}
{{- end }}

{{- $validEnvs := list "dev" "staging" "prod" }}
{{- if not (has .Values.environment $validEnvs) }}
  {{ fail (printf "environment must be dev, staging, or prod. Got: %s" .Values.environment) }}
{{- end }}

{{- if gt (int .Values.replicaCount) 10 }}
  {{ fail "replicaCount must not exceed 10" }}
{{- end }}
EOF

# Test validation
helm template myapp                          # Fails with missing values
helm template myapp -f valid-values.yaml   # Succeeds
```

**Explanation:** Validation prevents deployment with invalid configuration. `fail` function stops rendering with error message.

---

## Exercise 8: Design GitOps Value Management

**Solution:**
```bash
# Directory structure
# repo/
# ├── helm/
# │   ├── myapp/
# │   │   ├── Chart.yaml
# │   │   └── templates/
# │   └── values/
# │       ├── values-base.yaml
# │       ├── values-dev.yaml
# │       ├── values-staging.yaml
# │       └── values-prod.yaml
# └── kustomization.yaml

# Base values
cat > helm/values/values-base.yaml << 'EOF'
replicaCount: 1
image:
  repository: myapp
  tag: "latest"
EOF

# Environment overlays
cat > helm/values/kustomization.yaml << 'EOF'
resources:
- values-base.yaml
patchesJson6902:
- target:
    version: v1
    kind: ConfigMap
  patch: |-
    - op: replace
      path: /data/environment
      value: staging
EOF

# GitOps workflow
# 1. Commit values to repo
# 2. ArgoCD watches repo
# 3. Auto-deploy on commit
# 4. Promotion: dev → staging → prod
```

**Explanation:** GitOps stores values in Git. Changes tracked and auditable. Automated sync ensures cluster matches Git.

---

## Exercise 9: Create Values Comparison Tool

**Solution:**
```bash
# Export all values
helm get values app-dev > dev-values.yaml
helm get values app-staging > staging-values.yaml
helm get values app-prod > prod-values.yaml

# Compare files
diff dev-values.yaml staging-values.yaml > comparison.txt

# Show only differences
diff --unified=0 dev-values.yaml staging-values.yaml

# Generate summary
echo "=== Dev vs Staging ===" && \
diff dev-values.yaml staging-values.yaml | grep -E "^[<>]" && \
echo "=== Staging vs Prod ===" && \
diff staging-values.yaml prod-values.yaml | grep -E "^[<>]"

# Check for drift
echo "Original:" && helm template myapp -f values-prod.yaml | md5sum
echo "Current:" && helm get manifest app-prod | md5sum
```

**Explanation:** Comparison reveals unintended differences. Useful for identifying config drift and validating deployments.

---

## Exercise 10: Implement Complete Multi-Environment Workflow

**Solution:**
```bash
# Create chart with values
helm create myapp

# Create environment values
cat > values-all.yaml << 'EOF'
# Contains: dev, staging, prod configurations
EOF

# Deploy workflow
deploy_env() {
  local env=$1
  local namespace=$2
  kubectl create namespace $namespace
  helm install app-$env myapp \
    -f myapp/values.yaml \
    -f myapp/values-$env.yaml \
    -n $namespace
}

# Deploy all environments
deploy_env dev dev
deploy_env staging staging
deploy_env prod prod

# Verify
helm list -A
helm get values app-dev -n dev
helm get values app-staging -n staging
helm get values app-prod -n prod

# Test environment-specific functionality
kubectl get pods -A
kubectl describe deployment -n dev
kubectl logs -n prod

# Cleanup
helm uninstall app-dev -n dev
helm uninstall app-staging -n staging
helm uninstall app-prod -n prod
```

**Explanation:** Complete workflow demonstrates multi-environment best practices. Consistent process prevents errors.
