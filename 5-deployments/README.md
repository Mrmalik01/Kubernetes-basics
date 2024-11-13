## Deployments

Previously we were creating the pods directly, which is not an ideal approach in the production or even in the development environment. Thats why we need deployments.

- Represents multiple replicas of a Pod (seems like an AMI template for EC2 deployments)
- Describe a desired state that Kubernetes needs to achieve (similar to the auto-scaling groups, we can also define the number of pods)
- Deployment Controller master component converges actual state to the desired state

Deployments are just base templates that are used to create Kubernetes Pods.


**5.2-data_tier.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: data-tier-svc
  labels:
    app: deployment-learning
    env: dev
spec:
  ports:
  - port: 6379
    protocol: TCP
    name: redis
  selector:
    tier: data-tier
    env: dev
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-tier-deployment
  labels:
    tier: data-tier
    env: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: data
      env: dev
  template:
    metadata:
      labels:
        app: microservices
        tier: data
    spec: # Pod spec
      containers:
      - name: redis
        image: redis:latest
        imagePullPolicy: IfNotPresent
        ports:
          - containerPort: 6379
```
**Output**
```
❯ kubectl describe pod data-tier-deployment-84487494fd-ff7s7 -n dp-5                                                                                                                                                                                                                                         ─╯
Name:             data-tier-deployment-84487494fd-ff7s7
Namespace:        dp-5
Priority:         0
Service Account:  default
Node:             minikube/10.0.2.15
Start Time:       Mon, 11 Nov 2024 16:50:37 +0000
Labels:           env=dev
                  pod-template-hash=84487494fd
                  tier=data-tier
Annotations:      <none>
Status:           Running
IP:               10.244.0.4
IPs:
  IP:           10.244.0.4
Controlled By:  ReplicaSet/data-tier-deployment-84487494fd
Containers:
  redis:
    Container ID:   docker://75b217301f757df7109483b1bc09b6e186dff924ff20c604210aedbe8508189d
    Image:          redis:latest
    Image ID:       docker-pullable://redis@sha256:a06cea905344470eb49c972f3d030e22f28f632c1b4f43bbe4a26a4329dd6be5
    Port:           6379/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 11 Nov 2024 16:50:41 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4mt2t (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-4mt2t:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  24m   default-scheduler  Successfully assigned dp-5/data-tier-deployment-84487494fd-ff7s7 to minikube
  Normal  Pulling    24m   kubelet            Pulling image "redis:latest"
  Normal  Pulled     24m   kubelet            Successfully pulled image "redis:latest" in 3.862s (3.862s including waiting). Image size: 139514366 bytes.
  Normal  Created    24m   kubelet            Created container redis
  Normal  Started    24m   kubelet            Started container redis
```

This line shows that this pod is managed by a deployment
**Controlled By:  ReplicaSet/data-tier-deployment-84487494fd**


**Deployment Description**
```
❯ kubectl describe deployments data-tier-deployment -n dp-5                                                                                                                                                                                                                                                                ─╯
Name:                   data-tier-deployment
Namespace:              dp-5
CreationTimestamp:      Mon, 11 Nov 2024 16:50:37 +0000
Labels:                 env=dev
                        tier=data-tier
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               env=dev,tier=data-tier
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  env=dev
           tier=data-tier
  Containers:
   redis:
    Image:         redis:latest
    Port:          6379/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   data-tier-deployment-84487494fd (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  35m   deployment-controller  Scaled up replica set data-tier-deployment-84487494fd to 1
```

We can also use the scale command to scale each deployments, for example to scale the support tier to 5 replicas  
`kubectl scale -n dp-5 deployments support-tier --replicas=5`



### API Version

In Kubernetes, the apiVersion field specifies which API group and version a resource belongs to. Let’s break down the apps/v1 part and compare it to just v1.

#### apps/v1 vs v1
v1 is the core API version, used for fundamental resources like Pods, Services, ConfigMaps, and PersistentVolumes. It includes basic resources that are essential for Kubernetes operations.

apps/v1, on the other hand, is part of the "apps" API group and is used for managing higher-level application-related resources like Deployments, DaemonSets, StatefulSets, and ReplicaSets. This group is specifically designed for resources that manage applications and workloads.

#### Why Use apps/v1 for Deployments?
Deployments are considered an "application" management feature in Kubernetes. They allow you to manage things like rolling updates, scaling, and workload states. So, to keep things organized and logical, Kubernetes separated these resources into the apps API group. The apps/v1 group handles more complex application lifecycle needs, whereas v1 is for more basic or core resources.

#### Example 
v1 is like the "core" department in a company that manages fundamental functions, such as payroll or facilities.
apps/v1 is like a specialized department focused on applications, handling tasks like deployments, updates, and scaling.
In short, we use apps/v1 for Deployments because they’re part of the applications API, which is designed specifically for managing and scaling app-related resources. This separation helps Kubernetes organize resources and allows each API version to evolve independently as needed.

#### Explain the output
the kubectl describe pod output doesn’t directly mention v1 or apps/v1. This is because, at the Pod level, the API version or API group (like v1 or apps/v1) is abstracted away. Kubernetes shows details about the resource itself, not the API group that defined it.

Here’s why:

**Deployment vs. Pod Definitions:**

When you create a Deployment, you use the apps/v1 API, which manages the Deployment and indirectly creates a ReplicaSet and Pods.
The Deployment uses apps/v1 because it’s specifically for managing application lifecycle (like scaling and updating), but it produces Pods, which are core resources defined under the v1 API.

**Pods are Core Resources:**

The Pod itself is part of the core Kubernetes resources (v1), so its API version isn’t displayed here. Once created, the Pod operates independently and doesn’t need to know about the Deployment or ReplicaSet API version.
You can see the Controlled By: ReplicaSet field in the describe output, which points to the ReplicaSet that owns it (and indirectly to the Deployment). This hints that this Pod was created by a Deployment (apps/v1), but it doesn’t show the API version explicitly.

**Level of Abstraction:**

Kubernetes abstracts API details after resources are created, especially at the Pod level. API groups like v1 or apps/v1 are essential during resource creation or when writing YAML specifications but not during runtime operations.
In summary, once the Deployment creates a Pod, the Pod doesn't inherently retain or display the apps/v1 association. Kubernetes organizes resources by functionality and ownership (like showing that a ReplicaSet controls the Pod) but doesn't explicitly show API version details in runtime descriptions.

### Kubernetes Commands

**To deploy multiple deployments together**
`kubectl create deployments -n <namespace> -f <.yaml> -f <.yaml>`

**To Get all Deployments**

`kubectl get -n <namespace> deployments <deployment_name>`

**To Update the Replica Counter**

`kubectl scale -n <namespace> deployments <deployment_name> --replicas=5`

**To Delete All Deployments**

`kubectl delete -n <namespace> deployments -all`

❯ kubectl create deployments -n dp-5 -f 5-deployments/5.2-data_tier.yaml -f 5-deployments/5.3-app_tier.yaml -f 5-deployments/5.4-support-tier.yaml─╯