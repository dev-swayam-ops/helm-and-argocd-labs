# 04-helm-values-and-environments: Exercises

## Exercise 1: Create Multi-Environment Values Files
Design value files for different deployment environments.

**Task:**
- Create base values.yaml with common defaults
- Create values-dev.yaml for development (1 replica, low resources)
- Create values-staging.yaml for staging (2 replicas, medium resources)
- Create values-prod.yaml for production (3+ replicas, high resources)
- Show differences in resource allocation, image tags, and logging levels

## Exercise 2: Override Values at Installation
Learn various methods to override default values.

**Task:**
- Install chart with --set flag: `--set replicaCount=5`
- Install with values file: `-f values-prod.yaml`
- Combine multiple value files: `-f values.yaml -f values-prod.yaml`
- Override with command-line: `--set-string key=value`
- Verify applied values with `helm get values`

## Exercise 3: Implement Environment-Specific Secrets
Manage sensitive data per environment securely.

**Task:**
- Create secret values-dev-secrets.yaml with dev credentials
- Create secret values-prod-secrets.yaml with prod credentials
- Reference secrets in templates using environment variables
- Deploy same chart with different secrets per environment
- Verify pod environment variables differ

## Exercise 4: Design Values Hierarchy
Structure values for inheritance and override patterns.

**Task:**
- Create values.yaml with nested hierarchy
- Create global, database, cache sections
- Override specific sections per environment
- Show how child values override parent values
- Document inheritance rules

## Exercise 5: Implement Feature Flags
Use values to enable/disable features per environment.

**Task:**
- Create feature.enabled values for each environment
- Add conditional templates: `if .Values.features.cache.enabled`
- Deploy with different features in dev vs prod
- Show how same chart enables different features
- Document feature combinations per environment

## Exercise 6: Create Values Template Documentation
Generate schema and documentation for values file.

**Task:**
- Create detailed comments in values.yaml for each section
- Document type, required fields, and valid values
- Add examples for common configurations
- Create values.schema.json for validation
- Generate README documenting all values

## Exercise 7: Implement Value Validation
Validate values before deployment.

**Task:**
- Create values validation in templates using `required`
- Add type checking for critical values
- Implement range checking for resources
- Create conditional requirements (if X then Y required)
- Test validation with invalid and valid values

## Exercise 8: Design GitOps Value Management
Plan values management for GitOps workflows.

**Task:**
- Organize values in Git repo structure (environments/)
- Create values-staging.yaml and values-prod.yaml in repo
- Show how to sync from Git to cluster
- Document promotion workflow (dev→staging→prod)
- Implement kustomize overlays for values

## Exercise 9: Create Values Comparison Tool
Build ability to compare values across environments.

**Task:**
- Extract values from all releases: `helm get values <release>`
- Compare dev, staging, and prod configurations
- Generate comparison report showing differences
- Identify unintended configuration drift
- Use diff tools to highlight changes

## Exercise 10: Implement Complete Multi-Environment Workflow
Deploy application to all environments with proper values management.

**Task:**
- Create complete values files for all 3 environments
- Deploy to dev, staging, and prod namespaces
- Verify each environment has correct configuration
- Test environment-specific functionality
- Document deployment procedure for each environment
