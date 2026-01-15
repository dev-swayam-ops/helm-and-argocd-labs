# 01-helm-basics: Exercises

## Exercise 1: Create Your First Helm Chart
Generate a basic Helm chart and understand its structure.

**Task:**
- Create a new chart named "my-nginx-chart"
- Examine all files in the chart directory
- Describe the purpose of Chart.yaml, values.yaml, and templates/ folder
- Display the chart tree structure

## Exercise 2: Customize Chart Metadata
Modify chart information to match your application.

**Task:**
- Edit Chart.yaml in your chart
- Change the chart name to "webapp"
- Update the description to describe your application
- Set the version to "2.0.0"
- Run `helm lint` to verify changes
- Display the updated Chart.yaml

## Exercise 3: Explore Default Values
Understand chart configuration through values.yaml.

**Task:**
- Open values.yaml file
- Identify key configuration sections
- Note the default replica count
- Find image repository and tag settings
- Explain what each major section controls
- Create a custom values file: `custom-values.yaml`

## Exercise 4: Validate Chart Syntax
Ensure chart is valid before deployment.

**Task:**
- Run `helm lint` on your chart
- Fix any warnings or errors
- Run `helm template` to preview manifest output
- Compare template output with original YAML
- Validate manifest using `kubectl apply --dry-run=client -f <manifest>`

## Exercise 5: Install Chart with Default Values
Deploy a chart using built-in default values.

**Task:**
- Create a new namespace "chart-testing"
- Install your chart: `helm install my-app my-nginx-chart -n chart-testing`
- Verify release: `helm list -n chart-testing`
- Check deployed pods: `kubectl get pods -n chart-testing`
- Inspect release values: `helm get values my-app -n chart-testing`

## Exercise 6: Deploy with Custom Values
Override default values during installation.

**Task:**
- Install chart with custom values: `helm install web-app my-nginx-chart -n chart-testing --set replicaCount=3 --set image.tag=latest`
- Verify 3 pods are running
- Get the values: `helm get values web-app -n chart-testing`
- View the generated manifest: `helm get manifest web-app -n chart-testing`

## Exercise 7: Upgrade a Release
Update a deployment with new configuration.

**Task:**
- Upgrade the "my-app" release: `helm upgrade my-app my-nginx-chart -n chart-testing --set replicaCount=5`
- Check release history: `helm history my-app -n chart-testing`
- Verify 5 pods are running
- Get release info: `helm get all my-app -n chart-testing`

## Exercise 8: Rollback Release
Revert to a previous version of a release.

**Task:**
- View release history
- Rollback to revision 1: `helm rollback my-app 1 -n chart-testing`
- Verify rollback succeeded
- Check pod count returned to original (should be 1)
- View updated history

## Exercise 9: Compare Values Between Releases
Understand differences between deployed releases.

**Task:**
- Deploy two releases with different values
- Get values from first release
- Get values from second release
- Compare the differences
- Export values to files: `helm get values <release> > release-values.yaml`

## Exercise 10: Uninstall and Cleanup
Properly remove releases and resources.

**Task:**
- List all releases: `helm list -n chart-testing`
- Uninstall "my-app": `helm uninstall my-app -n chart-testing`
- Uninstall "web-app": `helm uninstall web-app -n chart-testing`
- Verify uninstallation: `helm list -n chart-testing` (should be empty)
- Delete namespace
- Remove custom values file
- Remove chart directory
