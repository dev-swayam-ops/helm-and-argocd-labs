# 06-argocd-basics: Exercises

## Exercise 1: Install ArgoCD in Kubernetes
Set up ArgoCD in your cluster and access the UI.

**Task:**
- Create argocd namespace
- Install ArgoCD using kubectl apply
- Verify all ArgoCD pods are running
- Get the initial admin password
- Access ArgoCD UI at localhost:8080
- Change admin password

## Exercise 2: Install and Configure ArgoCD CLI
Get hands-on with ArgoCD command-line tool.

**Task:**
- Download and install argocd CLI
- Verify installation with version command
- Login to ArgoCD server
- List accounts and users
- Test CLI connectivity to server

## Exercise 3: Connect Git Repository
Link a Git repository to ArgoCD for GitOps deployments.

**Task:**
- Create or identify Git repository (GitHub, GitLab, local Gitea)
- Add repository to ArgoCD with credentials
- Generate and use Personal Access Token
- List connected repositories
- Test repository connection
- Verify branch and path accessibility

## Exercise 4: Create First ArgoCD Application
Deploy a simple application using ArgoCD.

**Task:**
- Create application resource from Helm chart repository
- Specify source repository and chart
- Set destination cluster and namespace
- Apply application and verify creation
- Check application status
- Monitor application health

## Exercise 5: Understand ArgoCD Architecture
Learn core components and their interactions.

**Task:**
- Identify ArgoCD server component
- Identify repo-server component
- Identify application-controller component
- Document data flow between components
- Check component logs and status
- Understand role of each component

## Exercise 6: Explore ArgoCD UI Features
Navigate and use ArgoCD web interface effectively.

**Task:**
- Access applications view
- View application details
- Check resource topology
- Monitor real-time sync status
- Review application timeline
- Use search and filtering features

## Exercise 7: Configure ArgoCD RBAC
Set up basic role-based access control.

**Task:**
- Create additional user account
- Define role with specific permissions
- Assign role to user
- Test user login and permissions
- Create policy binding
- Verify permission restrictions

## Exercise 8: Set Up Notification Webhooks
Configure notifications for application events.

**Task:**
- Create notification service account
- Configure webhook endpoint
- Set up trigger conditions
- Test notification delivery
- Configure Slack/email notifications
- Verify notification triggers

## Exercise 9: Monitor Application Status
Track application health and sync state continuously.

**Task:**
- Check application health status
- Monitor sync status changes
- View resource status details
- Check operation history
- Review event logs
- Identify health degradation causes

## Exercise 10: Troubleshoot ArgoCD Issues
Diagnose and resolve common problems.

**Task:**
- Check logs for repo-server errors
- Debug application-controller issues
- Resolve repository connection failures
- Fix permission and RBAC problems
- Document and resolve sync issues
- Use CLI for advanced troubleshooting
