# Day 51 – Kubernetes Manifests and Your First Pods

# TASK: 
Set up a cluster. Today we will actually deploy something. We will learn the structure of a Kubernetes manifest file and use it to create Pods — the smallest deployable unit in Kubernetes. By the end of today, we should be able to write a Pod definition from scratch without looking at docs.

---

## The Anatomy of a Kubernetes Manifest

Every Kubernetes resource is defined using a YAML manifest with four required top-level fields:

```yaml
apiVersion: v1          # Which API version to use
kind: Pod               # What type of resource
metadata:               # Name, labels, namespace
  name: my-pod
  labels:
    app: my-app
spec:                   # The actual specification (what you want)
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

- `apiVersion` — tells Kubernetes which API group to use. For Pods, it is `v1`.
- `kind` — the resource type. Today it is `Pod`. Later you will use `Deployment`, `Service`, etc.
- `metadata` — the identity of your resource. `name` is required. `labels` are key-value pairs used for organization and selection.
- `spec` — the desired state. For a Pod, this means which containers to run, which images, which ports, etc.


---

## Create Your First Pod (Nginx)


`nginx-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f nginx-pod.yaml
```

```bash
kubectl get pods
kubectl get pods -o wide
```

```bash
# Detailed info about the pod
kubectl describe pod nginx-pod

# Read the logs
kubectl logs nginx-pod

# Get a shell inside the container
kubectl exec -it nginx-pod -- /bin/bash

# Inside the container, run:
curl localhost:80
exit
```

---
![](nginx-pod%20running.png)
![](exec%20in%20nginx-pod.png)
---


---

##  Create a Custom Pod (BusyBox)

`busybox-pod.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```


```bash
kubectl apply -f busybox-pod.yaml
kubectl get pods
kubectl logs busybox-pod
```

![](Busy-Box%20Pod.png)

Notice the `command` field — BusyBox does not run a long-lived server like Nginx. Without a command that keeps it running, the container would exit immediately and the pod would go into `CrashLoopBackOff`.
*Meaning* : As Nginx's Docker image has a built-in default process (the nginx server itself) that runs forever in the foreground — so the container naturally stays alive. BusyBox has no default long-running process; if you don't give it a command, it runs nothing, finishes instantly, and exits.


---

##  Imperative vs Declarative


We have been using the declarative approach (writing YAML, then `kubectl apply`). Kubernetes also supports imperative commands:

```bash
# Create a pod without a YAML file
kubectl run redis-pod --image=redis:latest

# Check it
kubectl get pods
```

For extracting the YAML that Kubernetes generated:
```bash
kubectl get pod redis-pod -o yaml
```

---
![](yaml%20created%20by%20k8s.png)
---


- On Comparing this output with your hand-written manifests. We Noticed how much extra metadata Kubernetes adds automatically (status, timestamps, uid, resource version).

- We can also use dry-run to generate YAML without creating anything:
  ```bash
  kubectl run test-pod --image=mysql --dry-run=client -o yaml
  ```


- This is a powerful trick — use it to quickly scaffold a manifest, then customize it.

  ![](Dry%20Run%20Yaml%20created%20by%20k8s.png)


- On Saving the dry-run output to a file and comparing its structure with our nginx-pod.yaml. We got ans to some questions...
What fields are the same? What is different?
  
  ![](Difference%20btw%20k8s%20gen%20yaml%20and%20our%20yaml.png)


---

##  Validate Before Applying


Before applying a manifest, We can validate it:
```bash
# Check if the YAML is valid without actually creating the resource
kubectl apply -f nginx-pod.yaml --dry-run=client

# Validate against the cluster's API (server-side validation)
kubectl apply -f nginx-pod.yaml --dry-run=server
```

- Now intentionally break your YAML (remove the `image` field or add an invalid field) and run dry-run again. See what error you get.

  ![](Dry%20Run%20Yaml%20created%20by%20k8s.png)


---

##  Pod Labels and Filtering


Labels are how Kubernetes organizes and selects resources. We added labels in our manifests — now we will use them:

```bash
# List all pods with their labels
kubectl get pods --show-labels

# Filter pods by label
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev

# Add a label to an existing pod
kubectl label pod nginx-pod environment=production

# Verify
kubectl get pods --show-labels

# Remove a label
kubectl label pod nginx-pod environment-
```

---
![](Pod%20with%20Label%20Names.png)
---
![](Pods%20with%20specific%20label%20name.png)
---
![](Filtering%20with%20Labels.png)
---


---

##  Clean Up
Delete all the pods you created:

```bash
# Delete by name
kubectl delete pod nginx-pod
kubectl delete pod busybox-pod
kubectl delete pod redis-pod

# Or delete using the manifest file
kubectl delete -f nginx-pod.yaml

# Verify everything is gone
kubectl get pods
```

---
![](Deleting%20Pods.png)
---


*Notice that when you delete a standalone Pod, it is gone forever. There is no controller to recreate it. This is why in production you use Deployments (We will read about it tommorrow) instead of bare Pods.*
