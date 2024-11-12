# Microsoft eShopOnWeb ASP.NET Core Reference Application on Kubernetes

I took the sample ASP.NET Core reference application to demonstrate the deployment of a single-process (monolithic) application to a Kubernetes cluster.

## Kubernetes Features
- Deployment object
```yaml
kind: Deployment
metadata:
  name: eshoponweb
  labels:
    app: eshoponweb
spec:...
```
- Service with LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: eshoponweb
  labels:
    app: eshoponweb
spec:
  ports:
    - port: 80
  selector:
    app: eshoponweb
    tier: frontend
  type: LoadBalancer
```
- Healthchecks (liveness probe using an HTTP GET request)
```yaml
livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          timeoutSeconds: 30
```
- Secrets
- Volumes
```yaml
containers:
  - image: onegronm/eshoponweb:v1.1
    name: eshoponwebmvc
    ports:
    livenessProbe:
    env:
      volumeMounts: # you can also mount a ConfigMap as a data volume inside a container
                - name: config
                  mountPath: "/config"
                  readOnly: true
volumes:
# You set volumes at the Pod level, then mount them into containers inside that Pod
- name: config
  configMap:
    # Provide the name of the ConfigMap you want to mount.
    name: configmap-demo
    # An array of keys from the ConfigMap to create as files. This creates two files. If you omit the items array entirely, every key in the ConfigMap becomes a file with the same name as the key, and you get 4 files
    items:
    - key: "app.properties"
      path: "app.properties"
    - key: "user-interface.properties"
      path: "user-interface.properties"
```
- Persistent Volume OK
- Persistent Volume Claim OK
- SQL Server container OK
- AKS deployment OK
- Access the Web UI (deprecated in Azure. See the Workloads section instead)
- ConfigMap OK
- Ingress Controller
- Pod Presets
- StatefulSet
- ReplicaSet
- DeamonSet
- Resoure usage monitoring
- Resource quotas
- Autoscaling
- Namespace 
- Namespace quota
- RBAC
- Pod Security Policies
- Helm Chart
- Helm Repository
- Flux
- Terraform

## Building the sample using Docker

You can run the Web sample by running these commands from the root folder (where the .sln file is located):

```
docker-compose up -d --build
```

You should be able to make requests to localhost:5106 for the Web project, and localhost:5200 for the Public API project once these commands complete. If you have any problems, especially with login, try from a new guest or incognito browser instance.

You can also run the applications by using the instructions located in their `Dockerfile` file in the root of each project. Again, run these commands from the root of the solution (where the .sln file is located).

## Creating the AKS Cluster
```bash
az login
az account list --output table
az account set --subscription "Visual Studio Enterprise"
az group create -l eastus -n OmarResourceGroup
az aks create -g OmarResourceGroup -n OmarManagedCluster
az aks get-credentials --resource-group OmarResourceGroup --name OmarManagedCluster
kubectl get nodes
```

## Pushing the image to Docker Hub and deploying
1. Create the repository in Docker Hub onegronm/eshoponweb
2. Login to Docker Hub through the Docker Desktop client or by running ```docker login```
3. Run ```cd C:\Data\Code\eshoponweb```
4. Run ```docker build --pull -t onegronm/eshoponweb:v1.0 -f src/Web/Dockerfile .```
5. Run ```docker push onegronm/eshoponweb:v1.0```
6. Run ```cd eshoponweb/kubernetes```
7. Run ```kubectl apply -k ./```
8. Run ```kubectl get service eshoponweb``` and take note of the external IP address and port on the LoadBalancer
9. Open the external IP in the web browser

## Deploying a SQL Server container
1. Create SA password by running ```kubectl create secret generic mssql --from-literal=SA_PASSWORD="MyC0m9l&xP@ssw0rd"```
2. Create a manifest to define the storage class and the persistent volume claim (pvc.yaml). The storage class provisioner is azure-disk, because this Kubernetes cluster is in Azure
3. Run ```cd eshoponweb/kubernetes```
4. Run ```kubectl apply -k ./```
5. Verify the PVC with ```kubectl describe pvc mssql-data```. The returned metadata includes a value called Volume. This value maps to the name of the blob. The value for volume matches part of the name of the blob in the image from the Azure portal.
6. Verify the Persitent Volume with ```kubectl describe pv```
7. Create a manifest to describe the container based on the SQL Server mssql-server-linux Docker image (sql-deployment.yaml)
8. Run ```kubectl apply -k ./```
9. Verify the services are running. Run the following command ```kubectl get services```
10. Connect to the SQL Server instance in SSMS using the external Ip address from the pod

## Creating the databases
1. Update Startup.cs's ConfigureDevelopmentServices method as follows:
```csharp
public void ConfigureDevelopmentServices(IServiceCollection services)
{
    // use in-memory database
    //ConfigureTestingServices(services);

    // use real database
    ConfigureProductionServices(services);

}
```

2. Ensure connection strings in appsettings.json point to sql container in azure
```yaml
"ConnectionStrings": {
    "CatalogConnection": "Server=20.81.109.96;User=sa;Password=MyC0m9l&xP@ssw0rd;Database=CatalogDb;",
    "IdentityConnection": "Server=20.81.109.96;User=sa;Password=MyC0m9l&xP@ssw0rd;Database=Identity;"
  }
```

3. Open a command prompt in the Web folder and execute the following commands:
```bash
dotnet restore
dotnet tool restore
dotnet ef database update -c catalogcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj
dotnet ef database update -c appidentitydbcontext -p ../Infrastructure/Infrastructure.csproj -s Web.csproj
```
These commands will create two separate databases, one for the store's catalog data and shopping cart information, and one for the app's user credentials and identity data

4. Run the application: ```docker compose up -d --build``` and open http://localhost:5106/ in a browser. You should be able to make requests to localhost:5106 for the Web project, and localhost:5200

5. You should be able to log in using the demouser@microsoft.com account with password Pass@word1

## Deployment manifest reference
```yaml
apiVersion: apps/v1
kind: Deployment # provides declarative updates for Pods and ReplicaSets. Describe the desired state
metadata: # 
  labels:
    app.kubernetes.io/name: load-balancer-example # label for the deployment
  name: hello-world # the name of the deployment
spec:
  replicas: 5 # the Deployment creates five replicated Pods
  selector: # defines how the Deployment finds which Pods to manage
    matchLabels: # select a label that is defined in the Pod template (app.kubernetes.io/name: load-balancer-example)
      app.kubernetes.io/name: load-balancer-example
  template: # Pod template
    metadata:
      labels: # label the Pod
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers: # Pod runs one container
        - image: gcr.io/google-samples/node-hello:1.0 # which runs the node-hello image
          name: hello-world # name of the container
          ports:
            - containerPort: 8080
```

## kubectl Cheat Sheet
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## kubectl Commands
```bash
kubectl version
kubectl cluster-info
kubectl config view # view kubectl configuration
kubectl config get-contexts
kubectl config use-context <context_name> # switch context

# Services
kubectl get service <service_name> –watch
kubectl get services
kubectl get services <deployment-name>
kubectl get services -l <label>
kubectl delete service -l <label>
kubectl port-forward service/hello-minikube 7080:8080 # alternative to service
kubectl describe <service> # find out what port was opened externally

# Pods
kubectl get pods # list all pods in non-kubernetes namespaces
kubectl get pods -A # list all pods in all namespaces
kubectl get pods -l <label> # get pods with specified label
kubectl get pods --output=wide # get pods with local IPs
kubectl describe pod <pod-id>
kubectl label pods <pod-name> <label> # apply a new label to a pod
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}') # get the Pod name, and we'll store in the environment variable POD_NAME
echo Name of the Pod: $POD_NAME
# access the Pod through the API by running:
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/

# Nodes
kubectl get nodes

# Deployments
kubectl get deployments
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4 # create a deployment object
kubectl expose deployment hello-minikube --type=NodePort --port=8080 # makes the container accessible from outside the Kubernetes virtual network
kubectl delete deployment hello-node
kubectl describe deployment # see the name of the label created

# Proxy
kubectl proxy # forward traffic into cluster-wide, private network

# Replica Sets
kubectl get replicasets
kubectl describe replicasets

# Events
kubectl get events # view cluster events

# Exec
kubectl exec -ti $POD_NAME -- curl localhost:8080 # run a command inside a pod

# ConfigMap
kubectl get configmap
kubectl get configmap <name> -o yaml
```

## kubectl Recipes

### Troubleshooting
```bash
kubectl get # list resources
kubectl describe pods # show what containers are inside a pod and what images are used
kubectl logs # print the logs from a container in a pod
kubectl exec # execute a command on a container in a pod
kubectl exec $POD_NAME bash # execute commands on the container
```

### Scaling out
```bash
kubectl scale --replicas=4 -f helloworld-replcontroller.yml 
kubectl scale --replicas=4 <pod_name>
kubectl delete <controller_name>
kubectl get rs # get ReplicaSet
kubectl scale deployment <deployment> --replicas=4
kubectl autoscale deployment <name> --cpu-percent=50 --min=3 --max=10 # the autoscaler increases the pods up to a maximum of 10 instances and a minimum of 3 instances
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT
curl $(minikube ip):$NODE_PORT # curl to the exposed IP and port. Execute the command multiple times. We hit a different Pod with every request. This demonstrates that the load-balancing is working
```

### Simple deployment
```bash
kubectl get pods # monitor the deployment
kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml
kubectl get deployments # list deployments
kubectl describe deployments <name>
kubectl get replicasets
kubectl describe replicasets
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
kubectl get services my-service # get external IP
http://40.76.173.122:8080/ # access external IP from browser
kubectl delete services my-service
kubectl delete deployment hello-world
```

### Updating a pod
```bash
kubectl get pods # view running pods
kubectl describe pods # view current image version of the app
kubectl set image deployment/frontend www=image:v2 # rolling update "www" containers of "frontend" deployment, updating the image
kubectl rollout restart deployment deployment/frontend # restart deployment
kubectl get pods # check the status of the new pods
kubectl rollout status deployments/kubernetes-bootcamp # confirm the update
kubectl rollout status -w deployment/frontend # watch rolling update status of "frontend" deployment until completion
kubectl describe pods # view current image version
kubectl rollout undo deployments/kubernetes-bootcamp # rollback to the previous deployment
```

### Logging
```bash 
kubectl logs -f deployment/redis-leader
```

### Kustomize
```bash
kubectl kustomize <kustomization_directory> # view Resources found in a directory containing a kustomization file
kubectl apply -k <kustomization_directory> # apply resources
kubectl kustomize ./ # generate resources
kubectl apply -k ./ # apply changes
kubectl get -k ./ # view the deployment object
kubectl diff -k ./ # preview changes
kubectl delete -k ./ # delete deployment object
```

## Troubleshooting AKS
1. Nodes in NotReady status
In Azure, do Node pools > Update image
