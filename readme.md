### Argo CD
- A declarative, GitOps continuous delivery tool for Kubernetes

### CICD Pipelines
- Continuous Integration/ Continuous DElivery pipelines
- Incremental code changes built and tested in an automated manner
- COde delivered to production environments quickly and seemlessly

## IaC
- Argo CD allows continuous delivery of infrastructure

### Why Argo CD?
- Possible to fully automate application and lifecycle manegment
- Including the infrastructure
- It is meant specifically to build continuous delivery pipelines to build continuous delivery pipelines for infrastructure that is deployed to kubernetes.

### Introducing GitOps
- A awy of implementing continuous deployment for cloud native applications by using Git as a single source of truth for declarative infrastructure and applications


### How Does Argo CD Work?
- Git repositories are the source of truth
- Infra as code using Kubernetes manifests

### Kubernetes Manifests
- Kustomize applications
- helm charts
- ksonnet applications
- jsonnet files
- Plain old YAML/Json files
- Any custom config management tool using plugins

### Creating a local K3s cluster
- brew install kubectl
- brew install k3d
    - k3d versiob
- mkdir argocd
- cd argocd
- nano cluster-config.yaml
- k3d cluster create argocd-cluster --config ./cluster-config.yaml

### Architectural Overview
- ArgoCD consists of
    - API Server
    - Repository Service
    - Application Controller
- Interact with ArgoCD using UI or CLI, gRPC
- GitHub repo notifies ArgoCD using webhooks events when new changes are committed
- ArgoCD built on top of K8 and works with k8 clusters

### API Server
- Application management and status reporting
- Application operations e.g. sync, rollback, user-defined actions
- Credential management
- Authentication and auth delegation to external identify providers 
- RBAC enforcement
- Listener/forwarder for Git WEbhook events


### Repository Server
- Internal service which maintains a local cache of the Git repository
- COntains the applications manifests


### Application Controller
- Kubernetes controller which continuously monitors running applications
- COmpares the current, live state of the application against the target state
- Target state is specified in the repo
- Invokes user-defined hooks for lifecycle events

### Installing ARgoCD
- kubectl get namespace
- kubectl create ns argocd
- kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml -n argocd
- kubectl get deployment -n argocd
- kubectl get service -n argocd
- brew install argocd


### Expose ArgoCD API Server
- kubectl patch svc argocd-server -n argocd -p '{"spec": {"type":"NodePort"}}'
- kubectl -n argocd get services
- kubectl port-forward svc/argocd-server -n argocd 8080:443
- kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d && echo


### Lick ArgoCD with Github Repo
- Copy Github repos HTTP url
- Go to ArgoCD UI. Go to Settings --> Repositories --> Connect Repo Using HTTPS

### Create ArgoCD Apps
- Create ArgoCD Application to deploy application from Github
- Applications --> New APp


- kubectl -n default get all
- kubectl port-forward svc/nginx-service -n argocd 8081:80
