# 02-helm-chart-design: Solutions

## Exercise 1: Plan Chart Directory Structure

**Solution:**
```bash
# Create chart
helm create mybank-app

# View structure
tree mybank-app

# Key directories:
# Chart.yaml - metadata
# values.yaml - default values
# templates/ - manifest templates
# charts/ - dependencies
# .helmignore - files to exclude

# Optional:
# crds/ - custom resource definitions
# _test/ - test manifests
```

**Explanation:** Standard Helm chart structure follows conventions for tool compatibility. Optional directories support advanced use cases like CRDs.

---

## Exercise 2: Implement Hierarchical Values

**Solution:**
```bash
# Edit values.yaml
cat > mybank-app/values.yaml << 'EOF'
global:
  environment: dev
  domain: example.com

image:
  repository: mybank
  tag: "1.0"
  pullPolicy: IfNotPresent

deployment:
  replicaCount: 2

service:
  type: ClusterIP
  port: 80

persistence:
  enabled: false
  size: 10Gi

ingress:
  enabled: false
EOF

# Create dev values
cat > mybank-app/values-dev.yaml << 'EOF'
global:
  environment: dev
deployment:
  replicaCount: 1
EOF

# Create prod values
cat > mybank-app/values-prod.yaml << 'EOF'
global:
  environment: prod
deployment:
  replicaCount: 3
persistence:
  enabled: true
ingress:
  enabled: true
EOF

# Install with dev values
helm install mybank mybank-app --values mybank-app/values-dev.yaml
```

**Explanation:** Hierarchical structure improves readability. Environment-specific files override defaults cleanly.

---

## Exercise 3: Create Chart Dependencies

**Solution:**
```bash
# Edit Chart.yaml
cat >> mybank-app/Chart.yaml << 'EOF'
dependencies:
  - name: postgresql
    version: "12.1.5"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
EOF

# Update dependencies
helm dependency update mybank-app/

# Add PostgreSQL values
cat >> mybank-app/values.yaml << 'EOF'
postgresql:
  enabled: true
  auth:
    username: bankuser
    password: changeme
    database: bankdb
EOF

# Verify dependency
helm dependency list mybank-app/
ls -la mybank-app/charts/
```

**Explanation:** Dependencies are declared in Chart.yaml. `helm dependency update` downloads them. Sub-chart values nest under parent chart values.

---

## Exercise 4: Implement Helper Templates

**Solution:**
```bash
# Create _helpers.tpl
cat > mybank-app/templates/_helpers.tpl << 'EOF'
{{- define "mybank.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "mybank.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{- define "mybank.labels" -}}
app.kubernetes.io/name: {{ include "mybank.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
EOF

# Use in deployment.yaml
# metadata:
#   name: {{ include "mybank.fullname" . }}
#   labels:
#     {{- include "mybank.labels" . | nindent 4 }}
```

**Explanation:** Helper templates reduce duplication. They're called with `include` function. Define once, use multiple times.

---

## Exercise 5: Design for RBAC

**Solution:**
```bash
# Create ServiceAccount
cat > mybank-app/templates/serviceaccount.yaml << 'EOF'
{{- if .Values.rbac.create }}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "mybank.fullname" . }}
{{- end }}
EOF

# Create ClusterRole
cat > mybank-app/templates/clusterrole.yaml << 'EOF'
{{- if .Values.rbac.create }}
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: {{ include "mybank.fullname" . }}
rules:
  - apiGroups: [""]
    resources: ["pods", "services"]
    verbs: ["get", "list", "watch"]
{{- end }}
EOF

# Add to values.yaml
cat >> mybank-app/values.yaml << 'EOF'
rbac:
  create: true
EOF
```

**Explanation:** RBAC resources enforce security. Conditional creation allows for different deployment scenarios.

---

## Exercise 6: Plan Persistence Strategy

**Solution:**
```bash
# Create PVC template
cat > mybank-app/templates/pvc.yaml << 'EOF'
{{- if .Values.persistence.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "mybank.fullname" . }}-pvc
spec:
  accessModes:
    {{- range .Values.persistence.accessModes }}
    - {{ . }}
    {{- end }}
  storageClassName: {{ .Values.persistence.storageClassName }}
  resources:
    requests:
      storage: {{ .Values.persistence.size }}
{{- end }}
EOF

# Add storage values
cat >> mybank-app/values.yaml << 'EOF'
persistence:
  enabled: false
  storageClassName: "standard"
  accessModes:
    - ReadWriteOnce
  size: 10Gi
EOF
```

**Explanation:** Conditional persistence allows optionally stateless deployments. accessModes and storageClassName are configurable per platform.

---

## Exercise 7: Design Multi-Cloud Compatibility

**Solution:**
```bash
# Cloud-agnostic service account
cat > mybank-app/templates/serviceaccount.yaml << 'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "mybank.fullname" . }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
EOF

# Add cloud-specific annotations in values
# values-aws.yaml
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::ACCOUNT:role/ROLE

# values-gcp.yaml
serviceAccount:
  annotations:
    iam.gke.io/gcp-service-account: sa@project.iam.gserviceaccount.com

# Install with cloud-specific values
helm install mybank mybank-app --values mybank-app/values-aws.yaml
```

**Explanation:** Cloud providers use different annotation systems. Use cloud-specific value files while keeping templates cloud-agnostic.

---

## Exercise 8: Create Chart Tests

**Solution:**
```bash
# Create test directory
mkdir -p mybank-app/templates/tests

# Create test pod
cat > mybank-app/templates/tests/test-connection.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mybank.fullname" . }}-test"
  labels:
    {{- include "mybank.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "mybank.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
EOF

# Run test
helm install mybank mybank-app -n test
helm test mybank -n test
```

**Explanation:** Test hooks validate deployments. `helm.sh/hook: test` annotation marks pods as tests. Tests help verify configuration correctness.

---

## Exercise 9: Plan Post-Installation Hooks

**Solution:**
```bash
# Pre-install hook for database init
cat > mybank-app/templates/job-db-init.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-db-init"
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  template:
    spec:
      containers:
      - name: db-init
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        command: ["/bin/sh", "-c"]
        args: ["python manage.py migrate"]
      restartPolicy: Never
EOF

# Pre-delete hook for cleanup
cat > mybank-app/templates/job-cleanup.yaml << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-delete-policy": before-hook-creation
spec:
  # ... cleanup logic
EOF
```

**Explanation:** Hooks execute at chart lifecycle points. Weights control execution order. Delete policies prevent orphaned resources.

---

## Exercise 10: Validate Complete Chart Design

**Solution:**
```bash
# Lint with strict mode
helm lint mybank-app --strict

# Generate templates
helm template mybank-app > manifests.yaml

# Dry-run
kubectl apply -f manifests.yaml --dry-run=server

# Create namespace
kubectl create namespace chart-validation

# Install
helm install mybank mybank-app -n chart-validation

# Verify
kubectl get all -n chart-validation
kubectl logs -l app=mybank -n chart-validation
helm status mybank -n chart-validation

# Test
helm test mybank -n chart-validation

# Uninstall
helm uninstall mybank -n chart-validation
kubectl delete namespace chart-validation
```

**Explanation:** Complete validation catches errors before production. Dry-run and test environments prevent surprises in real deployments.
