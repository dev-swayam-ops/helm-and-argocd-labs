# 08-gitops-workflows-and-promotion: Exercises

## Exercise 1: Create Multi-Environment Git Repository Structure
Set up Git repository with separate paths for dev, staging, and production.

**Task:**
- Create directory structure (dev/staging/prod)
- Create different manifests for each environment
- Initialize Git repository
- Commit all files
- Verify repository structure

## Exercise 2: Configure Dev Environment with Auto-Sync
Set up development ArgoCD application with automatic synchronization.

**Task:**
- Create dev application with automated sync enabled
- Set prune and self-heal policies to true
- Configure CreateNamespace option
- Deploy and verify syncing
- Modify Git and observe auto-sync behavior

## Exercise 3: Configure Staging with Manual Sync
Implement manual approval workflow for staging environment.

**Task:**
- Create staging application with manual sync
- Verify application shows OutOfSync initially
- Manually trigger sync after reviewing changes
- Check sync history and operation details
- Compare with dev auto-sync behavior

## Exercise 4: Configure Production with Strict Controls
Set up production with minimum required permissions.

**Task:**
- Create production application with manual sync
- Add finalizers for controlled deletion
- Set restrictive namespace policies
- Configure resource limits
- Implement change validation before sync

## Exercise 5: Implement Image Version Promotion
Automate image version updates across environments.

**Task:**
- Create deployment with pinned image version in dev
- Set different image versions for staging and prod
- Update image tags in Git repository
- Trigger ArgoCD refresh and observe propagation
- Document promotion workflow

## Exercise 6: Create GitOps Promotion Pipeline
Establish workflow for changes flowing dev → staging → prod.

**Task:**
- Create Git pull request workflow
- Set up branch protection rules
- Implement code review requirements
- Document promotion process
- Create checklist for each promotion step

## Exercise 7: Implement Rollback Strategy
Set up ability to revert failed deployments.

**Task:**
- Tag releases in Git repository
- Create rollback branch with previous version
- Trigger rollback via Git commit
- Verify resources reverted to previous state
- Document rollback procedures

## Exercise 8: Monitor Promotion Progress
Track deployments through the promotion pipeline.

**Task:**
- Check application status across all environments
- View sync history and operation details
- Monitor resource health during promotion
- Check logs for errors and warnings
- Create monitoring dashboard for environments

## Exercise 9: Implement Approval Gates
Add approval mechanism between environments.

**Task:**
- Set up GitOps approval workflow
- Require human review before staging/prod deployment
- Configure notification webhooks
- Test approval flow with blocked promotion
- Document approval requirements

## Exercise 10: Troubleshoot Promotion Failures
Diagnose and resolve issues in promotion workflow.

**Task:**
- Identify common promotion failures
- Check sync logs for error messages
- Verify Git repository connectivity
- Review ArgoCD RBAC permissions
- Fix configuration issues and retry
