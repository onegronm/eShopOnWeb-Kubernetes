# Microsoft eShopOnWeb ASP.NET Core Reference Application on Kubernetes

Sample ASP.NET Core reference application, powered by Microsoft, demonstrating deployment of a single-process (monolithic) application to a Kubernetes cluster.

## Running the sample using Docker

You can run the Web sample by running these commands from the root folder (where the .sln file is located):

```
docker-compose build
docker-compose up
```

You should be able to make requests to localhost:5106 for the Web project, and localhost:5200 for the Public API project once these commands complete. If you have any problems, especially with login, try from a new guest or incognito browser instance.

You can also run the applications by using the instructions located in their `Dockerfile` file in the root of each project. Again, run these commands from the root of the solution (where the .sln file is located).

## Planned features
- Minikube
- Draft
- Service with LoadBalancer
- Healthchecks
- Secrets
- Volumes
- WebUI
- ConfigMap
- Ingress Controller
- Pod Presets
- StatefulSet
- DeamonSet
- Resoure usage monitoring
- Autoscaling
- Namespace 
- Namespace quota
- RBAC
- Pod Security Policies
- Helm Chart
- Helm Repository
- Flux
- Terraform

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

```bash
minikube start
minikube dashboard
minikube service <service-name> # launches a web browser for you
minikube pause
minikube stop
minikube config set memory 16384 # increase default memory limit
minikube addons list


kubectl get po -A
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube --type=NodePort --port=8080
kubectl get services <deployment-name>
kubectl port-forward service/hello-minikube 7080:8080
kubectl describe pod <pod-id>
```
