

# Kubernetes Notes from course Certified Kubernetes Administrator (CKA) by Mumshad Mannambeth from KodeKloud

## COMMANDS 
List all namespaces in the cluster

```console
me@k8s:~$ kubectl get namespaces
me@k8s:~$ kubectl get ns
me@k8s:~$ kubectl get ns --no-headers
````

List all pods from default namespace
```console
me@k8s:~$ kubectl get pods
````

Get all pods from a specific namespace

```console
me@k8s:~$ kubectl get pods --namespace=kube-system
me@k8s:~$ kubectl get pods -n kube-system
```

Get all created objects:

```console
me@k8s:~$ kubectl get all
```

and the output:

```console
NAME                 READY   STATUS    RESTARTS   AGE
pod/myapp-pod        1/1     Running   0          17h
pod/myapp-rs-75php   1/1     Running   0          3h41m
pod/myapp-rs-79tnz   1/1     Running   0          3h41m
pod/myapp-rs-7j276   1/1     Running   0          3h41m
pod/myapp-rs-8klfv   1/1     Running   0          3h41m
pod/myapp-rs-9hjph   1/1     Running   0          3h41m
pod/myapp-rs-9hvd7   1/1     Running   0          3h41m
pod/myapp-rs-nqw4g   1/1     Running   0          6h29m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   28h

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs   8         8         8       6h29m
```

## PODS YML FILE DEFINITION

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
  
```

apiVersion possible values:

Kind  | version 
------|--------
POD | v1
Service | v1
ReplicaSet | apps/v1
Deployment | apps/v1

```console
me@k8s:~$ kubectl create -f pod-definition.yml
```

or 

```console
me@k8s:~$ kubectl apply -f pod-definition.yml
```

Delete a running pod:

```console
me@k8s:~$ kubectl delete pod myapp-pod
```

Get information about a running pod:

```console
me@k8s:~$ kubectl apply -f pod-definition.yml
```

The above command produces a output like the following:

```yaml
Name:         myapp-pod
Namespace:    default
Priority:     0
Node:         minikube/172.17.0.3
Start Time:   Sat, 01 Aug 2020 20:35:33 +0000
Labels:       app=myapp
              type=front-end
Annotations:  Status:  Running
IP:           172.18.0.3
IPs:
  IP:  172.18.0.3
Containers:
  nginx-container:
    Container ID:   docker://a777853943104c0e944dc8318d81f5d7f3da09db0ff17e8c802a085eaa695c55
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:0e188877aa60537d1a1c6484b8c3929cfe09988145327ee47e8e91ddf6f76f5c
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 01 Aug 2020 20:35:36 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-mvsfc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-mvsfc:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-mvsfc
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  13s   default-scheduler  Successfully assigned default/myapp-pod to minikube
  Normal  Pulling    12s   kubelet, minikube  Pulling image "nginx"
  Normal  Pulled     10s   kubelet, minikube  Successfully pulled image "nginx"
  Normal  Created    10s   kubelet, minikube  Created container nginx-container
```

List the kubernetes nodes:

```console
me@k8s:~$ kubectl get nodes
```

## REPLICATION CONTROLLER

* Replication Controller: older technology to set up replication, replaced by ReplicaSet.
* Replica Set: Currently used, uses selectors. 

replication_controller.yaml example:

```yaml
apiVersion: v1
kind: ReplicationController
metadata: 
  name: myapp-rc
  labels: 
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
```

```console
me@k8s:~$ kubectl apply -f replication_controller.yaml
me@k8s:~$ kubectl get replicationcontroller
```

Replicaset

replicaset-definition.yaml example:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels: 
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```


```console
me@k8s:~$ kubectl apply -f replicaset-definition.yaml
me@k8s:~$ kubectl get replicaset
```

Replicaset can monitor and manage pods not created with the replica set. RS knows what pods to monitor using **selectors** and **labels**.

```console
me@k8s:~$ kubectl get replicaset
me@k8s:~$ kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
myapp-rs   3         3         3       11s
````

Describe/see the configuration of a replicaset:

```console
me@k8s:~$ kubectl describe replicaset new-replica-set
```

```yaml
Name:         myapp-rs
Namespace:    default
Selector:     type=front-end
Labels:       app=myapp
              type=front-end
Annotations:  <none>
Replicas:     6 current / 6 desired
Pods Status:  6 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=myapp
           type=front-end
  Containers:
   nginx-container:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  32m    replicaset-controller  Created pod: myapp-rs-nqw4g
  Normal  SuccessfulCreate  32m    replicaset-controller  Created pod: myapp-rs-f5bgm
  Normal  SuccessfulCreate  30m    replicaset-controller  Created pod: myapp-rs-skdpv
  Normal  SuccessfulCreate  30m    replicaset-controller  Created pod: myapp-rs-422rf
  Normal  SuccessfulCreate  30m    replicaset-controller  Created pod: myapp-rs-7jv84
  Normal  SuccessfulDelete  29m    replicaset-controller  Deleted pod: myapp-rs-f5bgm
  Normal  SuccessfulDelete  29m    replicaset-controller  Deleted pod: myapp-rs-7jv84
  Normal  SuccessfulDelete  29m    replicaset-controller  Deleted pod: myapp-rs-422rf
  Normal  SuccessfulDelete  29m    replicaset-controller  Deleted pod: myapp-rs-skdpv
  Normal  SuccessfulCreate  2m31s  replicaset-controller  Created pod: myapp-rs-vqnd8
  Normal  SuccessfulCreate  2m31s  replicaset-controller  Created pod: myapp-rs-mmpzb
  Normal  SuccessfulCreate  2m31s  replicaset-controller  Created pod: myapp-rs-4d9hb
  Normal  SuccessfulCreate  2m31s  replicaset-controller  Created pod: myapp-rs-w7zwd

  ```

Scaling replicasets:

1.- Update replicas in replicaset definition and execute the kubectl replace command:

```console
me@k8s:~$ kubectl replace -f replicaset-definition.yml
```

2.- Use kubectl scale pointing to the definition file:

```console
me@k8s:~$ kubectl scale --replicas=6 -f replicaset-definition.yml
```

3.- Use kubectl scale with the replicaset name:
```console
me@k8s:~$ kubectl scale --replicas=3 replicaset myapp-rs
``` 

4.- Edit a replicaset or an object in memory. Changes are not applied to pods until pods are deleted and recreated but it works automatically with the replicas number.

```console
me@k8s:~$ kubectl edit rs new-replica-set
```


### REPLICASET TROUBLESHOOTING

* Error: **selector** does not match template **labels**

Solution: The selector in definition and the labels in the containers template must be the same.

* Error: no matches for kind "ReplicaSet" in version "v1"

Solution: the api version is not correct for this kind of resource. Replicasets are implemented in apiVersion apps/v1

* Error:  replicaset in version "v1" cannot be handled as a ReplicaSet: no kind "replicaset" is registered for version "apps/v1" in scheme

Solution: the resource type replicaset is mispelled and doesn't exist in current api version.


## DEPLOYMENTS

A Deployment provides declarative updates for Pods ReplicaSets.

the basic definition is the same as a replicaSet with a different kind type: Deployment and that it creates a deployment object.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```console
me@k8s:~$ kubectl apply -f nginx-deployment.yaml
me@k8s:~$ kubectl get deployments
me@k8s:~$ kubectl edit  deployment mydployment

```

## GENERATING DEFINITION FILES FROM SCRATCH

Create an nginx Pod with run and show definition (dry-run):

```console
me@k8s:~$ kubectl run jenkins --image=jenkins --dry-run=client -o yaml
```

Create a pod with labels

```console
me@k8s:~$ kubectl run redis --image=redis:alpine -l tier=db
```

Create a pod and expose it on port 8080

```console
me@k8s:~$  kubectl run custom-nginx --image=nginx --port=8080
```

Create a pod and a service in one step:

```console
me@k8s:~$  kubectl run httpd --image=httpd:alpine --port=80 --expose
```

Create a deployment definition file, then modify it and apply

```console
me@k8s:~$ kubectl create deployment  blue --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Other resources that can be created this way:

  * clusterrole
  * clusterrolebinding  
  * configmap           
  * cronjob             
  * deployment          
  * job                 
  * namespace           
  * poddisruptionbudget 
  * priorityclass
  * quota               
  * role                
  * rolebinding
  * secret              
  * service
  * serviceaccount      

## NAMESPACES

Namespaces are used for resource isolation.

Each namespace have their own policies and Resource limits.

Resource in the same namespace can reach eachother by their short name, but to reach others in another namespace the full dns name must be used. p.ex:


Kubernetes creates several namespaces automatically:

- kube-system
- default
- kube-public



Create a namespace:
```console
me@k8s:~$ kubectl create namespace dev
namespace/dev created
```

or with a definition file:

```yaml
apiVersion: v1
kind: Namespace

metadata: 
  name: dev
```


```console
me@k8s:~$ kubectl apply -f test-namespace.yml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
namespace/dev configured
```

Create a pod in the namespace:

```console
me@k8s:~$ kubectl apply -f pod-definition.yml -n dev
pod/myapp-pod created
```

List pods in a certain namespace:

```console
me@k8s:~$ kubectl get pods --namespace=dev
me@k8s:~$ kubectl get pods -n dev
```

Namespaces in Pods can also be defined in the definition yaml file:

```yaml
apiVersion: v1
kind: Pod

metadata: 
  name: myapp
  namespace: dev
  ...
```

If namespace is not specified the default one is used. But context can be switched in order to use a different namespace by default:

```console
me@k8s:~$ kubectl config set-context $(kubectl config current-context) --namespace=dev
```

List pods in all namespaces: (-A or --all-namespaces)

```yaml
me@k8s:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
dev           myapp-pod                          1/1     Running   0          23m
kube-system   coredns-66bff467f8-n8rlt           1/1     Running   0          30h
kube-system   etcd-minikube                      1/1     Running   0          30h
kube-system   kube-apiserver-minikube            1/1     Running   0          30h
kube-system   kube-controller-manager-minikube   1/1     Running   0          30h
kube-system   kube-proxy-b7fqj                   1/1     Running   0          30h
kube-system   kube-scheduler-minikube            1/1     Running   0          30h
kube-system   storage-provisioner                1/1     Running   0          30h
me@k8s:~$
```

## RESOURCE QUOTA

Limit resources in a namespace

```yaml
apiVerion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 2Gi
    limits.cpu: "10"
    limits.memory: 4Gi
```

```console
me@k8s:~$ kubectl create -f compute.quota.yml
```

## SERVICES

[go to kubernetes docs](https://kubernetes.io/docs/concepts/services-networking/service/)

Expose applications running on a set of Pods as a network service.

Defines a logical set of Pods an d a policy by which to access them and targeted usually using a selector (but not mandatory).

Service discovery using Kubernetes API Server. For non-nativa applications a network port or a LB beetween the application and the load backend can be used.

ServiceTypes:
- **NodePort:** Exposes the Service on each Node's IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. You'll be able to contact the NodePort Service, from outside the cluster, by requesting <NodeIP>:<NodePort>
- **ClusterIp:** Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default ServiceType
- **LoadBalancer:** Exposes the Service externally using a cloud provider's load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.

## NODEPORT

- port: the port in the clusterIp. This is the only mandatory port.
- targetPort: port in the Pod. Optional, assigned by K8s.
- NodePort: port in the node. Port valid range are: 3000-32767. Also optional. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service # must be a valid DNS label name
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

## SERVICES CLUSTER IP

Internal ips inside the cluster used by other Pods inside the cluster to access our service. Used in application layers/tiers.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend # must be a valid DNS label name
spec:
  type: ClusterIP
  ports:
    - targetPort: 80 
      port: 80
  selector:
    app: myapp
    type: front-end
```

get services

```console
me@k8s:~$ kubectl get svc -n test
me@k8s:~$ kubectl get services -n test
```

Describe a service

```console
master $ kubectl describe svc 
defaultName:       kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         172.17.0.74:6443
Session Affinity:  None
Events:            <none>
```

kubectl expose command:

Exposes a resource as a new Kuberbetes service. Possible resources (pod (po), service (svc), replicationcontroller (rc), deployment (deploy), replicaset (rs)).

Looks up a resource by name ans uses the selextor for that resource as the selector for a new service on the specified port.

- Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379. Automtically use the pod's labels as selectors.

```console
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

- Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes. Can't specify the port, so yml file must be edited:

```console
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```

# SECTION 3: SCHEDULING

## MANUAL SCHEDULING

Schedulers:

Every Pod definition has a field named nodeName that the scheduler fills. If no scheduler is available the pods get in a stastus "pending". The node can be assigned statically by using this field. If the Pod is already  created a Pod-bind-definition.yaml file must be created. (having a Pod named "nginx")

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

Use flag **-o wide** to display the node where is the Pod deployed:


```console
masster $ kubectl get pods -o  wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          75s   10.244.0.4   master   <none>           <none>
master $
```

## LABELS AND SELECTORS

[go to kubernetes docs](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

Labels are properties attached to items, selectors help to filter items based on those labels. Labels are key/value pairs. 

```yaml
...
apiVersion: v1
kind: Pod
metadata:
  labels: 
    app: App1
    env: production
    tier: frontend
spec:
  ...
```

Use the api to get pods filtered by selector:

```console
me@k8s:~$ kubectl get pods --selector app=App1
```

also with the -l sortcut:

```console
me@k8s:~$ kubectl get pods -l app=App1
```

Kubernetes uses selectors and labels to link objets, as pods with replicasets and services.

## Annotations:

Annotations are used to record other aspects or values of the object.

```yaml
...
apiVersion: apps/v1
kind: ReplicaSet
name: simple-webapp
metadata:
  labels: 
    app: App1
    env: production
    tier: frontend
  annotations:
    buildversion: 1.34
spec:
  ...
```

Use multiple labels as selectors to filter objects:

```console
me@k8s:~$ kubectl get pods --selector env=prod,bu=finance,tier=frontend
```

## TAINTS AND TOLERATIONS

Taints: avoid a pod can land on a certain node. Taints are meant to restrict nodes from accepting certain nodes.

Taint a node and with a tag and only the pods with a toleration equal to this particular taint can be placed in this node.

```
kubectl taint nodes node-name key=value:taint-effect
```

taint-effects:

- NoSchedule
- PreferNoSchedule
- NoSchedule

```console
me@k8s:~$ kubectl taint nodes nide1 app=blue:NoSchedule
```

Toleration are placed inside the spec section of a Pod definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: ningx

  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```





Kubernetes by default taints master node to avoid accpting Pods

```console
me@k8s:~$ kubectl describe node kubemaster | grep Taint
```

Taint a node: 


```console
me@k8s:~$ kubectl taint nodes node01 spray=mortein:NoSchedule
```

Remove a taint:

```console
master $ kubectl taint nodes master  node-role.kubernetes.io/master:NoSchedule-
node/master untainted
master $
```
## NODE SELECTORS

Assign a pod to a specific node by using nodeSelector property:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: ningx
  nodeSelector:
    size: Large # this is a label asigned to the node
```

Before the node must be labeled:

```console
kubectl label nodes <node-name> <label-key>=<label-value>
```
 
show the node labels

```console
me@k8s:~$ kubectl get nodes --show-labels
```

Advanced filterings cannot be applied.

## NODE AFFINITY

[Assign Pods to Nodes using Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)

Ensure that pods are hosted in particular nodes. Multiple selector terms and match expressions can be used.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: ningx

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: 
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In # NotIn/Exists (checks label, not value)
            values:
            - Large
```

Node affinity types available:
- requiredDuringSchedulingIgnoredDuringExecution 

When the pod is created for the first time.

- preferredDuringSchedulingIgnoredDuringExecution


## RESOURCE REQUIREMENTS AND LILMITS

If kubernetes cannot schedule a pod into any node because resources required are not available in any node a error like the following should appear: 

```
No nodes are available that match all of the following predicates:: Insufficeint cpu
```

Default resource requriment for a pod/container are:
* cpu: 0.5 units == 500m
* mem: 256Mi

is more is needed the spec -> containers section can be configured:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: ningx
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1 # in increments of 0.1
```

By default k8s limits the use of cpu and memory on container to 1 Cpu and 512 Mi if this limit is not overrided. Configure new limits:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: ningx
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1 # in increments of 0.1
      limits:
        memory: "2Gi"
        cpu: 2
```

When a pod tries to consume more resources than the specified configured limits two things can happen depending on the resource:

* If the pod tries to use more CPU than the limit k8s will throttle the cpu usage and won't let the pod use more cpu than configured.

* In case of the memory, the pod can use more than the limit. If the pod used constantly more memory it will be terminated.

## DAEMONSETS

[Kubernetes DaemonSets Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

```console
me@k8s:~$ kubectl get ds --all-namespaces
me@k8s:~$ kubectl get daemonset --all-namespaces

me@k8s:~$ NAMESPACE     NAME                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-flannel-ds-amd64     2         2         2       2            2           <none>                   6m20s
kube-system   kube-flannel-ds-arm       0         0         0       0            0           <none>                   6m20s
kube-system   kube-flannel-ds-arm64     0         0         0       0            0           <none>                   6m20s
kube-system   kube-flannel-ds-ppc64le   0         0         0       0            0           <none>                   6m20s
kube-system   kube-flannel-ds-s390x     0         0         0       0            0           <none>                   6m20s
kube-system   kube-keepalived-vip       1         1         1       1            1           <none>                   6m19s
kube-system   kube-proxy                2         2         2       2            2           kubernetes.io/os=linux   6m22s


A DaemonSet yaml is equal to a ReplicaSet without the replicas parameter which is not needed because the replica number equal to the number of nodes.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  labels: 
    app: elasticsearch
spec:
  template:
    metadata:
      name: elasticsearch
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: kube-system
        image: k8s.gcr.io/fluentd-elasticsearch:1.20
  selector:
    matchLabels:
      type: elasticsearch
```