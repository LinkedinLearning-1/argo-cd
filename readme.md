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


### Changing Argocd password
- argocd login localhost:8080
- yes
- argocd account update-password

### Syncing updated infrastructure manifests
- Check deployment replicas
- Update replicas in deployment
- Change number of replicas to 2
- Commit and push to master branch in github
- Change reflected in argocd 
- Sync in argocd
- kubectl -n default get replicaset


### Configuring deployments using kiubectl
- kubectl scale deploy nginx-deployment --replicas 3
- kubectl get all
- Current state does not match with source of truth

### Automated Sync, automated pruning and self-healing
- Go to app details
- Sync Policy --> Enable Auto Sync
- Automated Sync will not delete resources
- If you want to autoamted sync to be able to delete resource --> Enable Prune resources
- Changes made to the live cluster will not trigger automated sync. If you manually use kubectl to update your application. It is possible that your live deployment will be out of sync with what you have in Git. Enable self-healing to avoid this. With this setting the live state of our deployment will always match what we have in Git.

## Deploying infrastructure to external kubernetes cluster
### Creatiung external K3s cluster
- IP = ´ifconfig en0 | grep inet | grep -v inet6 | awk '{print $2}'´ && echo $IP.
prints implementing
- Define dev-cluster-config.yaml
- Create a new cluster 
    - k3d cluster create loony-dev-cluster --config dev-cluter-config.yaml
- In order to work with this external cluster, argocd needs to access this cluster by IP address
- Checks clusters
    - nano $HOME/.kube/config
- For deployment to an external cluster outside of where argocd is running, we need to access to an external IP address of that cluster
- Change IP in cionfig file for the external cluster to IP copied earlier. 
- kubectl version

### Creating new argocd project using the cli
- kubectl config get-contexts -o name
- kubectl config use-context k3d-argocd-cluster
- Use ARgocd from CLI
    - argocd cluster add k3d-loony-dev-cluster --name dev-cluster
    - Allows argocd to learn of the existance of the cluster and be able to deploy applications to this cluster.
- View projects in Argocd
    - argocd login localhost:8080 --insecure
    - argocd proj list
    - argocd repo list
    - argocd app list
- nano $HOME/.kube/config
    - copy server ip for dev cluster
- Create new argocd application within a new project
    - argocd proj create loony-argocd -d https://192.168.0.150:51766, default -s https://github.com/loonyuser/loony-argocd-public-repo

### Building and registering custom docker image
- Create a private repo in Github called loony-argocd-private-repo
- clone repo
- mkdir hello_app && hello_app
- mkdir app && cd app
- nano main.py
```
from flash import Flask
app = Flask(__name__)
@app.route('/')
def msg():
    return "Hello World! Deployed using Argo CD!"
if __name__ == "__main__":
    app-run(host='0.0.0.0')
```
- nano requirements.txt
    - Flask
- nano Dockerfile
``` 
FROM python:3.7
RUN mkdir /app
WORKDIR /app
ADD . /app/
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "/app/main.py"]
```
- docker build -f Dockerfile -t clouduserloony/hello-app:v1 .
- docker image ls
- Push to dockerhub
- Define secrets for kubernetes cluster for deployment to pull image from dockerhub
- kubectl config get-contexts -o name
- kubectl config use-context k3d-loony-dev-cluster
- kubectl get secret 
- nano $HOME/.docker/config.json
- kubectl create secret docker-registry regcred \
--docker-server=https://index.docker.io/v2/ \
--docker-username=clouduserloony \
--docker-password=Witches2019\
docker-email=cloud.user@loonycorn.com
- Create docker credentials to create a new secrets that our deployment can access
- Kubernetes deployment will use this secrets to pull custom image of python from dockerhub

### Kubernetes Deployment
```  
kind: Deployment
metadata:
    labels:
        app: hello-app
    name: hello-deployment
    annotations:
        link.argocd.argoproj.io/external-link: <dockerhub-user-account-link>
spec:
    replicas: 1
    selector:
        matchLabels:
            app: hello-app
        template:
            metadata:
                labels:
                    app: hello-app
                spec:
                    imagePullSEcrets:
                    - name: regcred
                    containers:
                    - name: hello-deployment
                      image: clouduserlony/hello-app:v1
                        imagePullPolicy:Always
```
- nano service.yaml
``` 
apiVersion: v1
kind: Service
metadata: 
    name: hello-app-service
spec:
    selector:
        app:hello-app
    ports:
    - protocol: "TCP"
      port: 6000
      targetPort: 5000
    type: NodePort
```
- Add all files to git

### Deploy App to external cluster using ARgocd
- Add git repo to argocd using interface
- argocd proj add-source loony-argocd https://github.com/loonyuser/loony-argocd-private-repo
- Create new app to manage new application for hello world python app.-
- select cluster for deploying app to.
- kubectl get all
- kubectl port-forward svc/hello-app-service -n default 8081:6000

### Update custom application
- nano main.py
- Change message
- docker build
- docker docker push
- nano deployment.yaml
- update version
- add and push git commit
- kubectl port-forward svc/hello-app-service -n default 8081:6000