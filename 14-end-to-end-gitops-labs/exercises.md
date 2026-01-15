# 14-end-to-end-gitops-labs: Exercises

## Exercise 1: Design Multi-Environment Architecture
Plan complete GitOps setup for dev/staging/prod environments.

**Task:**
- Define environment-specific configurations
- Design Helm chart structure
- Plan ArgoCD application layout
- Document promotion workflow
- Map repositories and branches
- Define access control per environment

## Exercise 2: Implement Multi-Tier Helm Charts
Create reusable Helm charts with environment overrides.

**Task:**
- Create base Helm chart structure
- Add environment-specific value files
- Implement dependency management
- Configure namespace isolation
- Test templating across environments
- Validate manifest generation

## Exercise 3: Set Up Git Workflow with Branches
Implement branch strategy for GitOps promotion.

**Task:**
- Create main branch for production
- Create develop branch for staging
- Set up feature branch workflow
- Configure branch protection rules
- Test merge procedures
- Validate automatic promotions

## Exercise 4: Configure ArgoCD Multi-Environment
Deploy ArgoCD applications for each environment.

**Task:**
- Create application manifests
- Configure auto-sync for dev
- Configure manual-sync for prod
- Set up approval workflows
- Implement project isolation
- Test cross-environment deployments

## Exercise 5: Integrate CI/CD with GitOps
Connect build pipeline to ArgoCD automation.

**Task:**
- Create CI pipeline (GitHub Actions/GitLab CI)
- Automate image building
- Update Git repository automatically
- Trigger ArgoCD sync
- Test end-to-end promotion
- Validate image tag updates

## Exercise 6: Implement Progressive Delivery
Deploy new versions using canary strategy.

**Task:**
- Configure Argo Rollouts for production
- Set up traffic splitting
- Define analysis templates
- Configure automated rollback
- Test failed deployment recovery
- Monitor metrics during rollout

## Exercise 7: Set Up Application Monitoring
Configure observability across deployment.

**Task:**
- Deploy Prometheus and Grafana
- Create ServiceMonitor resources
- Build deployment dashboards
- Define alerting rules
- Test alert notifications
- Validate metric collection

## Exercise 8: Implement Backup and Recovery
Create disaster recovery procedures.

**Task:**
- Backup ArgoCD configuration
- Document cluster state snapshots
- Test restoration procedures
- Create recovery runbook
- Test failover scenarios
- Validate data consistency

## Exercise 9: Troubleshoot Failed Deployment
Diagnose and resolve deployment issues.

**Task:**
- Identify sync failures
- Analyze error logs
- Fix manifest issues
- Test fixes in lower environments
- Implement in production safely
- Document resolution

## Exercise 10: Complete Production Rollout
Execute full-stack deployment across all environments.

**Task:**
- Update application version
- Deploy to dev (auto-sync)
- Validate in staging
- Promote to production
- Monitor canary deployment
- Verify complete rollout
