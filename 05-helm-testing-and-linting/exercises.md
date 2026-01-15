# 05-helm-testing-and-linting: Exercises

## Exercise 1: Lint Chart for Errors and Warnings
Perform static analysis on chart structure and configuration.

**Task:**
- Run `helm lint` on a chart
- Run `helm lint --strict` to enforce all rules
- Document all warnings found
- Fix each issue one by one
- Verify chart passes linting

## Exercise 2: Validate Chart Metadata
Ensure Chart.yaml has all required and recommended fields.

**Task:**
- Check Chart.yaml for required fields (apiVersion, name, version)
- Add optional fields (maintainers, keywords, home, sources)
- Validate semver format for version numbers
- Add proper chart description
- Verify with `helm lint`

## Exercise 3: Template Rendering with Dry-Run
Generate manifests without deploying resources.

**Task:**
- Render templates: `helm template chart`
- Use debug mode: `helm template --debug`
- Show specific template: `helm template -s templates/deployment.yaml`
- Render with custom values: `helm template -f values-prod.yaml`
- Compare outputs and identify differences

## Exercise 4: Validate Kubernetes Manifests
Ensure generated manifests are valid Kubernetes YAML.

**Task:**
- Generate manifests to file: `helm template > output.yaml`
- Validate with kubectl: `kubectl apply -f output.yaml --dry-run=client`
- Use schema validation: `--dry-run=server`
- Check for invalid field names
- Fix manifest issues

## Exercise 5: Create and Run Test Pods
Implement testing after chart deployment.

**Task:**
- Create test pod template in templates/tests/
- Define test validation steps (connectivity, readiness checks)
- Add proper hooks and cleanup policies
- Deploy chart: `helm install`
- Run tests: `helm test <release>`
- Verify test results

## Exercise 6: Implement Smoke Tests
Create simple validation tests for deployment.

**Task:**
- Create test pod that validates service connectivity
- Test health endpoint availability
- Check pod readiness
- Validate environment variables set correctly
- Create multiple test scenarios

## Exercise 7: Create Pre-Install Validation Hooks
Validate prerequisites before installation.

**Task:**
- Create job with `pre-install` hook
- Validate namespace exists
- Check required secrets are present
- Verify resource quotas available
- Document hook execution order

## Exercise 8: Implement Chart Schema Validation
Define and validate chart value schema.

**Task:**
- Create values.schema.json file
- Define types for all values
- Set required fields
- Add value constraints (min, max, enum)
- Validate with invalid values

## Exercise 9: Automate Testing in CI/CD
Set up automated chart testing in pipeline.

**Task:**
- Create test script that runs lint and template
- Add dry-run validation
- Test with multiple value files (dev, staging, prod)
- Add schema validation
- Document CI/CD integration

## Exercise 10: Create Complete Test Suite
Implement comprehensive testing for production chart.

**Task:**
- Run helm lint with all checks
- Validate templates with all value files
- Create and run multiple test pods
- Test helm upgrade and rollback
- Document test coverage and results
