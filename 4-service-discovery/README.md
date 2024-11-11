## Service Discovery

We need Services to enable the horizontal scaling in the above example. We will isolate each layer in our three-layer architecture and create a service for each of them to allow scaling. 

- Services support multi-Pod design
- Provides static IP for each tier
- Handles Pod IP change
- Load balancer

**Service Discovery Mechanism**
We can use the below two options to enable service discovery in containers. We can use the DNS service to allow communication btw services or use the environment names
- DNS
    - DNS records are automatically created in the cluster’s DNS
    - Containers automatically configured to query cluster’s DNS
- Environment Variables
    - Services address automatically injected in containers
    - Environment variables following naming convention based on the service name
  

We can create all resources within the same yaml file using the separator "---"

**4.2-data_tier.yaml**
This file contains both the service and the pods definitions. We are using the ClusterIP because its an internal database service which shouldn't be available outside
```
apiVersion: v1
kind: Service
metadata:
  name: data-tier-service
  labels:
    app: microservices
    env: dev
spec:
  ports:
  - port: 6379
    protocol: TCP
    name: redis
  selector:
    tier: data
  type: ClusterIP # This is an internal service, only accessible within the cluster
---
apiVersion: v1
kind: Pod
metadata:
  name: data-tier-pod
  labels:
    tier: data
    env: dev
    app: microservices
spec:
  containers:
    - name: redis
      image: redis:latest
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 6379
```
**Output**
```
❯ kubectl describe services -n sd-4 data-tier-service
Name:                     data-tier-service
Namespace:                sd-4
Labels:                   app=microservices
                          env=dev
Annotations:              <none>
Selector:                 tier=data
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.13.24
IPs:                      10.96.13.24
Port:                     redis  6379/TCP
TargetPort:               6379/TCP
Endpoints:                10.244.0.7:6379
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
❯ kubectl describe pods data-tier-pod -n sd-4
Name:             data-tier-pod
Namespace:        sd-4
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Mon, 11 Nov 2024 00:06:46 +0000
Labels:           app=microservices
                  env=dev
                  tier=data
Annotations:      <none>
Status:           Running
IP:               10.244.0.7
IPs:
  IP:  10.244.0.7
Containers:
  redis:
    Container ID:   docker://d8e1b7553a8b768f66c2208e24e15a0bb1f6b6893208b1cb1fd15a05689dd380
    Image:          redis:latest
    Image ID:       docker-pullable://redis@sha256:a06cea905344470eb49c972f3d030e22f28f632c1b4f43bbe4a26a4329dd6be5
    Port:           6379/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 11 Nov 2024 00:06:47 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ccl25 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-ccl25:
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
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m50s  default-scheduler  Successfully assigned sd-4/data-tier-pod to minikube
  Normal  Pulled     2m49s  kubelet            Container image "redis:latest" already present on machine
  Normal  Created    2m49s  kubelet            Created container redis
  Normal  Started    2m49s  kubelet            Started container redis
```

**4.3-app_tier.yaml**
In this configuration, we are using the environment variable approach to put the reference to the data tier as the url and port. In Kubernetes, each service gets an environment variable names with the format   
`<upper_case_service_name>_SERVICE_HOST`: To represent the IP address of the pod
`<upper_case_service_name>_SERVICE_PORT_<upper_case_port_name>`: To represent the port number of the pod
```
apiVersion: v1
kind: Service
metadata:
  name: app-tier-service
  labels:
    app: microservices
    env: dev
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    tier: app
    env: dev
---
apiVersion: v1
kind: Pod
metadata:
  name: app-tier-pod
  labels:
    tier: app
    env: dev
    app: microservices
spec:
  containers:
    - name: server
      image: lrakai/microservices:server-v1
      ports:
        - containerPort: 80
      env:
        - name: REDIS_URL
          value: redis://$(DATA_TIER_SERVICE_SERVICE_HOST):$(DATA_TIER_SERVICE_SERVICE_PORT_REDIS)
```

**4.4-support_tier.yaml**
In this pod, we do not need a service as it does not accept any traffic from any other services. Also, we are using the DNS discovery to get the IP address associated with the service. However, it is not possible with the port. So we can either hardcode it or use the service discovery environment variables.

Port: Needs to be extracted from SRC DNS record. You can get service port information using DNS SRV records, but that isn't something that we can use in the manifest file. It is possible to use the DNS SRV port record to configure the pod on start-up using something called init containers

Kubernetes adds a DNS A records for every service.
IP Address: `<service-name>.<namespace>`

```
apiVersion: v1
kind: Pod
metadata:
  name: support-tier-pod
  labels:
    tier: support
    env: dev
    app: microservices
spec:
  containers:
    - name: counter
      image: lrakai/microservices:counter-v1
      ports:
        - containerPort: 80
      env:
        - name: API_URL
          value: http://app-tier-service:8080

    - name: poller
      image: lrakai/microservices:poller-v1
      ports:
        - containerPort: 80
      env:
        - name: API_URL
          value: http://app-tier-service:$(APP_TIER_SERVICE_SERVICE_PORT)
```