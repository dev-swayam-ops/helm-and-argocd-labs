# 07-argocd-applications-and-sync: Exercises

## Exercise 1: Create Application from Git Repository
Set up an application deploying manifests from Git.

**Task:**
- Create Git repository with Kubernetes manifests
- Configure ArgoCD application pointing to repo
- Set targetRevision to specific branch
- Configure namespace creation
- Deploy and verify resources
- Check sync status

## Exercise 2: Create Application from Helm Chart
Deploy using Helm charts with ArgoCD.

**Task:**
- Reference external Helm repository
- Set chart name and version
- Override Helm values in application
- Configure value files from Git
- Deploy application
- Verify Helm-based deployment

## Exercise 3: Implement Automated Sync Policy
Enable automatic synchronization from Git.

**Task:**
- Create application with automated sync enabled
- Set prune policy for cleanup
- Enable self-healing
- Modify Git repository and observe auto-sync
- Verify resources sync automatically
- Document behavior

## Exercise 4: Configure Manual Sync Policy
Set up manual approval workflow.

**Task:**
- Create application with manual sync disabled
- Check OutOfSync status
- Manually trigger sync
- Wait for sync completion
- Review operation history
- Compare with automated sync

## Exercise 5: Manage Multiple Sync Policies
Use different sync strategies for different resources.

**Task:**
- Create application with mixed sync policies
- Use syncOptions for granular control
- Set CreateNamespace option
- Configure PruneLast strategy
- Handle CRD/RBAC ordering
- Document policy combinations

## Exercise 6: Implement Destination Cluster Configuration
Configure multi-cluster deployments.

**Task:**
- Register multiple cluster contexts
- Create applications for different clusters
- Configure destination server settings
- Use wildcards for dynamic destinations
- Verify correct cluster deployments
- Monitor cross-cluster health

## Exercise 7: Monitor and Analyze Sync Status
Track application synchronization details.

**Task:**
- Check operation progress during sync
- View resource-level sync status
- Identify drift between Git and cluster
- Analyze sync failure causes
- Review sync history and timeline
- Use logs for debugging

## Exercise 8: Configure Notification and Webhook Triggers
Set up event-driven notifications.

**Task:**
- Create notification service account
- Configure trigger conditions
- Set notification destinations
- Test notification delivery
- Create custom triggers
- Verify webhook integration

## Exercise 9: Manage Application Lifecycle Operations
Handle create, update, and delete operations.

**Task:**
- Create application resource
- Update application configuration
- Perform upgrades with sync
- Manage resource deletion
- Use cascading delete policies
- Document lifecycle events

## Exercise 10: Troubleshoot Application Sync Issues
Diagnose and resolve synchronization problems.

**Task:**
- Identify common sync failures
- Debug source validation errors
- Resolve destination connection issues
- Fix resource conflicts
- Handle permission problems
- Document troubleshooting procedures
