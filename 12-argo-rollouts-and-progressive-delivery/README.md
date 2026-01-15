# 12-argo-rollouts-and-progressive-delivery

## What You'll Learn
- Argo Rollouts for advanced deployment strategies
- Canary deployments with traffic splitting
- Blue-Green deployment pattern
- Progressive delivery with metrics analysis
- Traffic management and routing
- Automated rollback on failure
- Multi-wave deployment strategies

## Prerequisites
- Completed modules 00-11
- ArgoCD and Kubernetes understanding
- Application deployment knowledge
- Basic networking concepts
- Service mesh basics (optional)
- Metrics collection understanding

## Key Concepts

**Argo Rollouts:** Kubernetes controller for advanced deployments. Replaces standard Deployment with Rollout CRD. Enables canary, blue-green, and rolling strategies.

**Canary Deployment:** Gradually shift traffic to new version. Start small percentage. Monitor metrics. Increase traffic if healthy. Automatic rollback on failure.

**Blue-Green Deployment:** Two identical environments (blue=current, green=new). Deploy new version to green. Switch traffic when ready. Instant rollback by switching back.

**Progressive Delivery:** Deploy gradually with automated quality gates. Traffic splitting based on metrics. Pause on errors. Automated promotion or rollback.

**Metrics Analysis:** Query metrics during deployment. Prometheus, Datadog, New Relic integration. Automatic failure detection. Policy enforcement.

**Traffic Management:** Route percentage of traffic to canary. Use service mesh (Istio, Flagger) or Kubernetes native. Gradual increase from 5% to 100%.

## Hands-on Lab: Canary Deployment with Metrics

**Objective:** Deploy new version using canary strategy with metrics-based rollback

**Steps:**

1. **Install Argo Rollouts:**
```bash
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/download/v1.4.1/install.yaml

# Wait for controller ready
kubectl wait --for=condition=ready pod -l app=argo-rollouts \
  -n argo-rollouts --timeout=300s
```

2. **Create Rollout with canary strategy:**
```bash
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
        image: myapp:v1
        ports:
        - containerPort: 8080
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 5m}
      - setWeight: 25
      - pause: {duration: 5m}
      - setWeight: 50
      - pause: {duration: 5m}
      - setWeight: 75
      - pause: {duration: 5m}
EOF
```

3. **Create service for routing:**
```bash
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
    targetPort: 8080
  type: ClusterIP
EOF
```

4. **Trigger canary deployment:**
```bash
kubectl set image rollout/myapp myapp=myapp:v2 -n default

# Watch rollout progress
kubectl rollout status rollout/myapp -n default --timeout=5m

# Check canary status
kubectl describe rollout myapp -n default
```

5. **Monitor and verify:**
```bash
# Check rollout replicas
kubectl get rollout myapp -o wide

# View detailed status
kubectl get rollout myapp -o yaml | grep -A10 "status:"

# Verify traffic split
kubectl describe rollout myapp
# Output: Shows stable and canary pod counts
```

## Validation

- [ ] Argo Rollouts controller running
- [ ] Rollout resource created successfully
- [ ] Service created for traffic routing
- [ ] Canary pods deployed (10% of traffic)
- [ ] Gradual weight increase over time
- [ ] No traffic loss during transition
- [ ] All pods eventually running v2
- [ ] Service consistently routing traffic

## Cleanup

```bash
kubectl delete rollout myapp
kubectl delete service myapp
kubectl delete namespace argo-rollouts
```

## Common Mistakes

1. **Missing service** - Service required for traffic routing
2. **Wrong image format** - Use correct image registry and tag format
3. **Metrics not configured** - Analysis requires metrics endpoint
4. **Pause duration too short** - Not enough time to validate health
5. **No rollback trigger** - Should auto-rollback on error
6. **Metrics selector incorrect** - Must match actual metric labels
7. **Missing traffic routing** - Service mesh needed for traffic split

## Troubleshooting

**Rollout stuck:** Check metrics endpoint availability. Verify pause duration. Check error logs: `kubectl logs -n argo-rollouts deployment/argo-rollouts`.

**Canary pods not created:** Verify image exists and is pullable. Check resource quotas. Review pod events: `kubectl describe pod <pod-name>`.

**Traffic not splitting:** Verify service mesh installed (if required). Check network policies. Review iptables rules.

**Metrics query failing:** Verify Prometheus endpoint accessible. Check metric names and labels. Review AnalysisRun logs.

**Automatic rollback not triggering:** Check AnalysisTemplate configuration. Verify metrics threshold values. Review analysis status.

## Next Steps

- Implement metric-based analysis templates
- Set up traffic splitting with Istio/Flagger
- Configure automated rollback policies
- Integrate with monitoring dashboards
- Implement multi-step promotion workflows
- Set up notification webhooks for rollout events
