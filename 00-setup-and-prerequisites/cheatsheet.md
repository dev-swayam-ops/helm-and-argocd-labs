# 00-setup-and-prerequisites: Cheatsheet

## Docker Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `docker --version` | Check Docker version | `docker --version` |
| `docker run` | Run container | `docker run -d -p 8080:80 nginx` |
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers | `docker ps -a` |
| `docker stop <id>` | Stop container | `docker stop abc123def456` |
| `docker rm <id>` | Remove container | `docker rm abc123def456` |
| `docker logs <id>` | View container logs | `docker logs abc123def456` |
| `docker pull <image>` | Download image | `docker pull nginx` |

## Minikube Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `minikube version` | Check Minikube version | `minikube version` |
| `minikube start` | Start cluster | `minikube start --driver=docker --cpus=4 --memory=8192` |
| `minikube status` | Check cluster status | `minikube status` |
| `minikube stop` | Stop cluster | `minikube stop` |
| `minikube delete` | Delete cluster | `minikube delete` |
| `minikube dashboard` | Open dashboard | `minikube dashboard` |
| `minikube ip` | Get cluster IP | `minikube ip` |
| `minikube addons list` | List addons | `minikube addons list` |
| `minikube addons enable` | Enable addon | `minikube addons enable ingress` |

## kubectl Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl version` | Check kubectl version | `kubectl version --client` |
| `kubectl cluster-info` | Cluster information | `kubectl cluster-info` |
| `kubectl get nodes` | List nodes | `kubectl get nodes` |
| `kubectl get namespaces` | List namespaces | `kubectl get namespaces` |
| `kubectl create namespace` | Create namespace | `kubectl create namespace my-app` |
| `kubectl delete namespace` | Delete namespace | `kubectl delete namespace my-app` |
| `kubectl get pods` | List pods | `kubectl get pods -n my-app` |
| `kubectl describe pod` | Pod details | `kubectl describe pod nginx-xxx -n my-app` |
| `kubectl logs <pod>` | Pod logs | `kubectl logs nginx-xxx -n my-app` |
| `kubectl config current-context` | Current context | `kubectl config current-context` |
| `kubectl config get-contexts` | List contexts | `kubectl config get-contexts` |
| `kubectl config view` | View kubeconfig | `kubectl config view` |

## Helm Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `helm version` | Check Helm version | `helm version` |
| `helm repo add` | Add repository | `helm repo add bitnami https://charts.bitnami.com/bitnami` |
| `helm repo list` | List repositories | `helm repo list` |
| `helm repo update` | Update repo metadata | `helm repo update` |
| `helm search repo` | Search charts | `helm search repo nginx` |
| `helm show chart` | Chart information | `helm show chart bitnami/nginx` |
| `helm show values` | Chart values | `helm show values bitnami/nginx` |
| `helm show readme` | Chart README | `helm show readme bitnami/nginx` |
| `helm install` | Deploy chart | `helm install my-release bitnami/nginx --namespace my-app` |
| `helm list` | List releases | `helm list -A` |
| `helm get values` | Get release values | `helm get values my-release` |
| `helm upgrade` | Update release | `helm upgrade my-release bitnami/nginx --set replicaCount=3` |
| `helm uninstall` | Delete release | `helm uninstall my-release -n my-app` |
| `helm history` | Release history | `helm history my-release -n my-app` |

## Installation URLs

| Tool | URL |
|------|-----|
| Docker | https://www.docker.com/products/docker-desktop |
| Minikube | https://minikube.sigs.k8s.io/docs/start/ |
| kubectl | https://kubernetes.io/docs/tasks/tools/ |
| Helm | https://helm.sh/docs/intro/install/ |

## Popular Helm Repositories

| Repository | URL |
|------------|-----|
| Bitnami | https://charts.bitnami.com/bitnami |
| Stable (archived) | https://charts.helm.sh/stable |
| Jetstack | https://charts.jetstack.io |
| NGINX | https://helm.nginx.com |
| ArtifactHub | https://artifacthub.io |

## Resource Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-n, --namespace` | Target namespace | `-n my-app` |
| `-A, --all-namespaces` | All namespaces | `-A` |
| `--driver` | Minikube driver | `--driver=docker` |
| `--cpus` | CPU allocation | `--cpus=4` |
| `--memory` | Memory allocation | `--memory=8192` |
| `--set` | Override values | `--set replicaCount=3` |
| `--values, -f` | Values file | `-f values.yaml` |
| `-d` | Detached mode | `-d` (Docker) |
| `-p` | Port mapping | `-p 8080:80` (Docker) |
