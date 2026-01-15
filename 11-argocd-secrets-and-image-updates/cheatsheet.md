# 11-argocd-secrets-and-image-updates: Cheatsheet

## Sealed Secrets Installation

```bash
# Install controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Export public key
kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/status=active \
  -o jsonpath='{.items[0].data.tls\.crt}' | base64 -d > mycert.pem

# Install kubeseal CLI
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/kubeseal-linux-amd64
chmod +x kubeseal-linux-amd64 && mv kubeseal-linux-amd64 /usr/local/bin/kubeseal
```

## Sealed Secrets Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubeseal -f secret.yaml` | Seal secret | Creates SealedSecret from Secret |
| `kubeseal --recovery-unseal` | Debug unseal | Manual decryption (dev only) |
| `kubectl apply -f sealed-secret.yaml` | Deploy sealed secret | Controller auto-unseals |
| `kubectl get sealedsecret` | List sealed secrets | Shows SealedSecret resources |

## SealedSecret Template

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: my-secret
  namespace: myapp
spec:
  encryptedData:
    password: AgBvX3Z... # Encrypted value
    username: AgBkZ2F... # Encrypted value
  template:
    metadata:
      name: my-secret
      namespace: myapp
    type: Opaque
```

## Image Updater Installation

```bash
# Install image-updater
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml

# Check status
kubectl get deployment -n argocd argocd-image-updater
kubectl logs -n argocd -l app=argocd-image-updater
```

## Image Update Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `argocd-image-updater.argoproj.io/image-list` | Images to scan | `myapp=registry.io/myapp` |
| `argocd-image-updater.argoproj.io/myapp.update-strategy` | Update strategy | `latest`, `semver`, `digest` |
| `argocd-image-updater.argoproj.io/myapp.allow-tags` | Tag regex | `.*`, `v\d+\.\d+` |
| `argocd-image-updater.argoproj.io/git-branch` | Git branch for commits | `image-updates` |
| `argocd-image-updater.argoproj.io/write-back-method` | How to update | `git` or `argocd` |

## Application with Image Updates

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: myapp=myregistry.azurecr.io/myapp
    argocd-image-updater.argoproj.io/myapp.update-strategy: latest
    argocd-image-updater.argoproj.io/git-branch: image-updates
    argocd-image-updater.argoproj.io/write-back-method: git
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

## Secret in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        volumeMounts:
        - name: secret-vol
          mountPath: /etc/secrets
      volumes:
      - name: secret-vol
        secret:
          secretName: db-secret
```

## Image Registry Secret

```bash
# Create docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.azurecr.io \
  --docker-username=username \
  --docker-password=password \
  --docker-email=email@example.com

# Seal registry secret
kubeseal -f regcred.yaml -w sealed-regcred.yaml --cert mycert.pem

# Use in pod
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: myregistry.azurecr.io/myapp:1.0
```

## Troubleshooting Commands

| Issue | Command |
|-------|---------|
| SealedSecret not unsealing | `kubectl logs -n kube-system -l app.kubernetes.io/name=sealed-secrets` |
| Image Updater not scanning | `kubectl logs -n argocd -l app=argocd-image-updater` |
| Registry auth failed | `kubectl describe pod <pod-name>` - check ImagePullBackOff |
| Secret not in pod | `kubectl exec <pod> -- env \| grep SECRET` |
| Check sealed secret | `kubectl get sealedsecret <name> -o yaml` |

## Image Scanning

```bash
# Install Trivy
wget https://github.com/aquasecurity/trivy/releases/download/v0.40.0/trivy_linux_amd64.tar.gz
tar xfz trivy_linux_amd64.tar.gz && mv trivy /usr/local/bin/

# Scan image
trivy image myregistry.azurecr.io/myapp:1.0
trivy image --severity HIGH,CRITICAL myregistry.azurecr.io/myapp:1.0

# Generate report
trivy image --format json myregistry.azurecr.io/myapp:1.0 > scan-report.json
```

## Secret Best Practices

- Never commit plain secrets to Git
- Always seal before committing
- Rotate secrets regularly
- Use separate secrets per environment
- Implement least-privilege access
- Enable audit logging
- Use external secret managers for production
- Keep sealing keys secure in cluster

## Image Update Strategies

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| `latest` | Always latest tag | Development |
| `semver` | Version range | Controlled updates |
| `digest` | Specific image hash | Immutable deployments |

## Multi-Environment Secret Setup

```
dev/
├── secrets/
│   └── sealed-secrets.yaml
staging/
├── secrets/
│   └── sealed-secrets.yaml
prod/
├── secrets/
│   └── sealed-secrets.yaml
```

Each sealed with cluster-specific public key.

## Registry Credential Types

| Type | Authentication | Use |
|------|---|---|
| `docker-registry` | Username/Password | Docker Hub, ECR, ACR |
| `basic-auth` | Username/Password | Private registries |
| `bearer-token` | Token | Cloud registries |

## Verification Checklist

- [ ] Sealed Secrets controller running
- [ ] Sealing key backed up securely
- [ ] kubeseal installed and working
- [ ] Secrets encrypted before Git commit
- [ ] Image Updater deployment running
- [ ] Image update annotations configured
- [ ] Registry credentials secure
- [ ] Image scanning enabled
- [ ] Policy enforcement active
- [ ] Audit trail maintained
