## Init Containers

Sometimes we need to perform certain tasks before we start the service, pods or deployments. It could be fetching some secret key from secret store, initiate certain connection or even perform some tests or checks. 
- In some cases, we need to wait for a service, downloads, dynamic or decisions before starting a Pod's containers.
- Prefer to separate initialisation wait logic from the container image
- Initilisation is tighly coupled to the main  application
- Init containers allows you to run initialisation tasks before starting the main containers.
- Pods can declare any number of init containers
- Init containers run in order and to completion
- Use their own images
    - Sometimes you do not want your main containers to use the same image as the init containers because of security, compliance and many other reasons
- Easy way to block or delay starting an application
- Run every time a Pod is created
- They do not support `readinessProbe`

![Diagram1.8](../diagrams/diagram1.8.png)

Lets add an init container to our app-tier that will wait for Redis before starting any application servers.
```
apiVersion: v1
kind: Service
metadata:
  name: app-tier-svc
  labels:
    app: micro
    env: dev
    tier: app
spec:
  ports:
  - port: 8080
    name: server
  selector:
    tier: app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-tier-dp
  labels:
    app: micro
    env: dev
    tier: app
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: app
      env: dev
  template:
    metadata:
      labels:
        env: dev
        tier: app
        app: micro
    spec:
      containers:
      - name: server
        image: lrakai/microservices:server-v1
        imagePullPolicy: Always
        ports:
          - containerPort: 8080
        env:
          - name: REDIS_URL
            # Environment variable service discovery
            # Naming pattern:
            #   IP address: <all_caps_service_name>_SERVICE_HOST
            #   Port: <all_caps_service_name>_SERVICE_PORT
            #   Named Port: <all_caps_service_name>_SERVICE_PORT_<all_caps_port_name>
            value: redis://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)
            # In multi-container example value was
            # value: redis://localhost:6379 
          - name: DEBUG
            value: express:*
        livenessProbe:
          httpGet:
            path: /probe/liveness
            port: server
          initialDelaySeconds: 5
        readinessProbe:
          httpGet:
            path: /probe/readiness
            port: server
          initialDelaySeconds: 3
      initContainers:
        - name: await-redis
          image: lrakai/microservices:server-v1
          env:
          - name: REDIS_URL
            value: http://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)
          command:
            - npm
            - run-script
            - await-redis
```

```
> kubectl describe pod -n pb app-tier-dp-6c6d4c56df-mwvph
Name:             app-tier-dp-6c6d4c56df-mwvph
Namespace:        pb
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Thu, 14 Nov 2024 21:06:39 +0000
Labels:           app=micro
                  env=dev
                  pod-template-hash=6c6d4c56df
                  tier=app
Annotations:      <none>
Status:           Running
IP:               10.244.0.107
IPs:
  IP:           10.244.0.107
Controlled By:  ReplicaSet/app-tier-dp-6c6d4c56df
Init Containers:
  await-redis:
    Container ID:  docker://e68095499ccde06c5678c34596dba3d5d2d670e8881f3158fd7cba4f768ffc72
    Image:         lrakai/microservices:server-v1
    Image ID:      docker-pullable://lrakai/microservices@sha256:9e3e3c45bb9d950fe7a38ce5e4e63ace2b6ca9ba8e09240f138c5df39d7b7587
    Port:          <none>
    Host Port:     <none>
    Command:
      npm
      run-script
      await-redis
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 14 Nov 2024 21:06:40 +0000
      Finished:     Thu, 14 Nov 2024 21:06:41 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      REDIS_URL:  http://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xr7n9 (ro)
Containers:
  server:
    Container ID:   docker://0f95dda8ca3ffd8f795cd84a53cd8731c976cd09ec524a63527a69bf243ba595
    Image:          lrakai/microservices:server-v1
    Image ID:       docker-pullable://lrakai/microservices@sha256:9e3e3c45bb9d950fe7a38ce5e4e63ace2b6ca9ba8e09240f138c5df39d7b7587
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 14 Nov 2024 21:06:41 +0000
    Ready:          False
    Restart Count:  0
    Liveness:       http-get http://:server/probe/liveness delay=5s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:server/probe/readiness delay=3s timeout=1s period=10s #success=1 #failure=3
    Environment:
      REDIS_URL:  redis://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)
      DEBUG:      express:*
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xr7n9 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-xr7n9:
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
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m36s                default-scheduler  Successfully assigned pb/app-tier-dp-6c6d4c56df-mwvph to minikube
  Normal   Pulled     2m35s                kubelet            Container image "lrakai/microservices:server-v1" already present on machine
  Normal   Created    2m35s                kubelet            Created container await-redis
  Normal   Started    2m35s                kubelet            Started container await-redis
  Normal   Pulling    2m34s                kubelet            Pulling image "lrakai/microservices:server-v1"
  Normal   Pulled     2m34s                kubelet            Successfully pulled image "lrakai/microservices:server-v1" in 246ms (246ms including waiting). Image size: 65112854 bytes.
  Normal   Created    2m34s                kubelet            Created container server
  Normal   Started    2m34s                kubelet            Started container server
```

The init container ran before the actual server code. From the event logs, we can see the third and fourth step. We can also verify the init container condition by checking the logs directly.

```
> kubectl logs  -n pb app-tier-dp-6c6d4c56df-mwvph -c await-redis
npm info it worked if it ends with ok
npm info using npm@3.10.10
npm info using node@v6.11.0
npm info lifecycle server@1.0.0~preawait-redis: server@1.0.0
npm info lifecycle server@1.0.0~await-redis: server@1.0.0

> server@1.0.0 await-redis /usr/src/app
> node await.js

node_redis: WARNING: You passed "http" as protocol instead of the "redis" protocol!
Connection ok
npm info lifecycle server@1.0.0~postawait-redis: server@1.0.0
npm info ok 
```

### Kubernetes Commands

#### Command to check the logs from an init container within a Pod

`kubectl logs -n <namespace> <pod_name> -c <init_container_name>`