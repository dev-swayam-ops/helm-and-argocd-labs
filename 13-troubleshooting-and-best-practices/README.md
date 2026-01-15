# 13-troubleshooting-and-best-practices

## What You'll Learn
- Debugging ArgoCD and Helm deployments
- Common error diagnosis and resolution
- Logging and monitoring strategies
- Performance optimization techniques
- Security best practices
- High availability configuration
- Production readiness checklist

## Prerequisites
- Completed modules 00-12
- Kubernetes troubleshooting basics
- Log analysis skills
- Monitoring fundamentals
- Network troubleshooting knowledge
- Container debugging experience

## Key Concepts

**Debugging Workflow:** Identify symptoms. Check logs and events. Review configuration. Test connectivity. Validate permissions. Verify resource states.

**Log Aggregation:** Centralize logs from ArgoCD, applications, controllers. Use ELK, Splunk, CloudWatch. Query logs across components.

**Monitoring & Alerting:** Prometheus scrapes metrics. Grafana visualizes. AlertManager triggers notifications. Watch health and performance.

**Performance Tuning:** ArgoCD resource limits. Sync frequency optimization. Cache configuration. Garbage collection settings.

**Security Hardening:** RBAC enforcement. Secret encryption. Network policies. Pod security policies. Admission webhooks.

**HA Configuration:** Multiple replicas. Persistent storage. Load balancing. Database replication. Disaster recovery planning.

## Hands-on Lab: Diagnose and Fix Common Issues

**Objective:** Troubleshoot ArgoCD application sync failure and implement monitoring

**Steps:**

1. **Create application with intentional issue:**
```bash
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: broken-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/nonexistent/repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: broken-app
  syncPolicy:
    automated:
      prune: true
EOF
```

2. **Check application status:**
```bash
# Get application status
argocd app get broken-app
# Output: Shows OutOfSync and error

# Get detailed information
argocd app info broken-app

# View logs
argocd app logs broken-app
```

3. **Diagnose the issue:**
```bash
# Check ArgoCD server logs
kubectl logs -n argocd deployment/argocd-server -f

# Check repo-server logs (Git connectivity)
kubectl logs -n argocd deployment/argocd-repo-server -f

# Check application controller
kubectl logs -n argocd statefulset/argocd-application-controller -f

# Get events
kubectl get events -n argocd | grep broken-app
```

4. **Fix configuration:**
```bash
# Update with correct repository
kubectl patch application broken-app -n argocd \
  -p '{"spec":{"source":{"repoURL":"https://github.com/example/repo"}}}' \
  --type merge

# Force refresh
argocd app get broken-app --hard-refresh
```

5. **Implement monitoring:**
```bash
# Check ArgoCD metrics
kubectl exec -n argocd deployment/argocd-server -- \
  curl -s localhost:8083/metrics | head -20

# Create ServiceMonitor for Prometheus
kubectl apply -f - << 'EOF'
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-metrics
  namespace: argocd
spec:
  selector:
    matchLabels:
      app: argocd-metrics
  endpoints:
  - port: metrics
    interval: 30s
EOF
```

## Validation

- [ ] Application error identified
- [ ] Root cause determined from logs
- [ ] Configuration corrected
- [ ] Application synced successfully
- [ ] Metrics endpoint accessible
- [ ] Monitoring dashboard shows data
- [ ] Alerting rules configured
- [ ] Recovery documented

## Cleanup

```bash
argocd app delete broken-app
kubectl delete application broken-app -n argocd
kubectl delete servicemonitor argocd-metrics -n argocd
```

## Common Mistakes

1. **Not checking logs first** - Logs contain detailed error messages
2. **Wrong namespace** - ArgoCD runs in argocd namespace by default
3. **Ignoring events** - Kubernetes events show underlying issues
4. **No monitoring** - Cannot troubleshoot without visibility
5. **Assuming git issue** - Could be networking, RBAC, or manifests
6. **Not testing connectivity** - Verify repo and cluster access
7. **Overcomplicating fixes** - Start simple, add complexity if needed

## Troubleshooting

**Application OutOfSync:** Check Git connectivity. Verify repo URL and credentials. Check RBAC permissions.

**Sync fails:** Review detailed errors in logs. Check manifest syntax with `kubectl apply --dry-run`. Verify resource permissions.

**Slow syncs:** Reduce sync frequency. Optimize manifest loading. Check controller resources. Monitor network.

**High memory usage:** Tune garbage collection. Reduce application count. Increase ArgoCD replicas.

**Pod not starting:** Check image pullability. Verify resource requests/limits. Review pod events and logs.

**Network issues:** Test cluster-to-repo connectivity. Check firewall rules. Verify DNS resolution.

## Next Steps

- Implement centralized logging (ELK, Splunk)
- Set up comprehensive monitoring (Prometheus, Grafana)
- Create runbooks for common issues
- Implement disaster recovery procedures
- Set up automated health checks
- Document system architecture and dependencies
