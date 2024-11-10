## Services

The pods can be restarted in different nodes or could be assigned with different IPs, therefore Pods IP can change and therefore if we cannot expose them directly (unless we assign them static IP address somehow?) Thats where the service concept helps. It acts as a load balancer and manages the requests.

![Diagram1.5](diagrams/diagram1.5.png)

- **Networking:** A service defines the networking rules for accessing Pods in the cluster and from the internet
    - Use labels to select a group of Pods
    - Service has a fixed IP address
      - It allows the service to be accessible within the containers network and also outside the cluster itself
- **Load Balancing:** Distribute requests across Pods in the group

![Diagram1.6](diagrams/diagram1.6.png)

Using Service as a Web Server
```
apiVersion: v1
kind: Service
metadata:
  name: web-server
  labels:
    app: web-server
spec:
  ports:
  - port: 80
  selector:
    app: nginx-web-server
  type: NodePort

```

**Output**  
```
> Kubectl get services`
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        5h17m
web-server   NodePort    10.111.237.210   <none>        80:32085/TCP   7s

> Kubectl describe services web-server
Name:                     web-server
Namespace:                default
Labels:                   app=web-server
Annotations:              <none>
Selector:                 app=nginx-web-server
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.111.237.210
IPs:                      10.111.237.210
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32085/TCP
Endpoints:                10.244.0.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:
  Type     Reason                        Age   From                       Message
  ----     ------                        ----  ----                       -------
  Warning  FailedToUpdateEndpointSlices  98s   endpoint-slice-controller  Error updating Endpoint Slices for Service default/web-server: failed to create EndpointSlice for Service default/web-server: Unauthorized

# To get the IP address of the node itself
> kubectl describe nodes | grep -i address -A 1
Addresses:
  InternalIP:  192.168.49.2
```

### Services Types

1. NodePort
   - Purpose: Exposes the service on a static port on each node in the cluster.
   - How It Works: When you create a NodePort service, Kubernetes assigns a port (usually in the range 30000–32767) on each node. External clients can access the service by sending a request to any node's IP address on this port, which Kubernetes then forwards to the appropriate Pods.
   - Usage: Ideal for exposing services directly to external networks. However, it's generally used for development or testing since it exposes nodes directly to external traffic, which could pose security risks in production environments.
2. ClusterIP (Default)
   - Purpose: Exposes the service only within the Kubernetes cluster, making it accessible only to other services or Pods within the same cluster.
   - How It Works: Kubernetes assigns an internal IP (cluster IP) to the service, which acts as a virtual IP. Only other Pods in the cluster can access this IP.
   - Usage: ClusterIP is the default and is suitable for internal microservices communication where you don’t need external access.
3. LoadBalancer
   - Purpose: Exposes the service to the internet using a cloud provider’s load balancer, such as those provided by AWS, Azure, or GCP.
   - How It Works: When a LoadBalancer service is created, Kubernetes automatically provisions an external load balancer that routes traffic to the service. This is commonly supported in managed Kubernetes services where cloud providers handle the setup.
   - Usage: This type is suitable for production-grade applications where you need reliable, managed external access to your application
4. ExternalName
   - Purpose: Maps a Kubernetes service to an external DNS name rather than exposing a Pod directly.
   - How It Works: Instead of having a cluster IP or node port, the ExternalName service uses DNS CNAME records to redirect traffic to a specific external domain. The service simply forwards requests to this external address.
   - Usage: This type is useful for integrating external services into your cluster, allowing Pods to connect to external services through a Kubernetes-defined name.

**Summary Table**
| Service Type    | Exposes to               | Accessibility                             | Use Cases                                |
|-----------------|--------------------------|-------------------------------------------|------------------------------------------|
| ClusterIP       | Internal (within cluster)| Only accessible within the cluster        | Internal services communication          |
| NodePort        | External (on node IP)    | Accessible via node IP and static port    | Development/testing with direct node access |
| LoadBalancer    | External (via cloud LB)  | Accessible via external IP assigned by LB | Production-ready external access         |
| ExternalName    | External service         | Resolves to an external DNS name          | Integration with external services       |
