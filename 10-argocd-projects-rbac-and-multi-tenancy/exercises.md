# 10-argocd-projects-rbac-and-multi-tenancy: Exercises

## Exercise 1: Create AppProject for Team Isolation
Set up AppProject resource to isolate team applications.

**Task:**
- Create AppProject named "team-dev"
- Restrict sourceRepos to single Git repository
- Set destinations to team namespace
- Configure resource blacklist/whitelist
- Apply and verify project created
- List projects to confirm

## Exercise 2: Implement Namespace Isolation
Configure project to restrict deployments to specific namespaces.

**Task:**
- Create AppProject with namespace patterns (team-*)
- Set multiple destination namespaces
- Use wildcard patterns appropriately
- Create application in project
- Verify deployment to correct namespace only
- Test cross-namespace deployment rejection

## Exercise 3: Create Service Accounts per Team
Set up separate service accounts for team access.

**Task:**
- Create serviceaccount for team
- Generate and export token
- Create RBAC role for service account
- Configure rolebinding
- Grant project-specific permissions
- Test authentication with token

## Exercise 4: Configure RBAC for Multi-Tenancy
Implement role-based access control across projects.

**Task:**
- Create Role for project access
- Create RoleBinding for team
- Set up least-privilege permissions
- Create multiple roles (read-only, deployer, admin)
- Assign roles to service accounts
- Test permission enforcement

## Exercise 5: Manage Cluster Destinations
Configure which clusters each team can access.

**Task:**
- Register multiple clusters in ArgoCD
- Create AppProject with cluster destinations
- Restrict team to specific cluster
- Verify cluster routing
- Test cross-cluster prevention
- Document cluster access matrix

## Exercise 6: Enforce Source Repository Restrictions
Limit teams to specific Git repositories.

**Task:**
- Create multiple AppProjects
- Set different sourceRepos per project
- Configure repository patterns
- Test repository access enforcement
- Prevent unauthorized repo deployments
- Verify repo whitelist/blacklist

## Exercise 7: Implement Resource Quotas
Set per-namespace resource limits for teams.

**Task:**
- Create ResourceQuota in team namespace
- Configure CPU, memory, pod limits
- Apply quota to application deployments
- Test quota enforcement
- Monitor resource usage per team
- Document quota policies

## Exercise 8: Manage Application Ownership
Assign applications to specific projects and teams.

**Task:**
- Create applications in team projects
- Set project field in application spec
- Verify project assignment
- Test cross-project deployment prevention
- Update application project ownership
- Document ownership model

## Exercise 9: Configure Multi-Cluster RBAC
Set up RBAC across multiple clusters.

**Task:**
- Register multiple clusters
- Create cluster-specific RBAC roles
- Set per-cluster permissions
- Configure service accounts per cluster
- Test cluster-specific access
- Implement cross-cluster policies

## Exercise 10: Troubleshoot Multi-Tenancy Issues
Diagnose and resolve multi-tenancy problems.

**Task:**
- Identify permission denied errors
- Debug AppProject restrictions
- Fix namespace isolation issues
- Resolve service account problems
- Check RBAC bindings
- Document troubleshooting procedures
