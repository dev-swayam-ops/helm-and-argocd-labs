# 00-setup-and-prerequisites: Exercises

## Exercise 1: Verify Docker Installation
Confirm Docker is installed and running correctly. Run a simple nginx container and access it.

**Task:** 
- Check Docker version
- Run `docker run -d -p 8080:80 --name test-nginx nginx`
- Access http://localhost:8080 from your browser
- Stop and remove the container

## Exercise 2: Configure Minikube with Custom Resources
Set up Minikube with specific resource allocation for better performance.

**Task:**
- Delete existing Minikube (if any)
- Start Minikube with 4 CPUs and 8GB memory
- Set ingress addon enabled
- Verify with `minikube status`

## Exercise 3: Explore kubectl Contexts
Understand how kubectl manages cluster connections and contexts.

**Task:**
- Display current context: `kubectl config current-context`
- List all contexts: `kubectl config get-contexts`
- View the kubeconfig file: `kubectl config view`
- Understand the output structure

## Exercise 4: Create and Inspect Kubernetes Namespaces
Learn namespace management in Kubernetes.

**Task:**
- Create a namespace called "my-app"
- Create another namespace called "testing"
- List all namespaces
- Get details of "default" namespace using `kubectl describe namespace default`
- Delete one of the created namespaces

## Exercise 5: Initialize Helm and Add Repositories
Set up Helm for chart management and explore available charts.

**Task:**
- Display Helm version
- Add the stable, bitnami, and jetstack repositories
- List all added repositories
- Search for a chart: `helm search repo nginx`
- Show detailed chart information with `helm show chart bitnami/nginx`

## Exercise 6: Deploy Your First Helm Chart
Gain hands-on experience deploying an application using Helm.

**Task:**
- Create a new namespace "my-app-ns"
- Deploy bitnami/nginx chart with release name "web-server"
- Verify the deployment with `helm list`
- Check pods: `kubectl get pods -n my-app-ns`
- View release history: `helm history web-server -n my-app-ns`

## Exercise 7: Manage Helm Values
Understand how to customize Helm deployments using values.

**Task:**
- Deploy a chart with custom values: `helm install my-release bitnami/nginx --set replicaCount=3`
- Verify replicas created: `kubectl get pods`
- Get values of deployed release: `helm get values my-release`
- Update release: `helm upgrade my-release bitnami/nginx --set replicaCount=5`

## Exercise 8: Examine Kubernetes Resources
Explore different resource types in your cluster.

**Task:**
- List all resources: `kubectl api-resources`
- Get pods in default namespace: `kubectl get pods -A`
- Describe a Helm-deployed pod: `kubectl describe pod <pod-name>`
- View logs from a pod: `kubectl logs <pod-name>`

## Exercise 9: Helm Chart Search and Inspection
Learn to find and examine available charts before deployment.

**Task:**
- Search for "MySQL" in helm repositories
- Inspect MySQL chart: `helm show values bitnami/mysql | head -20`
- View chart README: `helm show readme bitnami/mysql`
- Compare chart versions: `helm search repo mysql --versions`

## Exercise 10: Cleanup and Verification
Ensure proper cleanup of all test resources.

**Task:**
- List all Helm releases: `helm list -A`
- Delete all created releases
- Delete all created namespaces
- Verify cleanup: `kubectl get namespaces` and `helm list -A` should be empty
- Stop Minikube: `minikube stop`
