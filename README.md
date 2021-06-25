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
- AKS

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

```bash
minikube start
minikube dashboard
minikube service <service-name> # launches a web browser for you
minikube pause
minikube stop
minikube config set memory 16384 # increase default memory limit
minikube addons list

kubectl version
kubectl config view # view kubectl configuration
kubectl cluster-info

kubectl get po -A
kubectl get nodes
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl get deployments
kubectl expose deployment hello-minikube --type=NodePort --port=8080 # make container accessible from outside the Kubernetes virtual network
kubectl port-forward service/hello-minikube 7080:8080 # alternative to service
kubectl proxy # forward traffic into cluster-wide, private network
kubectl get services <deployment-name>
kubectl describe pod <pod-id>
kubectl get events # view cluster events
kubectl delete deployment hello-node

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}') #  get the Pod name, and we'll store in the environment variable POD_NAME
echo Name of the Pod: $POD_NAME
# access the Pod through the API by running:
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/

# Troubleshooting
kubectl get # list resources
kubectl describe pods # show what containers are inside a pod and what images are used
kubectl logs # print the logs from a container in a pod
kubectl exec # execute a command on a container in a pod

kubectl exec $POD_NAME bash # execute commands on the container

```
