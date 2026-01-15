# 12-argo-rollouts-and-progressive-delivery: Cheatsheet

## Argo Rollouts Installation

```bash
# Install controller
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f \
  https://github.com/argoproj/argo-rollouts/releases/download/v1.4.1/install.yaml

# Verify installation
kubectl get deployment -n argo-rollouts
kubectl get crd | grep rollout
```

## Rollout Resource Template

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
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
        image: myapp:v1
        ports:
        - containerPort: 8080
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 5m}
      - setWeight: 50
      - pause: {duration: 5m}
```

## Deployment Strategies

| Strategy | Usage | Advantages | Disadvantages |
|----------|-------|-----------|---|
| Canary | Gradual rollout | Safe, monitored | Slow |
| Blue-Green | Instant switch | Fast rollback | Resource intensive |
| Rolling | Sequential update | Resource efficient | Slower feedback |

## Key Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl apply -f rollout.yaml` | Create rollout | Deploys initial version |
| `kubectl set image rollout/<name> <container>=<image>` | Update image | Triggers strategy |
| `kubectl rollout status rollout/<name>` | Check progress | Shows weight and steps |
| `kubectl describe rollout <name>` | Detailed info | Shows conditions and events |
| `kubectl delete rollout <name>` | Delete rollout | Removes all pods |

## Canary Configuration

```yaml
strategy:
  canary:
    steps:
    - setWeight: 10          # 10% of traffic
    - pause: {duration: 5m}  # Wait 5 minutes
    - setWeight: 25
    - pause: {duration: 5m}
    - setWeight: 50
    - pause: {}              # Manual approval
    - setWeight: 100
```

## Blue-Green Configuration

```yaml
strategy:
  blueGreen:
    activeSlot: blue        # Current active version
    autoPromotionEnabled: false
    autoPromotionSeconds: 30
    prePromotionAnalysis:
      templates:
      - name: analysis
```

## Analysis Template

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate
spec:
  metrics:
  - name: error-rate
    interval: 60s
    consecutiveErrorLimit: 2
    provider:
      prometheus:
        address: http://prometheus:9090
        query: |
          rate(errors_total[5m])
    successCriteria: "{{ result < 0.05 }}"
```

## Monitoring Commands

| Command | Purpose |
|---------|---------|
| `kubectl get rollout -w` | Watch rollout changes |
| `kubectl describe rollout <name>` | Show current status |
| `kubectl logs -n argo-rollouts` | Controller logs |
| `kubectl get analysisrun` | Show analysis runs |
| `kubectl describe analysisrun <name>` | Analysis details |

## VirtualService (Istio)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - route:
    - destination:
        host: myapp-stable
        port:
          number: 8080
      weight: 90
    - destination:
        host: myapp-canary
        port:
          number: 8080
      weight: 10
```

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| Check rollout status | `kubectl describe rollout <name>` |
| View controller logs | `kubectl logs -n argo-rollouts deployment/argo-rollouts` |
| Check analysis | `kubectl get analysisrun; kubectl logs <analysisrun>` |
| Verify service | `kubectl get svc; kubectl describe svc <name>` |
| Watch progress | `kubectl rollout status rollout/<name> --timeout=10m` |

## Common Issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Rollout stuck | Analysis failure | Check metrics/thresholds |
| Pods not updating | Image not pulling | Check image registry |
| Traffic not splitting | Service issue | Verify service labels |
| Metrics not found | Wrong query | Check Prometheus format |

## Rollout Lifecycle

1. Create Rollout (all replicas stable version)
2. Update image (canary pods created)
3. Traffic shift (percentage increases)
4. Pause for analysis (metrics collected)
5. Auto-promotion or manual approval
6. Complete (all replicas on new version)
7. Or rollback (revert to stable)

## Progressive Delivery Flow

```
Start: v1 (100%)
  ↓
Canary: v1 (90%), v2 (10%)
  ↓
Progress: v1 (50%), v2 (50%)
  ↓
Promote: v1 (0%), v2 (100%)
  ↓
Complete: All v2
```

## Best Practices

- Use metrics-based analysis
- Start with small percentages (5-10%)
- Pause between steps for validation
- Monitor error rates and latency
- Implement automatic rollback on failure
- Use service mesh for fine-grained control
- Maintain fast rollback capability
- Document promotion criteria
