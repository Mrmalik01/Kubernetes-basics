## Pods

- One or more containers in a Pod
- Pod containers all share a container network
- One IP address per Pod

### Pod Declaration

- Container image
- Container ports
- Container restart policy
- Resource limits

### Manifest Files

- Declare desired properties
- Manifests can describe all kinds of resources
- The spec contains resource-specific properties

#### Manifests in Action

- `kubectl create` - sends manifest to Kubernetes API Server
    - API Server does the following for Pod manifests
        1. Select a node with sufficient resources
        2. Schedule Pod onto node
        3. Node pulls Pod’s container image
        4. Starts Pod’s container

#### Why use Manifests?

- Can check into source control
- Easy to share
- Easier to work with
- name must be unique in metadata
- Spec is where all the main data goes

Here is an example of a Manifest file that contains the configuration of an NginX server

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-server
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

**Metadata** helps in identifying the resources running in this pod. **Spec** is the specification of the declared kind, and it could change for different development environment, for example, the NGINX configuration could change and require different CPUs and Memory capacity for Prod and Dev workloads

```
Name:             nginx-server
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Sun, 10 Nov 2024 16:53:56 +0000
Labels:           app=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.0.3
IPs:
  IP:  10.244.0.3
Containers:
  nginx:
    Container ID:   docker://aa31fcd0fbf380ef5d5009e1c601303fc24152bf919e66207d701c7bb217d234
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:28402db69fec7c17e179ea87882667f1e054391138f77ffaf0c3eb388efc3ffb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 10 Nov 2024 16:54:02 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rtqtn (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-rtqtn:
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
  Normal  Scheduled  42s   default-scheduler  Successfully assigned default/nginx-server to minikube
  Normal  Pulling    41s   kubelet            Pulling image "nginx:latest"
  Normal  Pulled     37s   kubelet            Successfully pulled image "nginx:latest" in 4.597s (4.597s including waiting). Image size: 196880357 bytes.
  Normal  Created    36s   kubelet            Created container nginx
  Normal  Started    36s   kubelet            Started container nginx
```

**QoS Class:BestEffort** Shows that this is the best effort container and will be the first one to get chunked with a Pod with a specific RR comes.

Currently the above container will not be accessible outside the container network. This is because the above IP address (10.244.0.3) is only available within the container network. This IP has no meaning outside the kubernetes infrastructure. With this configuration, only another container within that Pod can interact with the NGINX server.

### Labels
- Labels are used for identifying specific resources in the namespace. For example, whether the pod is frontend or backend? or X application or Y application?
- Labels are used for making selection in Kubernetes.
- Similar to Tags in AWS.
- Multiple tags are allowed

### Resource Request

The pods that we've seen so far don't have any resource requests set which makes it easier to schedule them because the scheduler doesn't need to find nodes that have these requests for amounts of resources. It'll just throw them onto any node that isn't under pressure or starved of resources. However, these pods will be the first to be evicted if a node becomes under pressure, it needs to free up resources. That's called best effort quality of service which was displayed in the describe output. Best effort pods can also create resource contention with other pods on the same node and usually it's a good idea to set resource requests.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-server
  labels:
    app: nginx-web-server
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      resources:
        requests:
          memory: "64Mi" # 64Mi is equivalent to 64 Mebibytes (MiB) ~ 67.1 MB
          cpu: "250m" # 250m is equivalent to 0.25 CPU
        limits:
          memory: "128Mi"
          cpu: "500m"
```
[For more information on MiB ~ MB, please follow this link](https://digilent.com/blog/mib-vs-mb-whats-the-difference/?srsltid=AfmBOoqmRcJEDVQQnKYWRhJ6TQYaDMrF2kLrDXeLcbt1p1hr2VEVgWkk)

```
Name:             nginx-server
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Sun, 10 Nov 2024 17:10:08 +0000
Labels:           app=nginx-web-server
Annotations:      <none>
Status:           Running
IP:               10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  nginx:
    Container ID:   docker://195db6f6f17bb21ad175ea2332e38da177b309db3572e7d08cd85c430e97ce3f
    Image:          nginx:latest
    Image ID:       docker-pullable://nginx@sha256:28402db69fec7c17e179ea87882667f1e054391138f77ffaf0c3eb388efc3ffb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 10 Nov 2024 17:10:09 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        250m
      memory:     64Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-649b8 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-649b8:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  56s   default-scheduler  Successfully assigned default/nginx-server to minikube
  Normal  Pulling    55s   kubelet            Pulling image "nginx:latest"
  Normal  Pulled     55s   kubelet            Successfully pulled image "nginx:latest" in 867ms (867ms including waiting). Image size: 196880357 bytes.
  Normal  Created    55s   kubelet            Created container nginx
  Normal  Started    55s   kubelet            Started container nginx
```

Now we can see the resource request and limits under the Nginx container configuration output. It means the Pod requires a CPU with atleast 250m CPU and 64 Mi with the limit of 500m CPU and 128 Mi

**QoS Class:Burstable**
Overall, there are three options: Guaranteed, Burstable and BestEffort

[For a Pod to be given a QoS class of Guaranteed:](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)

- Every Container in the Pod must have a memory limit and a memory request.
- For every Container in the Pod, the memory limit must equal the memory request.
- Every Container in the Pod must have a CPU limit and a CPU request.
- For every Container in the Pod, the CPU limit must equal the CPU request.

A Pod is given a QoS class of Burstable if:

- The Pod does not meet the criteria for QoS class Guaranteed.
- At least one Container in the Pod has a memory or CPU request or limit.

### Guaranteed QoS Class
- Strict Resource Requests and Limits: A Pod is classified as Guaranteed when every container in it has the same CPU and memory requests and limits.
- Consistency in Resource Availability: By defining equal requests and limits, you ensure that the Pod always has a reserved amount of resources that it can reliably use. This setup ensures predictability in terms of performance and resilience under load.
- Top Priority in Resource Allocation: Pods with Guaranteed QoS have the highest priority when the Kubernetes scheduler allocates resources. During a node’s resource crunch, Guaranteed Pods are less likely to be evicted (killed) by Kubernetes.
- Use Case: Ideal for critical workloads that require stable and reliable performance, where resource requirements are predictable and predefined.

### Burstable QoS Class
- Flexible Resource Requests and Limits: A Pod is classified as Burstable if at least one container has a memory or CPU request or limit, but it does not meet the stricter criteria for the Guaranteed class.
- Potential for Resource Flexibility: Burstable Pods can potentially use more resources than initially requested if the node has excess capacity, making them more adaptable to varying loads.
- Moderate Priority in Resource Allocation: Burstable Pods have a lower priority compared to Guaranteed Pods. During high resource usage, they are more likely to be evicted before Guaranteed Pods if resources become constrained on the node.
- Use Case: Suitable for workloads that need flexibility and can tolerate some variability in performance. Burstable Pods are common for less critical applications that may benefit from extra resources when available but can also cope with potential throttling or eviction.

### Kubernetes Commands

**Kubectl Create**

`kubectl create -f <name_of_yaml_file>.yaml`  - this command creates a pod running in the Kubernetes environment. 

- -f option: refers to creating a manifest file from the yaml

**Kubectl Get Pods**

`kubectl get pods` - to check all the pods running

**Kubectl Describe Pod**

`kubectl describe pod` - to describe all the pods running in the Kubernetes environment. This returns comprehensive configuration including network IPs, container config, volumes and recent events.

`kubectl describe pod <pod_name>` - to get information about all pods  

**Kubectl Delete**

`kubectl delete pod <pod_name>` - to delete a specific pod from the cluster