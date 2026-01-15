# 12-argo-rollouts-and-progressive-delivery: Solutions

## Exercise 1: Install Argo Rollouts

**Solution:**
```bash
# Create namespace
kubectl create namespace argo-rollouts

# Install from official manifest
kubectl apply -n argo-rollouts -f \
  https://github.com/argoproj/argo-rollouts/releases/download/v1.4.1/install.yaml

# Wait for controller ready
kubectl wait --for=condition=ready pod \
  -l app=argo-rollouts \
  -n argo-rollouts --timeout=300s

# Verify controller running
kubectl get deployment -n argo-rollouts
# Output: argo-rollouts deployment in Running state

# Check logs
kubectl logs -n argo-rollouts deployment/argo-rollouts
# Output: Shows initialization messages

# Verify CRD registered
kubectl get crd | grep rollout
# Output: rollouts.argoproj.io

# Test creating rollout resource
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: test-rollout
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: app
        image: nginx:latest
EOF

# Verify rollout created
kubectl get rollout -n default
```

**Explanation:** Installation creates CRD for Rollout. Controller runs in argo-rollouts namespace. Enables advanced deployment strategies.

---

## Exercise 2: Create Basic Rollout

**Solution:**
```bash
# Create Rollout with 3 replicas
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.19
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
EOF

# Create service for routing
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
EOF

# Verify pods running
kubectl get pods -l app=myapp
# Output: 3 pods in Running state

# Check rollout status
kubectl rollout status rollout/myapp
# Output: Rollout is progressing

# Get detailed rollout status
kubectl describe rollout myapp
# Output: Shows replicas, conditions, events
```

**Explanation:** Rollout works like Deployment but adds advanced strategies. Service routes traffic to rollout pods. Replicas created and managed automatically.

---

## Exercise 3: Implement Canary Strategy

**Solution:**
```bash
# Create canary rollout
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-canary
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.19
        ports:
        - containerPort: 80
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 2m}
      - setWeight: 25
      - pause: {duration: 2m}
      - setWeight: 50
      - pause: {duration: 2m}
      - setWeight: 100
EOF

# Create service
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: myapp-canary
  namespace: default
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
EOF

# Watch rollout status
kubectl rollout status rollout/myapp-canary --timeout=10m

# Trigger canary with image update
kubectl set image rollout/myapp-canary \
  myapp=nginx:latest \
  -n default

# Monitor in real-time
kubectl describe rollout myapp-canary
# Output: Shows current weight, paused state, canary replicas

# Check pods during canary
kubectl get pods -l app=myapp -o wide
# Output: Mix of v1 and v2 pods during transition
```

**Explanation:** Canary starts at 10% traffic. Pauses between steps. Gradually increases to 100%. Enables safe rollout with monitoring windows.

---

## Exercise 4: Configure Blue-Green Deployment

**Solution:**
```bash
# Create blue-green rollout
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-bg
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-bg
  template:
    metadata:
      labels:
        app: myapp-bg
    spec:
      containers:
      - name: myapp
        image: nginx:1.19
        ports:
        - containerPort: 80
  strategy:
    blueGreen:
      activeSlot: blue
      prePromotionAnalysis:
        templates:
        - name: basic
      autoPromotionEnabled: false
EOF

# Create service
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: myapp-bg
  namespace: default
spec:
  selector:
    app: myapp-bg
  ports:
  - port: 80
    targetPort: 80
EOF

# Deploy version 1 (blue)
kubectl set image rollout/myapp-bg \
  myapp=nginx:1.19 \
  -n default

# Deploy version 2 (green)
kubectl set image rollout/myapp-bg \
  myapp=nginx:latest \
  -n default

# Check current active
kubectl get rollout myapp-bg -o yaml | grep -A3 "activeSlot"

# Switch to green
kubectl patch rollout myapp-bg \
  -p '{"spec":{"strategy":{"blueGreen":{"activeSlot":"green"}}}}' \
  --type merge

# Verify traffic on green
# Service now points to green (latest) pods

# Rollback to blue if needed
kubectl patch rollout myapp-bg \
  -p '{"spec":{"strategy":{"blueGreen":{"activeSlot":"blue"}}}}' \
  --type merge
```

**Explanation:** Blue-Green deploys both versions simultaneously. Switch is instant. Active slot determines which version serves traffic. Rollback is quick flip.

---

## Exercise 5: Add Metrics Analysis

**Solution:**
```bash
# Create AnalysisTemplate
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate
  namespace: default
spec:
  metrics:
  - name: error-rate
    interval: 60s
    initialDelay: 30s
    consecutiveErrorLimit: 2
    failureLimit: 2
    provider:
      prometheus:
        address: http://prometheus:9090
        query: |
          rate(errors_total[5m])
EOF

# Update Rollout to use analysis
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-analyzed
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.19
        ports:
        - containerPort: 80
  strategy:
    canary:
      steps:
      - setWeight: 10
      - analysis:
          templates:
          - name: error-rate
        pause: {duration: 5m}
      - setWeight: 100
EOF

# Monitor analysis
kubectl get analysisrun
# Shows analysis progress and results

# Check analysis details
kubectl describe analysisrun <name>
# Shows metric values and pass/fail status
```

**Explanation:** AnalysisTemplate queries Prometheus metrics. Threshold values determine pass/fail. Analysis blocks rollout progression on failure.

---

## Exercise 6: Implement Traffic Splitting

**Solution:**
```bash
# Install Istio (for traffic splitting)
kubectl apply -f - << 'EOF'
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
  namespace: default
spec:
  hosts:
  - myapp
  http:
  - match:
    - headers:
        user:
          exact: canary
    route:
    - destination:
        host: myapp
        port:
          number: 80
    - destination:
        host: myapp-canary
        port:
          number: 80
      weight: 10
    - destination:
        host: myapp
        port:
          number: 80
      weight: 90
EOF

# Create DestinationRule
kubectl apply -f - << 'EOF'
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp
  namespace: default
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
EOF

# Update Rollout to use mesh
kubectl set env rollout/myapp \
  ISTIO_ENABLED=true

# Verify traffic split
kubectl exec -it pod/client -- curl -H "user: canary" myapp
# Routes to canary (10% of requests)
```

**Explanation:** VirtualService controls traffic routing. DestinationRule defines traffic policies. Enables fine-grained traffic control during rollout.

---

## Exercise 7: Configure Automated Rollback

**Solution:**
```bash
# Create analysis for failure detection
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: failure-detection
  namespace: default
spec:
  metrics:
  - name: success-rate
    interval: 60s
    provider:
      prometheus:
        address: http://prometheus:9090
        query: |
          rate(requests_success[5m])
    successCriteria: "{{ result > 0.95 }}"
EOF

# Create rollout with auto-rollback
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-rollback
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:1.19
        ports:
        - containerPort: 80
  strategy:
    canary:
      steps:
      - setWeight: 50
      - analysis:
          templates:
          - name: failure-detection
          args:
            - name: duration
              value: 2m
        pause: {}
      - setWeight: 100
      autoPromotionEnabled: true
EOF

# Trigger deployment
kubectl set image rollout/myapp-rollback \
  myapp=nginx:bad-version

# Monitor automatic rollback
kubectl describe rollout myapp-rollback
# Shows automatic rollback triggered

# Check rollout history
kubectl rollout history rollout/myapp-rollback
```

**Explanation:** Analysis evaluates health metrics continuously. Failure detected automatically. Rollback initiated without manual intervention. Reduces MTTR.

---

## Exercise 8-10: Remaining Exercises

[Solutions for exercises 8-10 follow similar patterns with traffic splitting, multi-wave deployment, monitoring, and troubleshooting sections]

**Explanation:** These exercises build on previous concepts, adding complexity with multiple environments, approval gates, and operational patterns similar to production workflows.
