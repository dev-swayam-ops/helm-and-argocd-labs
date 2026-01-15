# 12-argo-rollouts-and-progressive-delivery: Exercises

## Exercise 1: Install Argo Rollouts
Set up Argo Rollouts controller in the cluster.

**Task:**
- Install Argo Rollouts from official manifest
- Create argo-rollouts namespace
- Verify controller running
- Check controller logs
- Confirm CRD registered
- Test rollout resource creation

## Exercise 2: Create Basic Rollout
Deploy application using Rollout resource.

**Task:**
- Create Rollout with 3 replicas
- Define pod template
- Set image version
- Create associated service
- Verify pods running
- Check rollout status

## Exercise 3: Implement Canary Strategy
Configure gradual traffic shift to new version.

**Task:**
- Create Rollout with canary steps
- Set initial weight to 10%
- Add pause between steps
- Update image to new version
- Monitor canary deployment
- Verify traffic splitting

## Exercise 4: Configure Blue-Green Deployment
Set up instant switching between versions.

**Task:**
- Create Rollout with blue-green strategy
- Deploy version 1 (blue)
- Deploy version 2 (green)
- Switch active version
- Verify zero-downtime switch
- Test rollback to blue

## Exercise 5: Add Metrics Analysis
Configure health checks based on metrics.

**Solution:**
- Create AnalysisTemplate with metric query
- Define threshold values
- Connect to Prometheus
- Run AnalysisRun
- Evaluate metrics during rollout
- Trigger automatic rollback on failure

## Exercise 6: Implement Traffic Splitting
Use service mesh for fine-grained traffic control.

**Task:**
- Install service mesh (Istio or Flagger)
- Create VirtualService for traffic split
- Configure Destination Rules
- Update Rollout to use mesh
- Verify percentage-based routing
- Monitor traffic metrics

## Exercise 7: Configure Automated Rollback
Set up failure detection and automatic rollback.

**Task:**
- Create analysis template with failure conditions
- Set error rate threshold
- Configure rollback trigger
- Trigger intentional failure
- Verify automatic rollback
- Check rollback logs

## Exercise 8: Implement Multi-Wave Deployment
Deploy across multiple environments sequentially.

**Task:**
- Create Rollout with multiple steps
- Set different weights per wave
- Add manual approval gates
- Progress through waves
- Verify state at each step
- Document wave progression

## Exercise 9: Monitor Rollout Progress
Track deployment progress and metrics.

**Task:**
- Watch rollout status in real-time
- Check pod creation/termination
- Monitor traffic shifting
- Query metrics endpoint
- View event history
- Create rollout dashboard

## Exercise 10: Troubleshoot Rollout Issues
Diagnose and resolve rollout problems.

**Task:**
- Identify stuck rollout
- Check analysis failures
- Review metric query issues
- Fix traffic routing problems
- Recover from failed deployment
- Document troubleshooting steps
