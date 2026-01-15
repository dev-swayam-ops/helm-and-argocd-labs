# 10-argocd-projects-rbac-and-multi-tenancy: Cheatsheet

## AppProject Resource Template

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-project
  namespace: argocd
spec:
  description: Team project with isolation
  sourceRepos:
    - "https://github.com/example/team-repo"
    - "https://github.com/example/team-*"
  destinations:
    - namespace: "team-*"
      server: "https://kubernetes.default.svc"
    - namespace: "team-prod"
      server: "https://prod-cluster:6443"
  namespaceResourceBlacklist:
    - group: ""
      kind: ResourceQuota
    - group: "policy"
      kind: NetworkPolicy
  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"
  clusterResourceWhitelist:
    - group: "*"
      kind: "*"
  clusterResourceBlacklist:
    - group: ""
      kind: Namespace
```

## RBAC Roles

| Role | Permissions | Use Case |
|------|-----------|----------|
| read-only | get, list, watch | Developers viewing apps |
| deployer | get, list, create, update, delete | CI/CD pipelines |
| admin | All permissions | Team leads |

## Service Account Creation

```bash
# Create service account
kubectl create serviceaccount team-user -n argocd

# Generate token
kubectl create token team-user -n argocd --duration=8760h

# Create role and binding
kubectl create role team-role -n argocd --verb=get,list --resource=applications
kubectl create rolebinding team-binding --role=team-role --serviceaccount=argocd:team-user -n argocd
```

## Namespace Patterns

| Pattern | Matches | Example |
|---------|---------|---------|
| `exact-namespace` | Exact match | "team-dev" |
| `team-*` | Wildcard prefix | team-dev, team-staging |
| `*-prod` | Wildcard suffix | app-prod, api-prod |
| `team-[a-z]*` | Regex pattern | team-app, team-api |

## AppProject Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl create appproject` | Create project | With sourceRepos and destinations |
| `kubectl get appproject` | List projects | In namespace argocd |
| `kubectl describe appproject <name>` | Project details | Shows restrictions |
| `kubectl delete appproject <name>` | Delete project | Removes isolation |

## RBAC Commands

| Command | Purpose |
|---------|---------|
| `kubectl create serviceaccount <name>` | Create service account |
| `kubectl create role <name>` | Create role with rules |
| `kubectl create rolebinding <name>` | Bind role to account |
| `kubectl auth can-i <verb> <resource>` | Test permissions |
| `kubectl get roles -n argocd` | List RBAC roles |

## Common Permission Sets

### Read-Only Developer
```yaml
verbs: ["get", "list", "watch"]
resources: ["applications", "applicationsets"]
```

### CI/CD Deployer
```yaml
verbs: ["get", "list", "create", "update", "patch"]
resources: ["applications"]
```

### Project Manager
```yaml
verbs: ["get", "list", "create", "update", "delete"]
resources: ["applications", "appprojects"]
```

## Multi-Tenancy Isolation Layers

| Layer | Configuration | Example |
|-------|---------------|---------|
| Project | sourceRepos, destinations | team-a → only team-a repo |
| Namespace | destination namespace | team-a → team-a-* namespaces |
| Cluster | destination server | team-a → dev cluster only |
| RBAC | service account roles | team-a-user → read-only role |

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| Permission denied | `kubectl auth can-i <verb> <resource> --as <sa>` |
| Check project | `kubectl get appproject -n argocd` |
| Review project spec | `kubectl describe appproject <name> -n argocd` |
| Test RBAC | `kubectl auth can-i list applications --as system:serviceaccount:argocd:user` |
| Check bindings | `kubectl get rolebinding -n argocd` |
| View cluster access | `argocd cluster list` |

## Resource Quota Example

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: team-dev
spec:
  hard:
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "100"
    services: "10"
```

## Application Project Assignment

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  project: team-project  # Links to AppProject
  source:
    repoURL: https://github.com/example/team-repo
```

## Multi-Cluster Setup

```bash
# Register clusters
argocd cluster add dev-cluster
argocd cluster add prod-cluster

# List clusters
argocd cluster list

# Use in project destination
destinations:
  - server: https://dev-cluster:6443
    namespace: "team-*"
```

## Best Practices

- Separate AppProject per team
- Namespace patterns with wildcards
- Dedicated service accounts per team
- RBAC roles following least privilege
- Resource quotas per team namespace
- Cluster destinations restrict scope
- Regular access reviews
- Audit logging of changes

## Team Isolation Checklist

- [ ] AppProject created with team name
- [ ] sourceRepos restricted to team repos
- [ ] destinations limited to team namespaces
- [ ] Service account created for team
- [ ] RBAC roles assigned per role (viewer, deployer, admin)
- [ ] namespaceResourceBlacklist configured
- [ ] ResourceQuota set per namespace
- [ ] Cluster destinations restricted
- [ ] RBAC permissions tested
- [ ] Cross-team access prevented
