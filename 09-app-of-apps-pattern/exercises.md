# 09-app-of-apps-pattern: Exercises

## Exercise 1: Create Multi-Tier Directory Structure
Set up repository with infrastructure, platforms, and applications layers.

**Task:**
- Create infrastructure directory with namespaces and RBAC
- Create platforms directory with data stores
- Create applications directory with business logic
- Organize into logical subdirectories
- Commit to Git repository
- Verify directory structure

## Exercise 2: Create Infrastructure Applications
Deploy foundational cluster resources using ArgoCD applications.

**Task:**
- Create application for namespaces
- Create application for RBAC configuration
- Create application for secrets management
- Deploy infrastructure applications
- Verify all resources created
- Check application dependencies

## Exercise 3: Create Platform Applications
Set up shared services and data stores for applications.

**Task:**
- Create application for databases (PostgreSQL, MySQL)
- Create application for message queues (Redis, RabbitMQ)
- Create application for logging infrastructure
- Deploy platform applications
- Verify services are running
- Test connectivity between services

## Exercise 4: Create Business Application Layer
Deploy actual microservices and applications.

**Task:**
- Create applications for frontend services
- Create applications for backend services
- Create applications for API gateways
- Configure service dependencies
- Deploy applications
- Verify inter-service communication

## Exercise 5: Implement Parent Application (Bootstrap)
Create root application that manages all child applications.

**Task:**
- Create parent application manifest
- Configure to reference all child applications
- Set syncPolicy for parent
- Deploy parent application
- Verify child applications created automatically
- Check hierarchical structure in ArgoCD UI

## Exercise 6: Configure Sync Waves
Implement deployment ordering using sync waves.

**Task:**
- Add wave 0 for infrastructure
- Add wave 1 for platforms
- Add wave 2 for applications
- Deploy and observe order
- Verify resources created sequentially
- Prevent resource conflicts with correct ordering

## Exercise 7: Add Health Checks and Dependencies
Implement readiness gates between application layers.

**Task:**
- Add health checks to services
- Configure dependency conditions
- Set wait times between waves
- Test deployment with failed dependency
- Verify sync stops until dependency healthy
- Implement retry logic

## Exercise 8: Implement Cascading Deletion
Set up proper cleanup when deleting parent application.

**Task:**
- Add finalizers to parent application
- Delete parent application
- Verify all child applications deleted
- Verify all resources cleaned up
- Check namespaces removed
- Implement deletion checklist

## Exercise 9: Create Templated App-of-Apps
Use Helm or Kustomize to generate app-of-apps manifests.

**Task:**
- Create Helm chart for app-of-apps
- Parametrize environment-specific values
- Generate manifests for dev/staging/prod
- Deploy templated app-of-apps
- Modify values and verify changes propagate
- Reduce manifest duplication

## Exercise 10: Troubleshoot App-of-Apps Issues
Diagnose and resolve common app-of-apps problems.

**Task:**
- Identify partial sync failures
- Check child application statuses
- Review application controller logs
- Fix configuration issues
- Verify cascading operations
- Implement monitoring and alerting
