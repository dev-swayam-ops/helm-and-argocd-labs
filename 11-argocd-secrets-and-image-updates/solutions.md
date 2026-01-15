# 11-argocd-secrets-and-image-updates: Solutions

## Exercise 1: Install and Verify Sealed Secrets

**Solution:**
```bash
# Install sealed-secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Wait for controller to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=sealed-secrets \
  -n kube-system --timeout=300s

# Verify pod running
kubectl get pods -n kube-system -l app.kubernetes.io/name=sealed-secrets
# Output: sealed-secrets-xxxxx pod in Running state

# Check controller logs
kubectl logs -n kube-system -l app.kubernetes.io/name=sealed-secrets
# Output: Shows controller initialized and ready

# Export sealing key (public key)
kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/status=active \
  -o jsonpath='{.items[0].data.tls\.crt}' | base64 -d > mycert.pem

# Verify key file
ls -la mycert.pem
file mycert.pem
# Output: PEM certificate file created
```

**Explanation:** Controller runs in kube-system namespace. Public key is safe to share. Can be committed to Git. Private key stays in cluster.

---

## Exercise 2: Create and Seal a Secret

**Solution:**
```bash
# Create plain secret locally (dry-run)
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123 \
  -n myapp --dry-run=client -o yaml > secret.yaml

# View unencrypted secret
cat secret.yaml
# Output: Shows base64 encoded values (not encrypted)

# Install kubeseal CLI (if not installed)
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/kubeseal-0.18.0-linux-amd64.tar.gz
tar xfz kubeseal-0.18.0-linux-amd64.tar.gz
sudo mv kubeseal /usr/local/bin/

# Seal the secret using public key
kubeseal -f secret.yaml -w sealed-secret.yaml -n myapp --cert mycert.pem

# View sealed secret
cat sealed-secret.yaml
# Output: Shows SealedSecret with encrypted encryptedData

# Compare sizes
wc -l secret.yaml sealed-secret.yaml
# Output: Sealed version larger due to encryption

# Verify different seal produces different output
kubeseal -f secret.yaml -w sealed-secret2.yaml -n myapp --cert mycert.pem
diff sealed-secret.yaml sealed-secret2.yaml
# Output: Different each time (includes nonce)
```

**Explanation:** Plain secret created locally. Sealed using public key. Encryption adds overhead but ensures security. Each seal is unique.

---

## Exercise 3: Decrypt Sealed Secret in Cluster

**Solution:**
```bash
# Create namespace
kubectl create namespace myapp

# Apply sealed secret
kubectl apply -f sealed-secret.yaml

# Verify SealedSecret created
kubectl get sealedsecret -n myapp
# Output: db-secret SealedSecret listed

# Check controller unsealed automatically
kubectl get secret -n myapp
# Output: db-secret secret appears after unsealing

# Verify secret decrypted
kubectl get secret db-secret -n myapp -o yaml
# Output: Shows username and password in base64 (not encrypted)

# Decode and view values
kubectl get secret db-secret -n myapp -o jsonpath='{.data.username}' | base64 -d
# Output: admin

kubectl get secret db-secret -n myapp -o jsonpath='{.data.password}' | base64 -d
# Output: secret123

# Try to read sealed secret from different cluster (should fail)
# Copy sealed-secret.yaml to another cluster
# Apply: kubectl apply -f sealed-secret.yaml
# Output: Error - cannot decrypt with different private key

# Check controller logs for unsealing
kubectl logs -n kube-system -l app.kubernetes.io/name=sealed-secrets | grep -i unseal
# Output: Shows unsealing operation logs
```

**Explanation:** SealedSecret decrypted only by controller with matching private key. Each cluster has unique key. Cannot decrypt on wrong cluster.

---

## Exercise 4: Use Secrets in Deployments

**Solution:**
```bash
# Create deployment using secret
kubectl apply -f - << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-secret
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
        image: nginx:latest
        ports:
        - containerPort: 8080
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
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: db-secret
EOF

# Verify deployment created
kubectl get deployment -n myapp
# Output: app-with-secret listed

# Check pod environment variables
kubectl exec -n myapp deployment/app-with-secret -- env | grep DB_
# Output: Shows DB_USER=admin, DB_PASSWORD=secret123

# Check mounted secret files
kubectl exec -n myapp deployment/app-with-secret -- ls /etc/secrets
# Output: username, password files

# Read secret from pod
kubectl exec -n myapp deployment/app-with-secret -- cat /etc/secrets/username
# Output: admin

# Test with multiple secrets
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: api-secret
  namespace: myapp
type: Opaque
data:
  api-key: YWJjZGVmMTIzNDU2
EOF

# Update deployment to use both secrets
kubectl set env deployment/app-with-secret \
  API_KEY=- \
  --from=secret/api-secret/api-key \
  -n myapp
```

**Explanation:** Secrets passed as env vars or mounted volumes. Pod cannot read other namespaces' secrets. RBAC controls who can access secrets.

---

## Exercise 5: Install ArgoCD Image Updater

**Solution:**
```bash
# Install image-updater
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml

# Wait for pod ready
kubectl wait --for=condition=ready pod -l app=argocd-image-updater \
  -n argocd --timeout=300s

# Verify pod running
kubectl get pods -n argocd -l app=argocd-image-updater
# Output: argocd-image-updater-xxxxx pod Running

# Check logs
kubectl logs -n argocd -l app=argocd-image-updater
# Output: Shows initialization and ready messages

# Check RBAC created
kubectl get serviceaccount -n argocd | grep image-updater
kubectl get clusterrole | grep image-updater

# Verify updater can access applications
kubectl auth can-i list applications \
  --as system:serviceaccount:argocd:argocd-image-updater \
  -n argocd
# Output: yes

# Check updater version
kubectl exec -n argocd deployment/argocd-image-updater -- argocd-image-updater version
# Output: Shows version information
```

**Explanation:** Image Updater runs as separate deployment. Needs serviceaccount with application access. RBAC controls what it can modify.

---

## Exercise 6: Enable Automated Image Updates

**Solution:**
```bash
# Create application with image update annotations
kubectl apply -f - << 'EOF'
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: image-updater-demo
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: myapp=myregistry.azurecr.io/myapp
    argocd-image-updater.argoproj.io/myapp.update-strategy: latest
    argocd-image-updater.argoproj.io/myapp.allow-tags: ".*"
    argocd-image-updater.argoproj.io/git-branch: image-updates
    argocd-image-updater.argoproj.io/write-back-method: git
spec:
  project: default
  source:
    repoURL: https://github.com/example/app
    targetRevision: main
    path: manifests/
  destination:
    server: https://kubernetes.default.svc
    namespace: default
EOF

# Verify application created
argocd app get image-updater-demo

# Check image updater logs for scanning
kubectl logs -n argocd -l app=argocd-image-updater -f
# Output: Shows scanning for new image tags

# Wait for automatic image update
# Should create commit in Git repo with updated tag

# Check Git log for image update commit
git log --oneline -n 5
# Output: Shows image tag update commits

# Verify deployment updated
kubectl get deployment -o jsonpath='{.items[0].spec.template.spec.containers[0].image}'
# Output: myapp:latest or new tag

# Check update history
argocd app history image-updater-demo
# Output: Shows sync operations from image updater
```

**Explanation:** Annotations configure image updater behavior. Scans registry for new tags. Auto-commits to Git with updated image. Triggers deployment.

---

## Exercise 7: Manage Image Scanning

**Solution:**
```bash
# Install Trivy image scanner
wget https://github.com/aquasecurity/trivy/releases/download/v0.40.0/trivy_0.40.0_Linux-64bit.tar.gz
tar xfz trivy_0.40.0_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/

# Scan image for vulnerabilities
trivy image myregistry.azurecr.io/myapp:1.0
# Output: Shows vulnerabilities found

# Create policy to block scan failures
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: ConfigMap
metadata:
  name: image-updater-config
  namespace: argocd
data:
  scan-config.yaml: |
    max-severity: HIGH
    ignore-unfixed: true
EOF

# Test scan in deployment validation
kubectl apply -f - << 'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: image-scan-job
  namespace: default
spec:
  template:
    spec:
      containers:
      - name: scanner
        image: aquasec/trivy:latest
        args: ["image", "myregistry.azurecr.io/myapp:vulnerable"]
      restartPolicy: Never
EOF

# Check scan results
kubectl logs -n default job/image-scan-job
# Output: Shows scan results and vulnerabilities

# Integrate with deployment validator
kubectl set image deployment/app-with-secret \
  myapp=myregistry.azurecr.io/myapp:safe-version \
  -n myapp
```

**Explanation:** Trivy scans images for CVEs. Can block unsafe images. Prevents deployment of vulnerable versions. Audit trail of scans.

---

## Exercise 8: Configure Image Registry Access

**Solution:**
```bash
# Create Docker config secret for registry auth
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.azurecr.io \
  --docker-username=username \
  --docker-password=password \
  --docker-email=email@example.com \
  -n myapp

# Seal the registry secret
kubectl get secret regcred -n myapp -o yaml > regcred.yaml
kubeseal -f regcred.yaml -w sealed-regcred.yaml -n myapp --cert mycert.pem

# Create ImagePullSecret in pod
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: app-with-registry-auth
  namespace: myapp
spec:
  imagePullSecrets:
  - name: regcred
  containers:
  - name: app
    image: myregistry.azurecr.io/myapp:1.0
EOF

# Verify pod can pull from registry
kubectl get pod -n myapp app-with-registry-auth
# Output: Pod should transition to Running

# Test with service account
kubectl patch serviceaccount default -n myapp \
  -p '{"imagePullSecrets": [{"name": "regcred"}]}'

# Configure image updater credentials
kubectl apply -f - << 'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: image-updater-registry-creds
  namespace: argocd
type: Opaque
stringData:
  username: your-username
  password: your-password
EOF

# Check image updater can access registry
kubectl logs -n argocd -l app=argocd-image-updater | grep registry
# Output: Shows successful registry access
```

**Explanation:** imagePullSecrets provide registry authentication. Can be seeded or stored in cluster. Service accounts inherit imagePullSecrets.

---

## Exercise 9: Implement Image Update Policies

**Solution:**
```bash
# Create ImagePolicy resource
kubectl apply -f - << 'EOF'
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: myapp-policy
  namespace: argocd
spec:
  imageRepositoryRef:
    name: myapp
  policy:
    semver:
      range: '>=1.0.0 <2.0.0'
EOF

# Create ImageRepository to scan
kubectl apply -f - << 'EOF'
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: myapp
  namespace: argocd
spec:
  image: myregistry.azurecr.io/myapp
  interval: 5m
  secretRef:
    name: image-updater-registry-creds
EOF

# Verify policy created
kubectl get imagepolicy -n argocd
# Output: myapp-policy listed

# Test policy with different versions
kubectl get imagepolicy myapp-policy -n argocd -o yaml
# Shows policy limits and matching versions

# Create multiple policies for different environments
kubectl apply -f - << 'EOF'
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: myapp-prod-policy
  namespace: argocd
spec:
  imageRepositoryRef:
    name: myapp
  policy:
    semver:
      range: '>=1.0.0 <1.2.0'
---
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: myapp-staging-policy
  namespace: argocd
spec:
  imageRepositoryRef:
    name: myapp
  policy:
    semver:
      range: '>=1.0.0'
EOF

# Verify policies configured
kubectl get imagepolicy -n argocd
# Output: Multiple policies with different version constraints
```

**Explanation:** ImagePolicy defines allowed versions. Semver ranges control updates. Different policies for different environments. Prevents unsafe version jumps.

---

## Exercise 10: Troubleshoot Secrets and Image Issues

**Solution:**
```bash
# Debug SealedSecret not decrypting
kubectl get sealedsecret -n myapp -o yaml
# Check spec.encryptedData is present

kubectl logs -n kube-system -l app.kubernetes.io/name=sealed-secrets | tail -20
# Look for unsealing errors

# Verify private key in cluster
kubectl get secret -n kube-system | grep sealed-secrets
# Should show sealing-key-xxxxx

# Try manual unseal (for debugging only)
kubeseal --recovery-unseal sealed-secret.yaml

# Debug image updater not scanning
kubectl logs -n argocd -l app=argocd-image-updater | grep -i "error\|failed"

# Check image updater RBAC
kubectl auth can-i list applications --as system:serviceaccount:argocd:argocd-image-updater -n argocd

# Verify registry credentials
kubectl get secret image-updater-registry-creds -n argocd -o yaml
# Check credentials base64 encoded properly

# Test registry access manually
kubectl run -n argocd registry-test --image=curlimages/curl -it \
  -- curl -u username:password https://myregistry.azurecr.io/v2/_catalog

# Debug image scanning failures
trivy image --severity HIGH,CRITICAL myregistry.azurecr.io/myapp:latest
# Shows detailed scan output

# Check application sync status
argocd app get image-updater-demo
# Look for sync status and error messages

# Review Git commits from image updater
git log --oneline -n 10 | grep -i "image\|update"
# Verify commits created

# Check secret reference in deployment
kubectl get deployment -n myapp -o yaml | grep -A5 "secretKeyRef"
# Verify secret names match
```

**Explanation:** Logs show detailed errors. Manual checks isolate issues. RBAC and credentials most common problems. Git history confirms automations.
