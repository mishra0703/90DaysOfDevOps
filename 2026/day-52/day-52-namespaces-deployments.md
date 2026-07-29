# Day 52 – Kubernetes Namespaces and Deployments

## Task
Yesterday We created standalone Pods. The problem? Delete a Pod and it is gone forever — no one recreates it. Today We will fix that with Deployments, the real way to run applications in Kubernetes. We will also learn Namespaces, which let us organize and isolate resources inside a cluster.

---

## Expected Output
- At least 2 namespaces created and used
- A Deployment running with multiple replicas
- A scaled Deployment and a rolling update performed

---

## Explore Default Namespaces


Kubernetes comes with built-in namespaces. List them:

```bash
kubectl get namespaces
```

Pods we will definitely see :
- `default` — where your resources go if you do not specify a namespace
- `kube-system` — Kubernetes internal components (API server, scheduler, etcd and controller manager)
- `kube-public` — publicly readable resources
- `kube-node-lease` — node heartbeat tracking

Check what is running inside `kube-system`:
```bash
kubectl get pods -n kube-system
```

- These are the control plane components keeping your cluster alive. Do not touch them.

    ![](Default%20Namespaces.png)

---



## Create and Use Custom Namespaces


Create two namespaces — one for a development environment and one for staging:

```bash
kubectl create namespace dev
kubectl create namespace staging
```

Verify they exist:
```bash
kubectl get namespaces
```

---
![](creating%20namespace.png)
---



We can also create a namespace from a manifest:
```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

```bash
kubectl apply -f namespace.yaml
```

---
![](creating%20namespace%20through%20yaml%20file.png)
---



Running a pod in a specific namespace:
```bash
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
```
---
![](running%20pods%20in%20specific%20namespace.png)
---



Listing pods across all namespaces:
```bash
kubectl get pods -A
```

---
![](checking%20pods%20in%20specific%20namespace.png)
---


*Note :* `kubectl get pods` without `-n` only shows the `default` namespace. We must specify `-n <namespace>` or use `-A` to see everything.


---

## Create Your First Deployment


A Deployment tells Kubernetes: "I want X replicas of this Pod running at all times." If a Pod crashes, the Deployment controller recreates it automatically.

Create a file `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3

# selector.matchLabels in a Deployment must match template.metadata.labels — if they do not match, the Deployment will not manage the Pods 
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```


Key differences from a standalone Pod:
- `kind: Deployment` instead of `kind: Pod`
- `apiVersion: apps/v1` instead of `v1`
- `replicas: 3` tells Kubernetes to maintain 3 identical pods
- `selector.matchLabels` connects the Deployment to its Pods
- `template` is the Pod template — the Deployment creates Pods using this blueprint

Apply it:
```bash
kubectl apply -f nginx-deployment.yaml
```

---
![](Pod%20Deployment.png)
---

Check the result:
```bash
kubectl get deployments -n dev
kubectl get pods -n dev
```


---
![](Deployments%20and%20Pods%20in%20dev%20namespace.png)
---




---

## Self-Healing — Delete a Pod and Watch It Come Back


This is the key difference between a Deployment and a standalone Pod.

```bash
# List pods
kubectl get pods -n dev

# Delete one of the deployment's pods (use an actual pod name from your output)
kubectl delete pod <pod-name> -n dev

# Immediately check again
kubectl get pods -n dev
```

- The Deployment controller detects that only 2 of 3 desired replicas exist and immediately creates a new one. The deleted pod is replaced within seconds.

---
![](Deployment%20mainting%203%20pods%20everytime.png)
---
---

## Scale the Deployment


Change the number of replicas:
```bash
# Scale up to 5
kubectl scale deployment nginx-deployment --replicas=5 -n dev
kubectl get pods -n dev

# Scale down to 2
kubectl scale deployment nginx-deployment --replicas=2 -n dev
kubectl get pods -n dev
```

- Watch how Kubernetes creates or terminates pods to match the desired count.
- We can also scale by editing the manifest — change `replicas: 4` in your YAML file and run `kubectl apply -f nginx-deployment.yaml` again.


---
![](Scaling%20Deployment.png)
---
---

## Rolling Update


Update the Nginx image version to trigger a rolling update:
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
```

Watch the rollout in real time:
```bash
kubectl rollout status deployment/nginx-deployment -n dev
```
---
![](deployment%20rollout%20(updating%20container%20image).png)
---

Kubernetes replaces pods one by one — old pods are terminated only after new ones are healthy. This means zero downtime.

Check the rollout history:
```bash
kubectl rollout history deployment/nginx-deployment -n dev
```

Now roll back to the previous version:
```bash
kubectl rollout undo deployment/nginx-deployment -n dev
kubectl rollout status deployment/nginx-deployment -n dev
```
---
![](undo%20rollout%20of%20deployment.png)
---



Verify the image is back to the previous version:
```bash
kubectl describe deployment nginx-deployment -n dev | grep Image
```

---
![](checking%20image%20version%20after%20and%20before%20rollout.png)
---
---

## Clean Up


```bash
# This won't work as Pods will be creating again and again
kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging

# Deleting a deployment also deletes all the pods running inside it 
kubectl delete deployment nginx-deployment -n dev

# Deleting a namespace removes everything inside it. Be very careful with this in production
kubectl delete namespace dev staging production
```


```bash
kubectl get namespaces
kubectl get pods -A
```

---
![](Deleting%20namespaces.png)
---
