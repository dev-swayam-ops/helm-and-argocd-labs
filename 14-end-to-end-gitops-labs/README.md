# 14-end-to-end-gitops-labs

## What You'll Learn
- Complete GitOps workflow from commit to production
- Multi-environment promotion strategy (dev → staging → prod)
- Helm charts with ArgoCD for full stack deployment
- Automated CI/CD to GitOps integration
- Progressive delivery with Argo Rollouts
- Real-world application deployment patterns
- Monitoring and observability throughout pipeline

## Prerequisites
- Completed modules 00-13
- Kubernetes cluster (Minikube/Kind acceptable)
- Git repository access
- Docker registry account
- ArgoCD installed and configured
- Helm 3.x and kubectl configured
- Basic CI/CD pipeline knowledge

## Key Concepts

**End-to-End GitOps:** Source code commit triggers automated pipeline. Pipeline builds, tests, creates Helm charts, updates Git. ArgoCD watches Git and deploys changes automatically. No manual deployments.

**Multi-Environment Promotion:** Code flows through dev → staging → prod with different configurations. Same charts, different values per environment. Automated gates between stages with approval workflows.

**CI/CD to GitOps Bridge:** CI pipeline builds and publishes artifacts. Git repo updated with new image tags. ArgoCD detects Git changes and synchronizes clusters automatically. Single source of truth.

**AppOfApps Pattern:** Root application references multiple applications. Enables hierarchical structure. One sync command deploys entire stack. Rollback reverts all components together.

**Progressive Delivery Integration:** Canary deployments validate new versions before full rollout. Metrics analysis prevents bad deployments. Automatic rollback recovers from failures instantly.

## Hands-on Lab: Deploy Complete Application Stack Across Environments

**Objective:** Build and deploy application through dev → staging → prod using Helm + ArgoCD + Argo Rollouts with full monitoring

**Steps:**

1. **Create Git repository structure:**
```bash
# Create directory structure
mkdir -p gitops-repo/{dev,staging,prod,helm/myapp,ci}

# Create Helm chart
helm create gitops-repo/helm/myapp

# Initialize git repo
cd gitops-repo && git init && git config user.email "devops@lab.local"
git config user.name "DevOps Lab" && git add . && git commit -m "Initial commit"
```

2. **Set up multi-environment values:**
```bash
# Create environment-specific values
cat > gitops-repo/helm/myapp/values-dev.yaml << 'EOF'
replicas: 1
image:
  tag: latest
  pullPolicy: Always
ingress:
  host: myapp-dev.local
EOF

cat > gitops-repo/helm/myapp/values-prod.yaml << 'EOF'
replicas: 3
image:
  tag: v1.0.0
  pullPolicy: IfNotPresent
ingress:
  host: myapp.example.com
EOF
```

3. **Create ArgoCD Applications for each environment:**
```bash
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/gitops-repo
    targetRevision: main
    path: helm/myapp
    helm:
      releaseName: myapp
      values: |
        replicas: 1
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF

kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/user/gitops-repo
    targetRevision: main
    path: helm/myapp
    helm:
      releaseName: myapp
      values: |
        replicas: 3
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated: false
    syncOptions:
    - CreateNamespace=true
EOF
```

4. **Set up deployment with progressive delivery:**
```bash
# Create Rollout for canary deployment in prod
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
  namespace: prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 5m}
      - setWeight: 50
      - pause: {duration: 5m}
EOF
```

5. **Create monitoring and validation:**
```bash
# Create ServiceMonitor for Prometheus
kubectl apply -f - << 'EOF'
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
  namespace: prod
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
EOF

# Verify deployment
kubectl get applications -n argocd
kubectl get rollout -n prod
kubectl get pods -n prod -l app=myapp
```

6. **Test promotion workflow:**
```bash
# Push code change to git
cd gitops-repo && echo "v2" > VERSION && git add . && git commit -m "Bump version"
git push origin main

# ArgoCD automatically detects and syncs dev
argocd app get myapp-dev
# Status changes to Synced

# Manually trigger prod after approval
argocd app sync myapp-prod --prune

# Watch progressive delivery
kubectl rollout status rollout/myapp -n prod --timeout=10m
```

## Validation

- [ ] Git repository structure created
- [ ] Helm charts deployed to all environments
- [ ] ArgoCD applications synced successfully
- [ ] Dev auto-syncs on Git changes
- [ ] Prod manual-sync approval workflow
- [ ] Progressive delivery canary started
- [ ] Metrics collection working
- [ ] Multi-environment deployment verified

## Cleanup

```bash
argocd app delete myapp-dev myapp-prod
kubectl delete rollout myapp -n prod
kubectl delete application -n argocd -l app=myapp
kubectl delete namespace dev staging prod
```

## Common Mistakes

1. **No approval gates** - Prod should require manual approval
2. **Same values everywhere** - Use environment-specific value files
3. **Automatic sync on production** - Enable only for safe environments
4. **No rollback plan** - Test disaster recovery procedures
5. **Metrics not configured** - Cannot validate deployments without data
6. **Mixed manual/GitOps** - Keep Git as single source of truth
7. **No monitoring** - Cannot detect issues without observability

## Troubleshooting

**ArgoCD not syncing:** Check Git repository credentials and connectivity. Verify branch name matches target revision. Check RBAC permissions.

**Application failing in prod:** Review logs from rollout. Check metrics for errors. Verify all dependencies deployed. Review recent Git changes.

**Slow deployment:** Monitor ArgoCD sync time. Check image pull duration. Review manifest size and complexity. Analyze network latency to cluster.

**Rollout not progressing:** Check analysis metrics if configured. Verify pause duration. Review error logs. Check resource availability.

## Next Steps

- Implement automated rollback on metrics failure
- Add security scanning to CI/CD pipeline
- Set up multi-cluster GitOps across regions
- Implement backup and disaster recovery
- Create custom dashboards for visibility
- Document runbooks for production operations
