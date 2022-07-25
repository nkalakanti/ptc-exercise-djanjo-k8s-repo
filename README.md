# Simple Hello Django Project DEMO :hammer::wrench:

This is a sample django project to Practice Docker, Kubernetes, HELM, EKS and ArgoCD

Requirments:

1- [Docker Desctop](https://docs.docker.com/desktop/install/windows-install/)

2- [Kind For Kubernetes Testing](https://github.com/kubernetes-sigs/kind/releases/tag/v0.14.0)

3- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

4- [aws-cli](https://aws.amazon.com/cli/)

5- [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

6- [HELM](https://helm.sh/docs/intro/install/)

## 1- To Run the application using Docker

```console
docker build -t hellodjango:latest .
docker run --rm -d  -p 80:80/tcp hellodjango:latest
```

## 2- Push the image to [DockerHub](https://hub.docker.com/)

```console
#login to your DockerHub account using the desktop application
docker tag hellodjango YOUR_DOCKERHUB_USERNAME/hellodjango
docker push YOUR_DOCKERHUB_USERNAME/hellodjango
#Check your image on DockerHub
```

## 3- Create [Kind](https://kind.sigs.k8s.io/) cluster for k8s deployment

```console
kind create cluster --name testing
kubectl get nodes
```

Update the Deployment file inside of the `kubernetes-manifests` to have your DockerHub username

## 4- Run the application using [kubernetes](https://kubernetes.io/)

```console
kubectl apply -f kubernetes-manifests/Deployment.yaml
kubectl apply -f kubernetes-manifests/Service.yaml

#check if the deployment is done
kubectl get deployment
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hellodjango   1/1     1            1           4m30s

#port forward to check the application
kubectl port-forward service/hellodjango-service 80:80

#Open Browser on localhost:80
```

## 5- Run the application on [EKS](https://aws.amazon.com/eks/)

Setup your AWS account:

- Create AWS account
- Navigate to the IAM service
- Go for users
- Select your username
- Select Security Credintials
- Create Access Key `store the access key some where save as you won't be able to view it again`

### Configure AWS-CLI

```console
aws --version
aws-cli/2.6.4 Python/3.9.11 Windows/10 exe/AMD64 prompt/off

aws configure
AWS Access Key ID [None]: YOUR_ACCESS_KEY
AWS Secret Access Key [None]: YOUR_SECRET_KEY
Default region name [None]: eu-west-1
Default output format [None]: json
```

### Create EKS Cluster

```console
eksctl create cluster --config-file EKS/cluster.yaml
#This will take between 15~20 min

#Make sure that the cluster is ready
kubectl get nodes
NAME                                           STATUS   ROLES    AGE     VERSION
ip-192-168-15-173.eu-west-1.compute.internal   Ready    <none>   5m31s   v1.20.15-eks-99076b2
ip-192-168-43-234.eu-west-1.compute.internal   Ready    <none>   5m31s   v1.20.15-eks-99076b2
ip-192-168-82-227.eu-west-1.compute.internal   Ready    <none>   5m29s   v1.20.15-eks-99076b2
```

### 5.1 Deploy the application using Kubernetes manifests

```console
kubectl apply -f kubernetes-manifests/Deployment.yaml
kubectl apply -f kubernetes-manifests/Service-aws.yaml

#Get the application IP
kubectl get svc
#Visit the loadbalancer URL
```

### 5.2 Deploy the application using [HELM](https://helm.sh/docs/intro/install/)

```console
helm version
version.BuildInfo{Version:"v3.6.2", GitCommit:"ee407bdf364942bcb8e8c665f82e15aa28009b71", GitTreeState:"clean", GoVersion:"go1.16.5"}

helm install hellodjango-chart hellodjangochart/ --values hellodjangochart/values.yaml

#Get the application IP
kubectl get svc
#Visit the loadbalancer URL
```

## 6- Deploy monitoring using [HELM](https://helm.sh/docs/intro/install/) and [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus)

```console
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
```

### Check the Graphana Dashboard

```console
kubectl -n monitoring port-forward deployment/prometheus-grafana 3000
kubectl patch svc prometheus-grafana -n monitoring -p '{"spec": {"type": "LoadBalancer"}}'
```

- username: admin
- password: prom-operator

## 7- Deploy [ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/)

```console
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 7.1 Expose ArgoCD service as a LoadBalancer Service

```console
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

### 7.2 Expose ArgoCD service using port-forward

```console
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### 7.3 Get Login Credentials

```console
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

- username: admin

# Cleanup process

## Kind

```console
kind delete cluster --name testing
```

## EKS

```console
eksctl delete cluster -f EKS/cluster.yaml
```
