# 13-troubleshooting-and-best-practices: Cheatsheet

## Common ArgoCD Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Application OutOfSync | Manifest drift | `argocd app sync <name>` |
| Sync Failed | Manifest error | Check logs: `argocd app logs <name>` |
| Unable to connect repo | Auth error | Add credentials: `argocd repo add <url>` |
| Slow sync | High load | Enable cache, reduce app count |
| High CPU usage | Too many apps | Split across instances |

## Diagnostic Commands

```bash
# Get application status
argocd app get <name>

# View detailed info
argocd app info <name>

# Check sync status
argocd app get <name> | grep Sync

# View logs
argocd app logs <name>

# Force refresh from git
argocd app get <name> --hard-refresh

# Check all applications
argocd app list

# Get application YAML
kubectl get application <name> -o yaml

# View sync history
kubectl describe application <name> | grep "Operation State"
```

## Helm Troubleshooting

| Issue | Command | Notes |
|-------|---------|-------|
| Dry-run template | `helm template .` | Test manifest generation |
| Lint chart | `helm lint .` | Check chart structure |
| Debug values | `helm template . --debug` | See computed values |
| Check deps | `helm dependency list` | View chart dependencies |
| Update deps | `helm dependency update` | Fetch dependency charts |

## Log Aggregation Setup

```bash
# Forward ArgoCD logs to syslog
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
data:
  fluent-bit.conf: |
    [OUTPUT]
        Name syslog
        Match argocd.*
EOF

# View ArgoCD logs
kubectl logs -n argocd deployment/argocd-server -f
kubectl logs -n argocd deployment/argocd-repo-server -f
kubectl logs -n argocd statefulset/argocd-application-controller -f

# Search logs
kubectl logs -n argocd --all-containers=true -f | grep "ERROR\|WARN"
```

## Prometheus Queries

| Query | Purpose |
|-------|---------|
| `argocd_app_info` | Application count and status |
| `rate(argocd_app_sync_total[5m])` | Sync frequency |
| `histogram_quantile(0.95, argocd_app_sync_total)` | 95th percentile sync time |
| `argocd_app_reconcile_bucket` | Sync duration histogram |
| `increase(argocd_app_sync_total{phase="Failed"}[5m])` | Failed syncs |

## Monitoring Setup Checklist

- [ ] Prometheus scraping ArgoCD metrics
- [ ] ServiceMonitor created
- [ ] Grafana dashboards configured
- [ ] Alerts defined for failures
- [ ] Alert notifications configured
- [ ] Metrics retention policy set
- [ ] Recording rules created
- [ ] Dashboards linked to docs

## Security Hardening Checklist

- [ ] RBAC policies configured
- [ ] Network policies enforced
- [ ] Pod security policies applied
- [ ] TLS enabled for all connections
- [ ] Secrets encrypted at rest
- [ ] Audit logging enabled
- [ ] Container images scanned
- [ ] Resources limits set

## High Availability Checklist

- [ ] Multiple ArgoCD server replicas (≥2)
- [ ] Persistent storage configured
- [ ] Database replicated
- [ ] Load balancer in front
- [ ] Pod anti-affinity rules
- [ ] Health checks configured
- [ ] Resource requests/limits set
- [ ] Backup procedure documented

## Disaster Recovery Checklist

- [ ] Backup strategy documented
- [ ] RTO/RPO defined
- [ ] Backup testing scheduled
- [ ] Recovery runbook created
- [ ] Failover procedure tested
- [ ] Data restoration tested
- [ ] Communication plan prepared
- [ ] Insurance/SLA coverage reviewed

## Git Repository Issues

```bash
# Test git connectivity
kubectl exec -it deploy/argocd-repo-server -- \
  git clone <repo-url> /tmp/test

# Check git config
kubectl get secret argocd-secret -o yaml

# List configured repos
argocd repo list

# Add new repo with credentials
argocd repo add <url> \
  --username <user> \
  --password <token>

# Remove repo
argocd repo rm <url>
```

## Network Troubleshooting

```bash
# Test DNS resolution
kubectl exec -it deploy/argocd-server -- \
  nslookup github.com

# Test network connectivity
kubectl exec -it deploy/argocd-repo-server -- \
  nc -zv github.com 443

# Check firewall rules
kubectl exec -it deploy/argocd-server -- \
  iptables -L -n

# View network policies
kubectl get networkpolicy -A

# Test service connectivity
kubectl exec -it deploy/argocd-server -- \
  curl http://argocd-repo-server:8081/
```

## Performance Optimization

| Setting | Action | Impact |
|---------|--------|--------|
| Reduce sync frequency | Increase `syncOptions.syncInterval` | Lower controller load |
| Enable caching | Set cache TTL in controller | Faster comparisons |
| Server-side diff | Enable `ServerSideDiff` | Reduce bandwidth |
| Large app count | Split apps across instances | Linear scaling |
| Metrics query | Optimize Prometheus queries | Faster analysis |

## RBAC Configuration

```yaml
# Read-only viewer
g, developers, role:viewer
p, role:viewer, applications, get, */*, allow

# Admin
g, admins, role:admin
p, role:admin, *, *, */*, allow

# Application-specific
p, role:prod-deployer, applications, sync, prod/*, allow
```

## Debugging Workflow

1. Check application status: `argocd app get <name>`
2. View detailed error: `argocd app info <name>`
3. Check logs: `kubectl logs -n argocd deployment/argocd-server`
4. Verify configuration: `kubectl get application <name> -o yaml`
5. Test connectivity: `kubectl exec -it pod -- nc -zv host port`
6. Review events: `kubectl describe application <name>`
7. Check metrics: `curl http://argocd-metrics:8082/metrics`
8. Implement fix and verify: `argocd app sync <name>`

## Best Practices

- Use GitOps for all configuration (version controlled)
- Implement code review for deployments
- Separate dev/staging/prod applications
- Enable automatic sync with caution
- Use app-of-apps for hierarchical management
- Implement RBAC for multi-tenancy
- Monitor all metrics and set alerts
- Test disaster recovery regularly
- Document runbooks for common issues
- Keep ArgoCD and components updated

## Common Helm Mistakes

| Mistake | Solution |
|---------|----------|
| Image tag not updating | Use `imagePullPolicy: Always` or specific version |
| Values not overriding | Check indentation and type matching |
| Dependency not found | Run `helm dependency update` |
| Template syntax error | Use `helm template --debug` |
| Missing required value | Provide via `-f values.yaml` or `--set` |
