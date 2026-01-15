# 10-argocd-projects-rbac-and-multi-tenancy

## What You'll Learn
- ArgoCD Projects for application isolation
- Role-Based Access Control (RBAC) configuration
- Multi-tenancy best practices
- Team and namespace isolation
- Permission scoping and enforcement
- Policy enforcement with AppProjects
- Secure cluster and source management

## Prerequisites
- Completed modules 00-09
- ArgoCD installation and basics
- Kubernetes RBAC fundamentals
- Service accounts and roles knowledge
- Understanding of ArgoCD applications
- Git and repository management

## Key Concepts

**ArgoCD Project:** Isolates applications and resources. Controls allowed Git sources, Kubernetes destinations, and resources. Defines access boundaries between teams.

**RBAC in ArgoCD:** Role-based access to projects and applications. Service accounts for automation. Different permissions per role. Integrates with Kubernetes RBAC.

**Multi-tenancy:** Multiple teams sharing single ArgoCD instance. Namespace isolation. Source repository restrictions. Destination cluster limitations. Resource quotas.

**AppProject Resource:** Kubernetes CRD defining project boundaries. Scopes Git sources, K8s destinations, permissions. Enforces least-privilege access.

**Team Isolation:** Separate service accounts per team. Project-level access control. Namespace segregation. Repository restrictions.

**Cluster Scoping:** Limit teams to specific clusters. Namespace restrictions per cluster. Resource quotas. Prevent cross-cluster deployments.

## Hands-on Lab: Multi-Team ArgoCD Setup

**Objective:** Set up ArgoCD with two teams (team-a, team-b) with isolated projects, RBAC, and namespace restrictions

**Steps:**

1. **Create team namespaces:**
```bash
kubectl create namespace team-a
kubectl create namespace team-b
kubectl create namespace team-a-apps
kubectl create namespace team-b-apps
```

2. **Create AppProject for team-a:**
```bash
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a
  namespace: argocd
spec:
  sourceRepos:
    - "https://github.com/example/team-a-repo"
  destinations:
    - namespace: "team-a-*"
      server: "https://kubernetes.default.svc"
  namespaceResourceBlacklist:
    - group: ""
      kind: ResourceQuota
EOF
```

3. **Create AppProject for team-b:**
```bash
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-b
  namespace: argocd
spec:
  sourceRepos:
    - "https://github.com/example/team-b-repo"
  destinations:
    - namespace: "team-b-*"
      server: "https://kubernetes.default.svc"
EOF
```

4. **Create service accounts for each team:**
```bash
kubectl create serviceaccount team-a-user -n argocd
kubectl create serviceaccount team-b-user -n argocd
kubectl create rolebinding team-a-user-admin \
  --clusterrole=admin \
  --serviceaccount=argocd:team-a-user
kubectl create rolebinding team-b-user-admin \
  --clusterrole=admin \
  --serviceaccount=argocd:team-b-user
```

5. **Create applications for each team:**
```bash
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: team-a-app
  namespace: argocd
spec:
  project: team-a
  source:
    repoURL: https://github.com/example/team-a-repo
    targetRevision: main
    path: apps/
  destination:
    server: https://kubernetes.default.svc
    namespace: team-a-apps
  syncPolicy:
    automated:
      prune: true
EOF
```

6. **Verify isolation:**
```bash
kubectl get appproject -n argocd
# Output: team-a, team-b projects
argocd app list
# Output: Shows applications assigned to respective projects
```

## Validation

- [ ] AppProjects created for both teams
- [ ] Service accounts configured
- [ ] RBAC roles applied
- [ ] Applications deployed to correct project
- [ ] Namespace restrictions enforced
- [ ] Source repos restricted per project
- [ ] Cross-team deployments prevented

## Cleanup

```bash
argocd app delete team-a-app team-b-app
kubectl delete appproject team-a team-b -n argocd
kubectl delete serviceaccount team-a-user team-b-user -n argocd
kubectl delete rolebinding team-a-user-admin team-b-user-admin
kubectl delete ns team-a team-b team-a-apps team-b-apps
```

## Common Mistakes

1. **Too permissive projects** - Restrict sources and destinations tightly
2. **Missing namespace wildcards** - Use patterns like "team-*"
3. **Shared service accounts** - Create separate accounts per team
4. **Overprivileged roles** - Use least privilege principle
5. **Forgetting RBAC on clusters** - Set destination restrictions
6. **No resource quotas** - Add namespaceResourceBlacklist
7. **Missing namespace isolation** - Enforce at project level

## Troubleshooting

**Applications fail to deploy:** Check AppProject namespace restrictions. Verify destination server. Review source repo access.

**RBAC permission denied:** Check service account bindings. Verify role permissions. Review AppProject resource whitelist.

**Team cross-access:** Confirm source repos restricted. Check namespace patterns. Verify no wildcards too broad.

**Cluster not accessible:** Verify cluster registered. Check network connectivity. Review destination server in AppProject.

## Next Steps

- Implement external OIDC authentication
- Set up SSO integration with GitHub/GitLab
- Configure per-team RBAC policies
- Implement resource quota limits
- Set up audit logging for multi-tenant access
