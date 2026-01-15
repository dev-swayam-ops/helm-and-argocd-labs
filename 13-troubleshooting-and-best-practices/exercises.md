# 13-troubleshooting-and-best-practices: Exercises

## Exercise 1: Diagnose Application Sync Failure
Identify and resolve ArgoCD application sync errors.

**Task:**
- Create application with invalid manifests
- Check sync status and error messages
- Review detailed error logs
- Identify root cause
- Fix manifests
- Verify sync succeeds

## Exercise 2: Debug Git Repository Issues
Troubleshoot Git connectivity and authentication.

**Task:**
- Create application with unreachable repo
- Check repo server logs
- Verify credentials stored correctly
- Test git connectivity
- Update credentials
- Confirm repo access restored

## Exercise 3: Analyze Performance Bottlenecks
Identify and resolve slow ArgoCD operations.

**Task:**
- Monitor ArgoCD metrics
- Check application count impact
- Analyze sync frequency effects
- Review resource usage
- Identify bottlenecks
- Implement optimization (caching, limits)

## Exercise 4: Implement Centralized Logging
Set up log aggregation for ArgoCD.

**Task:**
- Configure log forwarding
- Set up ELK or similar stack
- Route ArgoCD logs to central system
- Create queries for common issues
- Set up log retention policies
- Test log searchability

## Exercise 5: Configure Prometheus Monitoring
Set up metrics collection from ArgoCD.

**Task:**
- Create ServiceMonitor for ArgoCD metrics
- Configure Prometheus scrape targets
- Create recording rules
- Set up Grafana dashboard
- Query metrics using PromQL
- Verify metric collection

## Exercise 6: Implement Security Hardening
Apply production security best practices.

**Task:**
- Enable RBAC policies
- Set up network policies
- Configure pod security policies
- Implement TLS for communications
- Encrypt secrets in transit
- Audit security configuration

## Exercise 7: Set Up High Availability
Configure redundant ArgoCD components.

**Task:**
- Deploy multiple ArgoCD server replicas
- Configure persistent storage for repo-server
- Set up database replication
- Implement load balancing
- Configure anti-affinity rules
- Test failover behavior

## Exercise 8: Create Disaster Recovery Plan
Implement backup and recovery procedures.

**Task:**
- Backup ArgoCD configuration
- Document recovery procedures
- Test backup restoration
- Create runbooks
- Test failover scenarios
- Document RTO/RPO

## Exercise 9: Troubleshoot Network Issues
Diagnose cluster and external connectivity problems.

**Task:**
- Test cluster-to-repo connectivity
- Verify DNS resolution
- Check firewall rules
- Test service mesh integration
- Debug iptables rules
- Resolve connectivity issues

## Exercise 10: Production Readiness Checklist
Validate system is ready for production.

**Task:**
- Complete security checklist
- Verify monitoring setup
- Test disaster recovery
- Document architecture
- Review scaling limits
- Sign off on production readiness
