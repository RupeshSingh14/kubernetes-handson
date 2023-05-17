# Kubernetes

Description: Contains handson kubernetes (k8's) projects from beginner to expert level from context of a developer, kubernetes commands and their usage.

[[_TOC_]]

* It is manager for application containers in a cluster ie.. a set of machines(physical or virtual) connected to a network
* It is made up of :
1. Master aka control plane. 1 or more(for prod environments for high availablity)
2. Nodes (workers) which can be configured upto 5000

## kind/ minikube
to set up kubernetes cluster for learning (local cluster)

## kubectl
CLI tool to interact with k8s master / api-server

## master/control-plane components

### api-server
APIs for clients to talk to the cluster and create workloads. we talk only to api-server via kubectl command

### etcd (et-c-d) 
A distributed key-value store to store cluster data.. pods, status all, info

### controller-manager
A process which continously monitors workloads/ node etc. 

### scheduler
workload scheduler. assigning pods

## node (each nodes contains)

### kubelet
An agent which creates containers and monitors

### container runtime (docker)
To create containers, we need to have container runtime on the node

### kube-proxy 
maintains the network rules on the node for communication among workloads in the cluster.


```
C:\Users\rupesh>kubectl version --output=yaml
clientVersion:
  buildDate: "2022-09-21T14:33:49Z"
  compiler: gc
  gitCommit: 5835544ca568b757a8ecae5c153f317e5736700e
  gitTreeState: clean
  gitVersion: v1.25.2
  goVersion: go1.19.1
  major: "1"
  minor: "25"
  platform: windows/amd64
kustomizeVersion: v4.5.7
```

```
kind create cluster --config cluster.yaml
```
to create a cluster whose configuration is defined in current directory inside cluster.yaml

```
$HOME/.kube/config
``` 
default defined path which contains configuration info for the cluster created
However, if we want to change its locations, we can do that by defining an environment variable like - environment variable KUBECONFIG=a/b/c

* kubectl command first looks for cluster info in this config file and then talks the clusters control plane for any command we issue
* Even if cluster run in cloud, we can download a copy of config file and keep it at desired location in our local for working and issuing command with kubectl. In such cases kubectl will read the config first and then connect to cloud cluster.

```
kubectl get nodes
``` 
shows all the running nodes in the cluster

```
kubectl cluster-info --context kind-dev-cluster
Kubernetes control plane is running at https://127.0.0.1:55917
CoreDNS is running at https://127.0.0.1:55917/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
to start a default defined cluster using kind

```
kubectl get nodes
NAME                        STATUS   ROLES           AGE     VERSION
dev-cluster-control-plane   Ready    control-plane   5m6s    v1.25.3
dev-cluster-worker          Ready    <none>          4m19s   v1.25.3
dev-cluster-worker2         Ready    <none>          4m32s   v1.25.3
```

If we get in to cluster control plane/master
```
root@dev-cluster-control-plane:/# cd /etc/kubernetes/manifests/
root@dev-cluster-control-plane:/etc/kubernetes/manifests# ls -l
total 16
-rw------- 1 root root 2396 Mar  5 06:00 etcd.yaml
-rw------- 1 root root 3873 Mar  5 06:00 kube-apiserver.yaml
-rw------- 1 root root 3412 Mar  5 06:00 kube-controller-manager.yaml
-rw------- 1 root root 1440 Mar  5 06:00 kube-scheduler.yaml
```

```
ps -aux
```
to check on all processes inside any linux vm or container

```
kind delete cluster --name dev-cluster
``` 
to delete a cluster with name "dev-cluster". It will also, remove the config file details the stored at .kube folfer.

## K8's Jargons
1. Workload is an application running in k8s cluster.
2. Pod is the basic building block to create workload.
3. It is the smallest deployable unit in K8s which can run one or more containers but out of all the containers a pod runs only one of the container is your app container. And all other containers are helpers aka Sidecar containers.
4. A pod represents a VM and containers represent the process. 

K8s resource yaml format

```
apiVersion: [api-version]
kind: [k8s workload type]
metadata:
  [name of our resource, additional labels]
spec:
  [Input parameters to create the resources. this will change, depends on workload type]
  [refer to the documentation]
```  

```
kubectl get pod - to check for all pods in the cluster
kubectl create -f simple-pod.yaml   - to create a pod from the simple-pod.yaml specifcation in the current directory.
kubectl delete -f simple-pod.yaml   - to delete the same.
kubectl describe pod - to get info on pods
```

Inside the description, Events section helps to understand the process of pod creation
```
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m39s  default-scheduler  Successfully assigned default/my-pod to dev-cluster-worker
  Normal  Pulling    2m38s  kubelet            Pulling image "nginx"
  Normal  Pulled     2m13s  kubelet            Successfully pulled image "nginx" in 24.5904644s
  Normal  Created    2m13s  kubelet            Created container nginx
  Normal  Started    2m13s  kubelet            Started container nginx
```

```
kubectl apply -f pod.yaml
```
to apply any changes in specification in a running pod, this will cause restart of pod with updated configuration.


### Image pull backoff 
when the image defined in pod spec is not available in repository, retries of pulling the image are retried by the kublet in the node with some time gap, this is called image pull backoff. This can happen when image version is wronly defined, image is not available indefined repo, internet issues or like wise issues.

### Crash loop backoff
when the image as defined in pod spec, fails to run or goes down, retries of pulling the image and running it is performed by the 
kublet in the node, this is called crash loop backoff. 

```
kubectl get pod
NAME     READY   STATUS    RESTARTS   AGE
my-pod   1/1     Running   0          53m
```

Ready 1/1 - means only one pod has to be created and that has been done

### Status 
``` 
Pending - node is yet to be assigned
ContainerCreating - Kubelet is working on creating container
Running - kubelet started the container
ErrImagePull/ ImagePullBackOff - Failed pulling image. Kubelet will retry with some delay
Completed - Container did its job and exited successfully
Error - Container exited with error
CrashLoopBackOff - Problem running in the container but no issues while pulling it. Kubelet retries to run it with some delay
Terminating - Pod is getting deleted.
```
```
kubectl describe pod pod-3 - to see only details of pod-3
kubectl get pod pod-2 - to see only status of pod-2
```

* labels tag is used in metadata to catogerize the pods. This is helpful to uniquely query a category of pods when we have 100s of pods running in a cluster as below

```
kubectl get pod -l dept=dept-1
NAME    READY   STATUS    RESTARTS   AGE
pod-1   1/1     Running   0          5m43s
pod-2   1/1     Running   0          5m43s
```

Other ways of querying pods 
```
kubectl get pod -l team=team-a
kubectl get pod -l team!=team-a
kubectl get pod -l dept=dept-1, team=team-a
```

```
kubectl get pod -o wide - to get other info like nodes and ip, -o is output
kubectl get pod pod-1 -o yaml - to get all info of pod-1 output in yaml format
kubectl delete pod pod-2 - to delete a pod named pod-2
```

* Kublet(the agent in nodes) has default behaviour to restart any pod if the not keeping running. This should not happen in case where pod is down after it has completed its work. We can override this restart behaviour by defining restartPolicy like below and then run the pod

```
apiVersion: v1
kind: Pod
metadata: 
  name: my-pod
spec:
  restartPolicy: OnFailure # Always(by default), Never and OnFailure
  containers:
  - name: ubuntu
    image: ubuntu
```

|Docker     |  Kubernetes | Description |
|:----------|:------------|:------------|
|Entrypoint | Command     | command option defined will override any entrypoint defined for image |
|CMD        | args        | args option defined will be unable to override any entrypoint defined for image |


Example:

```
apiVersion: v1
kind: Pod
metadata: 
  name: my-pod
spec:
  restartPolicy: OnFailure # Always(by default), Never and OnFailure
  #terminationGracePeriodSeconds: 1   #Kills the process without any wait of sleep
  containers:
  - name: ubuntu
    image: ubuntu
    #args:         # args option defined will be unable to override any entrypoint defined for image
    #-  "sleep"
    #-  "3600"
	command:       # args option defined will be unable to override any entrypoint defined for image
	- "date"
```

`kubectl logs my-pod` : to check the container logs produced by pod named my-pod

`kubectl exec -it my-pod -- bash`: to enter (exec) inside the running pod named my-pod in interactive mode and open a bash shell
and exit to exit

`kubectl logs my-pod -c nginx-1` : to check the container logs for a specifc container named nginx-1 for pod named my-pod which has multiple containers

`kubectl exec -it my-pod -c util -- bash` : to get into the specific container named util incase of multicontainers running in a pod

`kubectl port-forward [pod-name] 8080:80` : to access our application APIs from our host for debuging.

* Helm is used as build tool for k8s and can let us have defined variables in pod yaml spec.

* A multi container pod is like a VM itself, where pod is the VM running many containers like processes. If we check the hostname for all
these containers all with have hostname as its pod.

* 1 app per pod should be run

* Replica Set is a low level resource that is part of controller manager and that manages pod ie.. it ensures that our desired replicas for the given pod spec are running all the time.

* Replica set has restartPolicy: Always

*Deployment manages Replica Set. To manage resources, we use metadata.labels

* Replica set is defined like pod spec in yaml

```
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name:my-rs
spec:
  selector:
    matchLabels:
	  app: my-app
	replicas: 3   
	template: 
	 {spec of pod}
```   

`kubectl get replicaset or kubectl get rs`:  to get all the replica sets

`kubectl describe replicaset / rs`: to describe the replica set

`kubectl delete pod/my-replicaset`: to delete a pod named my-replicaset

`kubectl delete pod --all`: to delete all the pods

Complex matchExpressions can be like below:

```
spec:
  selector:
    matchExpressions:
    - key: "team"
	  operator: In      #Not In can be also be used
	  values: [ "team-a", "team-b"]
```

`Deployment`: use to create and manage workload via replicaset. It manages replica set.

**1 Deployment ~ 1 Microservice**

* Deployment is List of ReplicaSet which is List of Pod

* At a time when we run the Deployment, only one Replica Set will be active. In production, when we would need to deploy new version of our code we will make changes in deployment file. k8s will create a new replica set equivalent for this new version. Once the pods are deployed and running, the older version of replica set and its pod will get removed by k8s for zero down time deployment

`kubectl get deploy / deployment`: to get the list of running deployment in namespace.

`kubectl create -f [file-name]`: to create resource in the given file

`kubectl create -f .`: to create resources from all the yamls available in current directory

`kubectl create -f [any-url]`: to create resources from the yaml in given url

* Create command will only be creating resources if the resources are not present in the kubernetes cluster. It is not idempotent.

* Apply command is used to create or update the resources. It is idempotent.

```
kubectl get [kind]                      to get all resources of specific kind   
kubectl get [kind] [resource-name]      to get the specific resource
kubectl get [kind] --show-labels        to show labels
kubectl get [kind] -l app=my-app        to query resources
kubectl get [kind] -o yaml              to format the output in yaml

kubectl describe [kind] [resource-name]               to describe the resource
kubectl delete [kind] [resource-name]                 to delete the given resource
kubectl logs [kind]/[resource-name]                   to check the log
kubectl exec -it [kind]/[resource-name] -- bash       to access the container of the pod
kubectl port-forward [kind]/[resource-name] 8080:80   to access our application APIs from our host for debugging
```

* rollout is used to bring new versions of pods via deployment. This also inclused roll back.

`kubectl rollout history deploy/deploy-order-service`: to check the history of roll outs

```
kubectl rollout undo deploy/deploy-order-service - to rollback to immediate previous revision
kubectl rollout undo deploy/deploy-order-service --to-revision=1 - to rollback to revision number 1 available in rollout history
kubectl rollout history deploy/deploy-order-service --revision=5 - to get detail rollout history for revision 5 (should be available in history)
```

## Deployment Strategy

`recreate`: terminate the old pods and then create the new pods

`rolling update`: gradually roll out the changes. We can have mix of old and new pods temporarily defined in number of percentage

`maxSurge`: maximum number of additional pods that can  be created as part of roll out of newer version and then gradually older version will get removed.

`maxUnavailable`: max number of pods that can be removed
  
`Service (svc)`: it is a logical (network layer) abstraction for a set of pods and expose them via a single stable endpoint ie.. stable IP address, DNS name.

* Service is implemented by inbuilt k8s proxy called kube-proxy. It is  a simple proxy which mantains the records of IPs of pods mapped to the service. It delagtes call at random and does not do round robin LB. 

## Service types

`ClusterIP`: default for communication within k8s cluster. Cannot be accessed outside the cluster. AWS/GCP - private subnet communication. Mostly used.

`NodePort`: used for testing. can be accessed from outside via K8s master/nodes via specific port. used for testing purpose.

`LoadBalancer`: Is used in cloud providers to recieve traffic from outside and distribute evenly.

`NameSpace`: logical separation of environments in k8s cluster.

`kubectl get namespace / ns`: if no namespace is defined, everything is created in default namespace

```
kubectl get ns 
NAME                 STATUS   AGE
default              Active   144m
kube-node-lease      Active   144m  - all kube- related ns is created by k8s
kube-public          Active   144m
kube-system          Active   144m - this ns contains important services running as pods
local-path-storage   Active   143m - created by kind
```

`kubectl create ns dev`: to create an namespace named dev

`kubectl get pod -n dev`: to get all pods info running in namespace named dev

`kubectl apply -f . -n dev`: to create all resources mentioned yamls in current directory in dev namespace

`kubectl delete ns dev`: to delete dev namespace

* We can also use deployment files to define the namespace as part of metadata instead of using it in command like below:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
  namespace: dev
```

* Probes are tools to measure the health of the pod ie.. app running in the pod by checking
1. has it started?
2. is it alive?
3. is it ready to serve requests?

* An alive pod may not be ready to serve requests maybe beacause of slow intialization or dependent downstream service down.
Kublet runs the probes. configuring the probes is optional

## Probes types

1. `startupProbe`: checks if app inside pod has started. if check fails restart. It starts as soon as container starts. If check passes, 
startupProbe stops.

2. `livenessProbe`: checks if app is still alive. if check fails restart. It starts once the startupProbe completes. It is executed throughout the pod lifecycle.

3. `readinessProbe`: checks if app is ready to take requests from service. if check fails remove from service. It starts once the startupProbe completes. It is executed through out pod lifecycle.

`Probe options` (part of container spec)

`exec`: to execute any command to check. like cat /dir/subdir/somefile Ex. kafka server
`httpGet`: to invoke any http endpoint. like health endpoint. Ex - spring boot app
`tcpSocket`: to check if app started listening on specifc port. Ex - DBs

### Probe properties 
```  
initialDelaySeconds - 0
periodSeconds       - 10
timeoutSeconds      - 1
successThreshold    - 1
failureThreshold    - 3
```

## Config Map and Secrets
to keep the configuration data separately from the application

`Config Map`: non sensitive data like application.properties. Properties can be as key value pair or as files. max size incase of files is 1mb

`Secret`: sensitive data like credentials. Value is base64 encoded. Use case like - ssh keys, credentials, service accounts etc
In real world scenario, k8s admin can set up role based access control (RBAC) to not allow not normal users to see secret. In cloud, secret manager is used.

* configMaps created are stored in etcd and when pods needs these config maps as env or file, its injected wherever its running.
using config map and secret approach we can store many kind of files like private files, certs, sql scripts and get them injected in container.	

`kubectl get cm`: to check on config maps

`kubectl get cm app-properties -o yaml`: to get the congif map named app-properties to be output as a yaml file

`kubectl describe cm app.properites`: to check on cm named app.properties

* We can use configmaps as values or inject them directly in pod and also import files as config map

creating secret via cli:

```
kubectl create secret generic my-secret --from-literal=username=Rupesh --from-literal=password=admin
```

this will create a secret will values encoded in base64
for base64 value of Rupesh, run in linux: echo -n Rupesh | base64

```
cat secret-file.yaml | base64
c3dpc3MuYmFuay5hY2NvdW50OjAwNw0Kc2Vuc2l2aXR5OmNvbmZpZGVudGlhbA0KdXNlcm5hbWU6UnVwZXNoDQpwYXNzd29yZDphZG1pbg==
```

## Persistent volumes
1. must for a stateful application for having durable storage.
2. PV or persitent volumes are storage abstractions/ volume plugins
3. They provide storage similar to node in cluster which provides CPU/Memory

`Storage class`: Type of storage characterised with IOPS, type like SSD, HDD, speed

`Persistent volume claim (PVC)`: request to create PV. Resources which links PV and POD. Ex - Request to create 5GB of GCP PD SSD.

`Persistent Volumes`: actual storage created for a specific storage class.  Ex - 5gb of GCP PD SSD

* PVs can be created into modes -
1. `Static Provisioning`: to attach an available PD to the pod

2. `Dynamic Provsioning`: on demand PVC to create and get a PD attached to a pod

`Access mode`: to tell k8s that in which mode storage should be attached the nodes

1. `ReadWriteOnce` (can read write from only a node)

2. `ReadWriteOncePod` (can read write from only a pod in a node)

3. `ReadOnlyMany` (can read from many nodes)

4. `ReadWriteMany` (can read and write from many nodes)

## Stateful set
1. Same as deployment but for stateful workload like db. Each pod will have unique/ stable hostname and will not be defined arbitrary.
2. It is for any application that has workload with stick identity.
3. for ex - 3 replicas will have pod names with pod-name-1, pod-name-2, pod-name-3
4. Scale out as 0, 1 , 2, 3 and scales in as 3, 2, 1, 0

`Headless service`: used for stateful apps. service will not have any IP. DNS entries would be created for pod-name.svc-name

`curl my-pod-0.my-headless-service`: for using a particular pod

* For stateful services, we can have a stateful PV per stateful pod using PVCs. In such cases even if pod is deleted or down, PV is still available and any new instance of pod getting up with previous stateful pod will get this pv attached thus maintaing the state.

`Horizontal Pod AutoScaler`: for cpu and memory(RAM)
```
range 
min - request
max - limit
```

`Memory(RAM)`:   1M, 50M , 1G or 1Mi, 50Mi, 1Gi ie.. 1mb of RAM, 50mb, 1gb of ram

`CPU`:  1, 100m, 500m ie.. 1 CPU, 0.1 CPU or 1/10th of a CPU or half of CPU ie.. for each cycle of 100ms 1 CPU can be consumed by the container.

* In case of exceeding limits, for memory - kublet will kill the container and restart whereas for CPU - Container will not be killed but throttled.

* top command is used to check on cpu and memory usage
* metrics server can be installed in k8s cluster to check on metrics by command: `kubectl top nodes or pods`

* horizontal pod autoscaler is a deployment monitor which senses and autscales and scales down the pods horizontally and will only work if we have metric server installed.

`ab -n 20000 -c 5 http://nginx-service`: upon running a load test using apache beam for 20K requests with 5 concurrent users cloud show graceful auto scale of pods and down scale with HPA. Probes are neccersary for applying HPA otherwise if pods are not ready, requests delegates to newly created pod will fail.

finding suitable resource configuration for pod is neccersary and purely depends on application design and usage.
1 GB RAM and 500m CPU can be a decent standard for average apps to apply and then load test to measure as per our requirement and increase and decrease accordingly.

If we run short of nodes while HPA wants to autoscale the pods, such pods will go in pending state and will not be created. In cloud, to tackle this we use `cluster auto scaler` which senses the pending stage pods because of nodes contraints and attaches extra nodes based on our provided configurations for cluster autoscaling.

## Ingress

Smart router/Proxy to bring traffic into the cluster inside a namespace
contains a set of routing rules
we need Ingress controller(implements the rules) to manage ingress resources (contains routing rules)
K8s provides nginx based ingress and AWS and GCP ingress.

In case of multiple namespace, ingress only using path based routing will fail since ingress controller which applies the routes does not sits in any local namespace and thus will raise error for duplicates port being used for same route, so in such cases we apply hostname + path based routing.
we can add the host name to the file with admin privilage at path: 
C:\Windows\System32\drivers\etc\hosts in window or /etc/hosts in linux
127.0.0.1 dev.my-app.com
127.0.0.1 test.my-app.com