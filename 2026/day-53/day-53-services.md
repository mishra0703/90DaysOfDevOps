# Day 53 – Kubernetes Services

## Task
We have Deployments running multiple Pods, but how do we actually talk to them? Pods get random IP addresses that change every time they restart. Services solve this by giving your Pods a stable network endpoint. Today we will create different types of Services and understand when to use each one.


---

## Why Services?

Every Pod gets its own IP address. But there are two problems:
1. Pod IPs are **not stable** — when a Pod restarts or gets replaced, it gets a new IP
2. A Deployment runs **multiple Pods** — which IP do you connect to?

A Service solves both problems. It provides:
- A **stable IP and DNS name** that never changes
- **Load balancing** across all Pods that match its selector

```
[Client] --> [Service (stable IP)] --> [Pod 1]
                                   --> [Pod 2]
                                   --> [Pod 3]
```

---

## Deploy the Application


First, we will create a Deployment that we will expose with Services. Create `app-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f app-deployment.yaml
kubectl get pods -o wide
```



- Note the individual Pod IPs. These will change if pods restart — that is the problem Services fix.

    ![](pods%20ip%20got%20change%20after%20re-creation.png)


---

## ClusterIP Service (Internal Access)


ClusterIP is the default Service type. It gives our Pods a stable internal IP that is only reachable from within the cluster.

Create `clusterip-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

Key fields:
- `selector.app: web-app` — this Service routes traffic to all Pods with the label `app: web-app`
- `port: 80` — the port the Service listens on
- `targetPort: 80` — the port on the Pod to forward traffic to

```bash
kubectl apply -f clusterip-service.yaml
kubectl get services
```



- We should see `web-app-clusterip` with a CLUSTER-IP address. This IP is stable — it will not change even if Pods restart.

---
![](Got%20cluster%20ip.png)
---
![](cluster%20ip%20is%20same%20even%20after%20re-creating%20all%20pods.png)
---


Now test it from inside the cluster:
```bash
# Run a temporary pod to test connectivity
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh

# Inside the test pod, run:
wget -qO- http://web-app-clusterip
exit
```

- We saw the Nginx welcome page. The Service load-balanced your request to one of the 3 Pods.

    ![](hit%20from%20other%20pod%20in%20the%20same%20cluster.png)


---

## Discover Services with DNS


Kubernetes has a built-in DNS server. Every Service gets a DNS entry automatically:
```
<service-name>.<namespace>.svc.cluster.local
```

Test this:
```bash
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh

# Inside the pod:
# Short name (works within the same namespace)
wget -qO- http://web-app-clusterip

# Full DNS name
wget -qO- http://web-app-clusterip.default.svc.cluster.local

# Look up the DNS entry
nslookup web-app-clusterip
exit
```

Both the short name and the full DNS name resolve to the same ClusterIP. In practice, you use the short name when communicating within the same namespace and the full name when reaching across namespaces.


---
![](Hitting%20pod%20with%20Dns.png)
---
---

## NodePort Service (External Access via Node)


A NodePort Service exposes your application on a port on every node in the cluster. This lets you access the Service from outside the cluster.

Create `nodeport-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

- `nodePort: 30080` — the port opened on every node (must be in range 30000-32767)
- Traffic flow: `<NodeIP>:30080` -> Service -> Pod:80

```bash
kubectl apply -f nodeport-service.yaml
kubectl get services
```

---
### Difference between ClusterIp and NodePort 
---
![](clusterip%20format.png)
---
---
![](nodePort%20format.png)
*You can see port mapping in port column*
---



Access the service:
```bash
# If using Minikube
minikube service web-app-nodeport --url

# If using Kind, get the node IP first
kubectl get nodes -o wide
# Then curl <node-internal-ip>:30080

# If using Docker Desktop
curl http://localhost:30080
```


---
![](accessed%20web-app%20using%20nodeport.png)
---
*Note :*
- We must use extraPortMappings in cluster's config file if we are running our Kubernetes cluster locally inside Kind (Kubernetes in Docker) and want to access the service via localhost.
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080 # The NodePort inside the cluster (In service.yml it is named as nodePort)
    hostPort: 30080      # The Port you will use on your machine
    protocol: TCP

```
---

## LoadBalancer Service (Cloud External Access)


In a cloud environment (AWS, GCP, Azure), a LoadBalancer Service provisions a real external load balancer that routes traffic to your nodes.

Create `loadbalancer-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f loadbalancer-service.yaml
kubectl get services
```
---
![](loadbalancer%20service.png)
---

- On a local cluster (Minikube, Kind, Docker Desktop), the EXTERNAL-IP will show `<pending>` because there is no cloud provider to create a real load balancer. This is expected.

- In a real cloud cluster, the EXTERNAL-IP would be a public IP address or hostname provisioned by the cloud provider.

- Kubernetes service shows `<pending>` because it is set to type: LoadBalancer, which requires a cloud provider's external load balancer to assign an IP address. Local clusters lack this cloud infrastructure by default, meaning no controller is available to allocate the IP.

---


## Understand the Service Types Side by Side


Check all three services:
```bash
kubectl get services -o wide
```

---
![](all%20three%20services.png)
---

---

### Difference b/w all three services

| Type | Accessible From | Use Case |
|------|----------------|----------|
| ClusterIP | Inside the cluster only | Internal communication between services |
| NodePort | Outside via `<NodeIP>:<NodePort>` | Development, testing, direct node access |
| LoadBalancer | Outside via cloud load balancer | Production traffic in cloud environments |

---

- Each type builds on the previous one:
    - LoadBalancer creates a NodePort, which creates a ClusterIP
    - So a LoadBalancer service also has a ClusterIP and a NodePort


Verify this:
```bash
kubectl describe service web-app-loadbalancer
```
We will see all three: a ClusterIP, a NodePort, and the LoadBalancer configuration.

*Note :* 
- LoadBalancer type still gets ClusterIp & NodePort automatically, since it's built on top of ClusterIP + NodePort
- LoadBalancer service = ClusterIP + NodePort

---
![](Loadbalancer%20Describe.png)
---
---

---

### *Points to Remember*

- `selector` in a Service must match `labels` on the Pods — if they do not match, the Service routes traffic to nothing
- `kubectl get endpoints <service-name>` shows which Pod IPs a Service is currently routing to
- `port` is what the Service listens on; `targetPort` is what the Pod listens on — they do not have to be the same number
- NodePort range is 30000-32767; if you do not specify `nodePort`, Kubernetes picks one automatically
- Use `kubectl describe service <name>` to see the full configuration including Endpoints
- `kubectl get services -o wide` shows the selector each service uses
- To test ClusterIP services, you must test from inside the cluster (use a temporary pod)
