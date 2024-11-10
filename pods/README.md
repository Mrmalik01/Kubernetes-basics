## Pods

- One or more containers in a Pod
- Pod containers all share a container network
- One IP address per Pod

**Pod Declaration**

- Container image
- Container ports
- Container restart policy
- Resource limits

**Manifest Files**

- Declare desired properties
- Manifests can describe all kinds of resources
- The spec contains resource-specific properties

**Manifests in Action**

- `kubectl create` - sends manifest to Kubernetes API Server
    - API Server does the following for Pod manifests
        1. Select a node with sufficient resources
        2. Schedule Pod onto node
        3. Node pulls Pod’s container image
        4. Starts Pod’s container

**Why use Manifests?**

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