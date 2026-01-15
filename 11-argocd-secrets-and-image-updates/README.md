# 11-argocd-secrets-and-image-updates

## What You'll Learn
- Managing secrets in ArgoCD securely
- Sealed Secrets for encryption at rest
- External Secrets Operator integration
- Automated image update strategies
- Image scanning and validation
- Policy enforcement for image updates
- GitOps secrets best practices

## Prerequisites
- Completed modules 00-10
- Kubernetes secrets fundamentals
- ArgoCD applications knowledge
- Container registry understanding
- Git repository management
- Helm charts knowledge
- Basic encryption concepts

## Key Concepts

**Sealed Secrets:** Encrypts secrets in Git. Only decryptable in target cluster. Public key in Git, private key in cluster. Encrypt before committing.

**External Secrets Operator:** Fetches secrets from external vaults. AWS Secrets Manager, HashiCorp Vault, Azure Key Vault. Syncs to K8s secrets.

**Image Updates:** Automated tag updates in manifests. ImageUpdater scans registries for new tags. Updates Git repo with new image versions.

**Image Scanning:** Vulnerability scanning before deployment. Trivy, Snyk integration. Prevents deployment of vulnerable images.

**Policy Enforcement:** AppProject policies enforce image update rules. Restrict image sources. Require image scanning.

**GitOps Secrets:** Secrets stored encrypted in Git. Decryption at deployment time. Audit trail of secret changes.

## Hands-on Lab: Sealed Secrets with Image Updates

**Objective:** Set up Sealed Secrets for secure configuration and enable automated image updates

**Steps:**

1. **Install Sealed Secrets controller:**
```bash
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Wait for controller ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=sealed-secrets \
  -n kube-system --timeout=300s

# Get sealing key
kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/status=active \
  -o jsonpath='{.items[0].data.tls\.crt}' | base64 -d > mycert.pem
```

2. **Create and seal a secret:**
```bash
# Create plain secret
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123 \
  -n myapp --dry-run=client -o yaml > secret.yaml

# Seal the secret
kubeseal -f secret.yaml -w sealed-secret.yaml -n myapp

# Apply sealed secret
kubectl apply -f sealed-secret.yaml
```

3. **Create deployment using sealed secret:**
```bash
kubectl apply -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 1
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
        image: myapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
EOF
```

4. **Enable Image Updater:**
```bash
# Install ArgoCD Image Updater
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml

# Check logs
kubectl logs -n argocd deployment/argocd-image-updater -f
```

5. **Verify configuration:**
```bash
# Check sealed secrets controller
kubectl get pods -n kube-system -l app.kubernetes.io/name=sealed-secrets

# Check image updater
kubectl get pods -n argocd -l app=argocd-image-updater

# Verify secret
kubectl get secret my-secret -n myapp -o yaml
```

## Validation

- [ ] Sealed Secrets controller running
- [ ] Secrets encrypted in Git
- [ ] Deployment using SealedSecret
- [ ] Image Updater installed
- [ ] Automatic image tag updates working
- [ ] No plain secrets in repository
- [ ] Audit trail in Git history

## Cleanup

```bash
kubectl delete -n argocd deployment argocd-image-updater
kubectl delete sealedsecret my-secret -n myapp
kubectl delete deployment myapp -n myapp
rm mycert.pem sealed-secret.yaml secret.yaml
```

## Common Mistakes

1. **Committing plain secrets** - Always seal before Git commit
2. **Sharing sealing keys** - Keep private key secure in cluster
3. **Wrong namespace for seal** - Seal secrets with correct namespace
4. **No image policy** - Define clear image update rules
5. **Scanning disabled** - Enable image scanning in policy
6. **No audit trail** - Use Git commit history for tracking
7. **Unsigned images allowed** - Require image signature verification

## Troubleshooting

**SealedSecret not decrypting:** Check controller logs. Verify secret namespace matches. Ensure private key present.

**Image Updater not working:** Check registry credentials. Verify image tag pattern. Review updater logs.

**Permissions denied:** Verify service account can access secrets. Check AppProject resource whitelist.

**Images not updating:** Check registry connectivity. Verify image tag matches policy. Check Git credentials.

**Secret not available to pods:** Verify SealedSecret created in correct namespace. Check secret key references.

## Next Steps

- Integrate with external secret managers (Vault, AWS Secrets Manager)
- Implement image scanning with Trivy or Snyk
- Set up signature verification for images
- Create automated secret rotation
- Implement policy enforcement for image updates
