# 03-helm-templating-and-functions: Solutions

## Exercise 1: Master Template Syntax Basics

**Solution:**
```bash
# Create template with basic syntax
cat > test.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Values.appName }}
  namespace: {{ .Release.Namespace }}
data:
  chart: {{ .Chart.Name }}
  release: {{ .Release.Name }}
  image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
EOF

# Create values
cat > values.yaml << 'EOF'
appName: myapp
image:
  repository: nginx
  tag: "1.21"
EOF

# Render template
helm template myrelease . --values values.yaml
```

**Explanation:** Template syntax uses `{{ }}` for expressions. Variables use dot notation to access values and chart metadata.

---

## Exercise 2: Implement Built-in Functions

**Solution:**
```bash
cat > template.yaml << 'EOF'
data:
  name: {{ .Values.appName | quote }}
  upper: {{ .Values.appName | upper }}
  lower: {{ .Values.appName | lower }}
  default: {{ .Values.optional | default "fallback" }}
  truncated: {{ .Values.longName | trunc 10 }}
  combined: {{ .Values.appName | upper | quote }}
EOF

cat > values.yaml << 'EOF'
appName: MyApp
longName: very-long-application-name
EOF

# Render
helm template . --values values.yaml
# Output: name: "MyApp", upper: MYAPP, lower: myapp
```

**Explanation:** Functions transform values. Pipe operator `|` chains functions left-to-right. Each function takes previous output as input.

---

## Exercise 3: Create Conditional Templates

**Solution:**
```bash
cat > template.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  template:
    spec:
      containers:
      - name: app
        {{- if eq .Values.environment "prod" }}
        resources:
          limits:
            memory: "512Mi"
        {{- else }}
        resources:
          limits:
            memory: "256Mi"
        {{- end }}
EOF

cat > values.yaml << 'EOF'
environment: prod
replicaCount: 3
autoscaling:
  enabled: false
EOF

# Template renders different based on values
helm template . --values values.yaml
```

**Explanation:** `if` conditions include/exclude blocks. `eq` compares values. `not` negates. Multiple conditions possible with `and`/`or`.

---

## Exercise 4: Design Loops and Iterations

**Solution:**
```bash
cat > configmap.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  {{- range $key, $value := .Values.config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
  hosts: |
    {{- range .Values.hosts }}
    - {{ . }}
    {{- end }}
EOF

cat > values.yaml << 'EOF'
config:
  DEBUG: "true"
  LOG_LEVEL: info
  MAX_CONNECTIONS: "100"
hosts:
  - app.example.com
  - api.example.com
EOF

# Render with loops
helm template . --values values.yaml
# ConfigMap has all config entries and hosts listed
```

**Explanation:** `range` iterates collections. Map iteration gives `$key, $value`. Array iteration gives values. Loop scope uses `$` for parent context.

---

## Exercise 5: Implement Named Templates

**Solution:**
```bash
cat > _helpers.tpl << 'EOF'
{{- define "common.labels" -}}
app: {{ .Chart.Name }}
release: {{ .Release.Name }}
{{- end }}

{{- define "common.selector" -}}
app: {{ .Chart.Name }}
{{- end }}

{{- define "common.name" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 }}
{{- end }}
EOF

cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "common.name" . }}
  labels:
    {{- include "common.labels" . | nindent 4 }}
spec:
  selector:
    matchLabels:
      {{- include "common.selector" . | nindent 6 }}
EOF

# Named templates render and indent properly
helm template . --debug
```

**Explanation:** `define` creates named templates. `include` calls them and processes output. Used for reusable components and consistency.

---

## Exercise 6: Handle Complex Data Structures

**Solution:**
```bash
cat > config.yaml << 'EOF'
apiVersion: v1
kind: ConfigMap
data:
  labels.yaml: |
    {{- .Values.labels | toYaml | nindent 4 }}
  annotations.yaml: |
    {{- .Values.annotations | toYaml | nindent 4 }}
  env-vars: |
    {{- range $k, $v := .Values.envVars }}
    - name: {{ $k }}
      value: {{ $v | quote }}
    {{- end }}
EOF

cat > values.yaml << 'EOF'
labels:
  app: myapp
  environment: prod
annotations:
  owner: devops
  contact: team@example.com
envVars:
  DATABASE_HOST: db.local
  DATABASE_PORT: "5432"
EOF

# Preserves YAML structure
helm template . --values values.yaml
```

**Explanation:** `toYaml` converts objects to YAML format. `nindent` ensures proper indentation. Good for nested config and metadata.

---

## Exercise 7: String and List Manipulation

**Solution:**
```bash
cat > template.yaml << 'EOF'
data:
  appname-safe: {{ .Values.appName | lower | replace " " "-" }}
  has-db: {{ contains "database" .Values.features | quote }}
  parts: {{ .Values.name | split "-" | join "_" }}
  first-host: {{ first .Values.hosts }}
  last-host: {{ last .Values.hosts }}
  tag-list: {{ join "," .Values.tags }}
EOF

cat > values.yaml << 'EOF'
appName: "My App"
features: "database,cache,queue"
name: "my-app-name"
hosts:
  - app1.com
  - app2.com
tags:
  - prod
  - critical
EOF

helm template . --values values.yaml
```

**Explanation:** String functions manipulate text. List functions work with arrays. `split` creates lists, `join` combines them.

---

## Exercise 8: Use Comparison and Logical Functions

**Solution:**
```bash
cat > template.yaml << 'EOF'
data:
  is-prod: {{ eq .Values.environment "prod" | quote }}
  is-not-dev: {{ ne .Values.environment "dev" | quote }}
  has-high-resources: {{ and (ge .Values.replicas 3) (gt .Values.memory 512) | quote }}
  is-enabled: {{ or .Values.feature1 .Values.feature2 | quote }}
  should-scale: {{ and (eq .Values.environment "prod") (not .Values.disabled) | quote }}
EOF

cat > values.yaml << 'EOF'
environment: prod
replicas: 5
memory: 1024
feature1: false
feature2: true
disabled: false
EOF

helm template . --values values.yaml
```

**Explanation:** Comparison functions return boolean. Logical functions combine conditions. Used in if statements and data generation.

---

## Exercise 9: Apply Math and Type Functions

**Solution:**
```bash
cat > template.yaml << 'EOF'
data:
  calculated-memory: {{ mul .Values.replicaCount 512 | quote }}
  cpu-count: {{ add .Values.baseCpu .Values.extraCpu | quote }}
  percentage: {{ div 100 .Values.replicas | quote }}
  is-int: {{ typeOf .Values.replicaCount | quote }}
  as-float: {{ .Values.percentage | asFloat | quote }}
  computed-value: {{ sub 1000 (mul 10 .Values.index) | quote }}
EOF

cat > values.yaml << 'EOF'
replicaCount: 3
baseCpu: 2
extraCpu: 1
percentage: 33.33
index: 5
EOF

helm template . --values values.yaml
# Output: memory: "1536", cpu-count: "3", percentage: "33"
```

**Explanation:** Math functions enable computed configuration. Type functions help with validation. Useful for dynamic resource allocation.

---

## Exercise 10: Validate Complete Templating Implementation

**Solution:**
```bash
# Create complete chart
helm create template-full

# Create complex template using all features
cat > template-full/templates/deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "template-full.fullname" . }}
spec:
  {{- if not .Values.autoscaling }}
  replicas: {{ .Values.replicas }}
  {{- end }}
  template:
    metadata:
      labels:
        {{- include "template-full.labels" . | nindent 8 }}
    spec:
      containers:
      - name: app
        image: {{ .Values.image | quote }}
        {{- if .Values.resources }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
        {{- end }}
        env:
        {{- range $k, $v := .Values.env }}
        - name: {{ $k }}
          value: {{ $v | quote }}
        {{- end }}
EOF

# Test
helm lint template-full
helm template template-full
kubectl apply -f <(helm template template-full) --dry-run=server

# Install and verify
kubectl create namespace template-test
helm install full template-full -n template-test
kubectl get deployment -n template-test
helm uninstall full -n template-test
```

**Explanation:** Production charts combine all template features. Rigorous testing ensures correctness before deployment.
