

# Kubernetes Notes from course Certified Kubernetes Administrator (CKA) KodeKloud

## Commands 
kubectl get namespaces

List all pods from default namespace
```
kubectl get pods
````

Get all pods from a specific namespace

```
kubectl get pods --namespace=kube-system
kubectl get pods -n kube-system
```

Get all created objects:

```
$ kubectl get all
```

and the output:

```
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

## Pods yml file definition

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

```
kubectl create -f pod-definition.yml
```

or 

```
kubectl apply -f pod-definition.yml
```

Delete a running pod:

```
kubectl delete pod myapp-pod
```

Get information about a running pod:

```
kubectl apply -f pod-definition.yml
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

```
kubectl get nodes
```

## Replication Controller

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

```
kubectl apply -f replication_controller.yaml
kubectl get replicationcontroller
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


```
kubectl apply -f replicaset-definition.yaml
kubectl get replicaset
```

Replicaset can monitor and manage pods not created with the replica set. RS knows what pods to monitor using **selectors** and **labels**.

```
$ kubectl get replicaset
$ kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
myapp-rs   3         3         3       11s
````

Describe/see the configuration of a replicaset:

```
kubectl describe replicaset new-replica-set
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
vagrant@vagrant:~$ kucectl scale --replicas=3 replicaset myapp-rs
kucectl: command not found
vagrant@vagrant:~$ kubectl scale --replicas=3 replicaset myapp-rs
replicaset.apps/myapp-rs scaled
vagrant@vagrant:~$ kubectl get replicaset
NAME       DESIRED   CURRENT   READY   AGE
myapp-rs   3         3         3       32m
vagrant@vagrant:~$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
myapp-pod        1/1     Running   0          11h
myapp-rs-nqw4g   1/1     Running   0          33m
myapp-rs-vqnd8   1/1     Running   0          3m16s
vagrant@vagrant:~$ kubectl describe replicaset myapp-rs
Name:         myapp-rs
Namespace:    default
Selector:     type=front-end
Labels:       app=myapp
              type=front-end
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
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
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  59m   replicaset-controller  Created pod: myapp-rs-skdpv
  Normal  SuccessfulCreate  59m   replicaset-controller  Created pod: myapp-rs-422rf
  Normal  SuccessfulCreate  59m   replicaset-controller  Created pod: myapp-rs-7jv84
  Normal  SuccessfulDelete  58m   replicaset-controller  Deleted pod: myapp-rs-7jv84
  Normal  SuccessfulDelete  58m   replicaset-controller  Deleted pod: myapp-rs-f5bgm
  Normal  SuccessfulDelete  58m   replicaset-controller  Deleted pod: myapp-rs-422rf
  Normal  SuccessfulDelete  58m   replicaset-controller  Deleted pod: myapp-rs-skdpv
  Normal  SuccessfulCreate  31m   replicaset-controller  Created pod: myapp-rs-vqnd8
  Normal  SuccessfulCreate  31m   replicaset-controller  Created pod: myapp-rs-mmpzb
  Normal  SuccessfulCreate  31m   replicaset-controller  Created pod: myapp-rs-4d9hb
  Normal  SuccessfulCreate  31m   replicaset-controller  Created pod: myapp-rs-w7zwd
  Normal  SuccessfulDelete  28m   replicaset-controller  Deleted pod: myapp-rs-mmpzb
  Normal  SuccessfulDelete  28m   replicaset-controller  Deleted pod: myapp-rs-4d9hb
  Normal  SuccessfulDelete  28m   replicaset-controller  Deleted pod: myapp-rs-w7zwd
  ```

Scaling replicasets:

1.- Update replicas in replicaset definition and execute the kubectl replace command:

```
kubectl replace -f replicaset-definition.yml
```

2.- Use kubectl scale pointing to the definition file:

```
kubectl scale --replicas=6 -f replicaset-definition.yml
```

3.- Use kubectl scale with the replicaset name:
```
kubectl scale --replicas=3 replicaset myapp-rs
``` 

4.- Edit a replicaset or an object in memory. Changes are not applied to pods until pods are deleted and recreated but it works automatically with the replicas number.

```
kubectl edit rs new-replica-set
```


### Replicaset troubleshooting

* Error: **selector** does not match template **labels**

Solution: The selector in definition and the labels in the containers template must be the same.

* Error: no matches for kind "ReplicaSet" in version "v1"

Solution: the api version is not correct for this kind of resource. Replicasets are implemented in apiVersion apps/v1

* Error:  replicaset in version "v1" cannot be handled as a ReplicaSet: no kind "replicaset" is registered for version "apps/v1" in scheme

Solution: the resource type replicaset is mispelled and doesn't exist in current api version.


## Deployments

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

```
$ kubectl apply -f nginx-deployment.yaml
$ kubectl get deployments
```

## Generating definition files from scratch

Create an nginx Pod with run and show definition:

```
kubectl run jenkins --image=jenkins --dry-run=client -o yaml
```

Create a deployment definition file, then modify it and apply

```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
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


  httpd-frontend; Replicas: 3; Image: httpd:2.4-alpine

## Namespaces


