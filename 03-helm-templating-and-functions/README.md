# 03-helm-templating-and-functions

## What You'll Learn
- Master Helm template syntax (Go templating)
- Use built-in Helm functions and filters
- Implement conditional logic in templates
- Create loops and iterations
- Use named templates and includes
- Handle data structures and transformations

## Prerequisites
- Completed 02-helm-chart-design module
- Understanding of YAML and JSON
- Familiarity with template concepts
- Go basics (beneficial but not required)

## Key Concepts

### Template Syntax
Go template language with Helm-specific extensions for dynamic manifest generation.

### Functions & Filters
Built-in and Helm functions to transform and manipulate template values.

### Conditionals
if/else logic to include or exclude template sections based on values.

### Loops
range iteration over lists, maps, and other data structures.

### Named Templates
Reusable template blocks called with `include` or `template`.

## Hands-on Lab: Advanced Templating

### Step 1: Create Templating Chart
```bash
# Create chart
helm create template-demo

# Remove default templates
rm -rf template-demo/templates/*.yaml
rm -rf template-demo/templates/tests/
```

### Step 2: Create Helper Template
```bash
cat > template-demo/templates/_helpers.tpl << 'EOF'
{{- define "template-demo.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "template-demo.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
EOF
```

### Step 3: Create Deployment with Conditions
```bash
cat > template-demo/templates/deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "template-demo.fullname" . }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  template:
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
        {{- if .Values.livenessProbe }}
        livenessProbe:
          {{- toYaml .Values.livenessProbe | nindent 10 }}
        {{- end }}
EOF
```

### Step 4: Create ConfigMap with Loop
```bash
cat > template-demo/templates/configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "template-demo.fullname" . }}-config
data:
  {{- range $key, $value := .Values.config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
EOF
```

### Step 5: Test Templates
```bash
# Create values.yaml
cat > template-demo/values.yaml << 'EOF'
replicaCount: 2
image:
  repository: nginx
  tag: "1.21"
autoscaling:
  enabled: false
config:
  APP_NAME: MyApp
  DEBUG: "false"
EOF

# Validate
helm lint template-demo
helm template template-demo

# Install
kubectl create namespace template-test
helm install template-release template-demo -n template-test
kubectl get all -n template-test
```

**Expected Output:**
```
==> Linting template-demo
[OK] all templates validate

Deployment created with 2 replicas
ConfigMap with config values applied
```

## Validation
- `helm lint` passes without errors
- `helm template` generates valid YAML
- Conditionals evaluate correctly
- Loops iterate through values
- Named templates render properly

## Cleanup
```bash
helm uninstall template-release -n template-test
kubectl delete namespace template-test
rm -rf template-demo
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Template syntax errors | Run `helm template --debug` |
| Indentation issues | Use `nindent` filter |
| Undefined variables | Check scope in loops |
| Missing quotes | Use `quote` filter |
| Function name errors | Verify in Helm function reference |

## Troubleshooting

**Problem:** "undefined variable" error
```bash
# In loops, use $ to access root context
{{- range .Values.items }}
  name: {{ $.Release.Name }}-{{ .name }}
{{- end }}
```

**Problem:** YAML indentation broken
```bash
# Correct indentation usage
{{ toYaml .Values | nindent 2 }}
```

**Problem:** Conditional not working
```bash
# Check syntax
{{- if .Values.enabled }}
{{- if eq .Values.env "prod" }}
```

## Next Steps
- Explore **04-helm-values-and-environments**
- Learn chart testing and hooks
- Implement pre/post-installation hooks
- Design for CI/CD pipelines
