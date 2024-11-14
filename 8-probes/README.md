## Probes 

Probes are health checks in Kubernetes. We do not want to send traffic to unavailable pods or unfunctional pods that are not ready to accept the traffic. It could be the pods are crashing and require restart or there's some bug in the code.

### 1. Readiness Probes — "Is the Pod Ready to Handle Traffic?"
**Purpose**: A Readiness Probe checks if a Pod is ready to start receiving traffic. This is particularly important when the application in the Pod has to be "ready" before it can respond to requests (like a web server).

**Example of External Dependencies: Imagine a Pod that needs to**:

- Connect to a database
- Fetch some data from an external API
- Load certain configurations or files  
 
**How It Works**: If the Readiness Probe fails, Kubernetes won’t send any traffic to that Pod. This means services will only route traffic to Pods that are fully ready to handle it.

**When to Use**: Use Readiness Probes when a Pod takes time to get ready after it starts, or if it needs to check for dependencies like databases or APIs before handling requests.

### 2. Liveness Probes — "Is the Pod Still Healthy?"
**Purpose**: A Liveness Probe checks if the Pod is "alive" or in a healthy state. If the Pod becomes unresponsive or "broken" (stuck in an error or crash), the Liveness Probe can detect this and let Kubernetes restart the Pod automatically.

**Example**: If an application has a bug that causes it to freeze or stop responding, the Liveness Probe will catch it and trigger a restart, helping to keep the application available without manual intervention.

**How It Works**: If the Liveness Probe fails, Kubernetes considers the Pod unhealthy and restarts it to try to fix the issue.

**When to Use**: Use Liveness Probes if your application may occasionally need a restart to recover from crashes or bugs, or if it’s critical for the application to be responsive at all times.

**Readiness Probe**: Makes sure the Pod is ready to serve traffic. If it’s not ready, Kubernetes won’t send requests to it.  
**Liveness Probe**: Checks if the Pod is healthy and responsive. If it’s broken, Kubernetes will restart it automatically.

### Highlights
- Probes can be declared in a Pod's containers, within the manifest file
- All containers probes must pass for the Pod to pass. In case of multi-container pods, all the container should pass the test, otherwise the whole pod will become unavailable.
- Probe actions can be a command that runs in the container, an HTTP Get request, or opening a TCP socket.
- By default, the probes check the pods every 10 seconds.

When it comes to our REDIS container, i.e. the data tier, liveness check can be done by opening the TCP socket and the readiness by `redis-cli ping` command.

Data Tier (Redis)
- Liveness: Open TCP socker
- Readiness: redis-cli ping command

**7.2-data_tier.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: data-tier-svc
  labels:
    app: micro
    tier: data
    env: dev
spec:
  ports:
  - port: 6379
    name: REDIS
    protocol: TCP
  selector:
    tier: data
    env: dev
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-tier-dp
  labels:
    app: micro
    tier: data
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
        tier: data
        env: dev
    spec:
      containers:
      - name: redis
        image: redis:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 6379
        livenessProbe:
          tcpSocker:
            port: redis
          initialDelaySeconds: 15
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 5
```
We have added some initial delay to allow the container to get started before we start the checks. **Three consecutive probes needs to fail before it marks the pods as unusable.**

App Tier (Server)
- Liveness: HTTP GET /probe/liveness
- Readiness: HTTP GET /probe/readiness
- 


### Kubernetes Commands

#### Command to pause rollout
`kubectl rollout -n <namespace> pause deployment <deployment_name>`

