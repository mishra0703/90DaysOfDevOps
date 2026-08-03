# Day 58 – Metrics Server and Horizontal Pod Autoscaler (HPA)

## Task
Yesterday we set resource requests and limits. Today we put that to work. Install the Metrics Server so Kubernetes can see actual resource usage, then set up a Horizontal Pod Autoscaler that scales our app up under load and back down when things calm down.

---

## Install the Metrics Server


1. Check if it is already running: `kubectl get pods -n kube-system | grep metrics-server`
2. If not, install it:
   - Minikube: `minikube addons enable metrics-server`
   - Kind/kubeadm: apply the official manifest from the metrics-server GitHub releases

   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/  latest/download/components.yaml
   ```


3. On local clusters, you may need the `--kubelet-insecure-tls` flag (never in production)

   ```bash
   kubectl -n kube-system patch deployment/metrics-server --type=json -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
   ```


4. Wait 60 seconds, then verify: `kubectl top nodes` and `kubectl top pods -A`

---
![](metrics%20of%20all%20nodes%20and%20pods.png)
---
---

## Explore kubectl top


1. Run `kubectl top nodes`, `kubectl top pods -A`, `kubectl top pods -A --sort-by=cpu`

   ---
   ![](sorting%20metrics%20data.png)
   ---


2. `kubectl top` shows real-time usage, not requests or limits — these are different things
   - `kubectl top` displays the live, actual CPU and memory your pods or nodes are consuming right now
   - We can use `kubectl describe` to check your requests and limits.
   - `kubectl top` is for viewing live activity. `Requests` and `Limits` are rules for scheduling and safety


3. Data comes from the Metrics Server, which polls kubelets every 15 seconds
   - It means that a central tool called the `Metrics Server` asks individual worker nodes for fresh resource usage numbers, such as CPU and memory use.
   - **Polls** : The act of asking for data on a set schedule rather than waiting for data to be sent.
   - **Kubelet**: An agent running on each worker node that watches over the running containers


---

## Create a Deployment with CPU Requests


1. Write a Deployment manifest using the `registry.k8s.io/hpa-example` image (a CPU-intensive PHP-Apache server)
2. Set `resources.requests.cpu: 200m` — HPA needs this to calculate utilization percentages
3. Expose it as a Service: `kubectl expose deployment php-apache --port=80`


Without CPU requests, HPA cannot work — this is the most common HPA setup mistake.
   
   ---
   ![](Deployment%20for%20HPA.png)
   ---

---

## Create an HPA (Imperative)


1. Run: `kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`
2. Check: `kubectl get hpa` and `kubectl describe hpa php-apache`
3. TARGETS may show `<unknown>` initially — wait 30 seconds for metrics to arrive


This scales up when average CPU exceeds 50% of requests, and down when it drops below.

---
![](HPA%20Created%20with%2050percent%20CPU%20limit.png)
---
---

## Generate Load and Watch Autoscaling


1. Start a load generator: `kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"`
2. Watch HPA: `kubectl get hpa php-apache --watch`
3. Over 1-3 minutes, CPU climbs above 50%, replicas increase, CPU stabilizes
4. Stop the load: `kubectl delete pod load-generator`
5. Scale-down is slow (5-minute stabilization window) — you do not need to wait

---
![](HPA%20Scaled%20Up%20as%20CPU%20Spikes%20Up.png)
---
![](Replicas%20creating%20autom%20as%20CPU%20spikes%20up%20.png)
---
### *HPA Scaling Down Automatically after 5-7mins of stabilization window :*

![](HPA%20Scaling%20Down%20the%20app%20-%201.png)
---
![](HPA%20Scaling%20Down%20the%20app%20-%202.png)
---
---

## Create an HPA from YAML (Declarative)


1. Delete the imperative HPA: `kubectl delete hpa php-apache`
2. Write an HPA manifest using `autoscaling/v2` API with CPU target at 50% utilization
3. Add a `behavior` section to control scale-up speed (no stabilization) and scale-down speed (300 second window)
4. Apply and verify with `kubectl describe hpa`


*Important point :*
   - `autoscaling/v2` supports multiple metrics and fine-grained scaling behavior that the imperative command cannot configure.

### *What does the `behavior` section control?*

- It can be use to tunes how fast or slow your workload scales up and down
-  It lets us set stabilization windows to prevent flapping and define explicit rules (policies) limiting the number or percentage of pods added or removed over time


---
![](Declarative%20HPA.png)
---
---

## *Points to Remember :*

- HPA requires `resources.requests` — without them TARGETS shows `<unknown>`
- `kubectl top` = actual usage. `kubectl describe pod` = configured requests/limits
- HPA checks every 15 seconds. Scale-up is fast, scale-down has a 5-minute stabilization window
- `autoscaling/v1` = CPU only. `autoscaling/v2` = CPU + memory + custom metrics
- Formula: `desiredReplicas = ceil(currentReplicas * (currentUsage / targetUsage))`
- HPA works with Deployments, StatefulSets, and ReplicaSets

