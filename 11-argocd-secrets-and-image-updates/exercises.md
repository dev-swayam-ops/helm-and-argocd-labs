# 11-argocd-secrets-and-image-updates: Exercises

## Exercise 1: Install and Verify Sealed Secrets
Set up Sealed Secrets controller in the cluster.

**Task:**
- Install sealed-secrets from official manifest
- Verify controller running
- Export sealing key (public key)
- Confirm key file generated
- Test controller health
- Check logs for errors

## Exercise 2: Create and Seal a Secret
Encrypt a secret using Sealed Secrets.

**Task:**
- Create plain secret YAML locally
- Install kubeseal CLI tool
- Seal secret with public key
- Generate sealed-secret.yaml
- Verify encrypted content
- Compare plain vs sealed

## Exercise 3: Decrypt Sealed Secret in Cluster
Apply sealed secret and verify decryption.

**Task:**
- Apply SealedSecret to cluster
- Controller unseals automatically
- Verify secret appears as regular Secret
- Check secret values decrypted
- Confirm only cluster can decrypt
- Export sealing key from cluster

## Exercise 4: Use Secrets in Deployments
Reference sealed secrets in pod environment.

**Task:**
- Create deployment with secret references
- Use secretKeyRef in env vars
- Mount secrets as volumes
- Verify pod access to secrets
- Test multiple secret references
- Document secret usage patterns

## Exercise 5: Install ArgoCD Image Updater
Set up automatic image update controller.

**Task:**
- Install image-updater from manifest
- Verify pod running in argocd namespace
- Check logs for startup messages
- Configure registry credentials
- Test image scanning
- Document updater configuration

## Exercise 6: Enable Automated Image Updates
Configure automatic image tag updates.

**Task:**
- Create application with image annotations
- Set update strategy (latest, semver)
- Configure Git commit credentials
- Enable auto-push of changes
- Monitor image tag updates
- Verify Git commits created

## Exercise 7: Manage Image Scanning
Implement vulnerability scanning before deployment.

**Task:**
- Install image scanning tool (Trivy or Snyk)
- Configure image scanning policy
- Set scan failure policies
- Test image scanning workflow
- Prevent vulnerable image deployment
- Document scanning procedures

## Exercise 8: Configure Image Registry Access
Set up credentials for private image registries.

**Task:**
- Create registry credentials secret
- Configure registry authentication
- Test registry access
- Set up multiple registries
- Configure fallback registries
- Document registry configuration

## Exercise 9: Implement Image Update Policies
Define rules for automatic image updates.

**Task:**
- Create ImagePolicy resource
- Set allowed update ranges
- Configure update triggers
- Restrict update sources
- Test policy enforcement
- Document update rules

## Exercise 10: Troubleshoot Secrets and Image Issues
Diagnose and resolve configuration problems.

**Task:**
- Debug secret decryption failures
- Resolve image update permissions
- Fix registry credential issues
- Check image scanning failures
- Review logs for errors
- Document troubleshooting procedures
