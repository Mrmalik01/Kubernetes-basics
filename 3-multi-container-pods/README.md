## Multi-Container Pods

### Namespace
It is used for isolating resources according to users, environments, or applications. We can even use Role-based access control (RBAC) to secure access per namespace

**Namespace**
```
apiVersion: v1
kind: Namespace
metadata:
  name: ms-v1
  labels:
    app: counter
```

Multi-Countainer Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: redis
      image: redis:latest
      ports:
        - containerPort: 6379
      imagePullPolicy: IfNotPresent

    - name: server
      image: lrakai/microservices:server-v1
      ports:
        - containerPort: 8080
      env:
        - name: REDIS_URL
          value: redis://localhost:6379
    
    - name: counter
      image: lrakai/microservices:counter-v1
      env:
        - name: API_URL
          value: http://localhost:8080

    - name: poller
      image: lrakai/microservices:poller-v1
      env:
        - name: API_URL
          value: http://localhost:8080
```

In Kubernetes, imagePullPolicy controls the conditions under which the container image specified for a Pod is pulled (downloaded) from the image registry. The imagePullPolicy has three possible options:

1. IfNotPresent
Behavior: The image is pulled only if it is not already present on the node.
Common Use: This is the most common and default policy for tagged images other than latest.
Use Case: Suitable for environments where images rarely change or when network bandwidth should be conserved.
2. Always
Behavior: Kubernetes pulls the image every time the Pod is created, regardless of whether the image already exists on the node.
Common Use: This is the default policy for the latest tag, ensuring that the most recent version of the image is always pulled.
Use Case: Useful for development or test environments where the image may frequently change, or when you want to ensure that the container always uses the latest image version.
3. Never
Behavior: Kubernetes will never pull the image; instead, it will use only the local image if it is present.
Common Use: This is not the default for any tag and must be explicitly set.
Use Case: Suitable for fully offline environments or for situations where the image is preloaded onto the node manually.

| Option        | Description                                       | Default For                  | Use Case                                        |
|---------------|---------------------------------------------------|------------------------------|-------------------------------------------------|
| IfNotPresent  | Pulls only if the image is missing on the node    | Tagged images (other than latest) | Stable images with infrequent updates          |
| Always        | Pulls the image every time a Pod is created       | latest tag images            | Development or environments needing the latest version |
| Never         | Does not pull the image, uses local version only  | None                         | Offline use or preloaded images only            |

For database images in production, use imagePullPolicy: IfNotPresent. This ensures stability, avoids frequent and potentially disruptive image pulls, and reduces network dependency during restarts. In development, you might opt for Always if testing changes to the database image

This command is used for creating pods within a specific namespace using the **-n** option -   
`❯ kubectl create -f 3-multi-container-pods/3.2-multi_container.yaml -n ms-v1`

### Kubernetes Commands

**To create a Pod with a specific namespace**

`kubectl create -f <name_of_pod>.yaml -n <name_of_namespace>` 

**To describe a Pod with a specific namspace**

`kubectl describe -n <name_of_namespace> pod <name_of_pod>`

**To get the logs for a specific container within a Pod**

`kubectl logs -n <name_of_namespace> <pod_name> <app_name> --tail 10`

You add add tail option to get the 10 most recent log data from your container

-f to get the live 10 most recent logs

`kubectl logs -n microservice counter-app counter-poller  --tail 10 -f`