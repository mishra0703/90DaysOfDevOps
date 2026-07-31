# Day 56 – Kubernetes StatefulSets

## Task
Deployments work great for stateless apps, but what about databases? We need stable pod names, ordered startup, and persistent storage per replica. Today we will learn StatefulSets — the workload designed for stateful applications like MySQL, PostgreSQL, and Kafka.

---

## Understand the Problem


1. Create a Deployment with 3 replicas using nginx
2. Check the pod names — they are random (`app-xyz-abc`)
3. Delete a pod and notice the replacement gets a different random name

---
![](pods-have%20diff%20names.png)
---

This is fine for web servers but not for databases where you need stable identity.

| Feature | Deployment | StatefulSet |
|---|---|---|
| Pod names | Random | Stable, ordered (`app-0`, `app-1`) |
| Startup order | All at once | Ordered: pod-0, then pod-1, then pod-2 |
| Storage | Shared PVC | Each pod gets its own PVC |
| Network identity | No stable hostname | Stable DNS per pod |


- Why would random pod names be a problem for a database cluster?
    - Database clusters (like MySQL replication) needs stable, predictable identities — each node has a specific role (primary/replica, or a specific shard) that other nodes reference by name/address. 
    - With a regular Deployment, pods get random names (db-7f9c8d-x2kpl) and random IPs that change on every restart — so other nodes lose track of who's who, replication config breaks, and there's no reliable way to say "connect to the primary" or "this is node 2 of 3."
    - This is exactly why Kubernetes has StatefulSets — they give pods stable, ordered names (db-0, db-1, db-2) that persist across restarts, plus stable network identity and dedicated storage per pod — solving exactly this problem.


---

## Create a Headless Service


1. Write a Service manifest with `clusterIP: None` — this is a Headless Service
2. Set the selector to match the labels you will use on your StatefulSet pods
3. Apply it and confirm CLUSTER-IP shows `None`

A Headless Service creates individual DNS entries for each pod instead of load-balancing to one IP. StatefulSets require this.

---
![](headless-service.png)
---
---

## Create a StatefulSet


1. Write a StatefulSet manifest with `serviceName` pointing to your Headless Service
2. Set replicas to 3, use the nginx image
3. Add a `volumeClaimTemplates` section requesting 100Mi of ReadWriteOnce storage
4. Apply and watch: `kubectl get pods -l <your-label> -w`

Observe ordered creation — `web-0` first, then `web-1` after `web-0` is Ready, then `web-2`.
---
![](statefulsets%20pods%20creation%20order.png)
---

Check the PVCs: `kubectl get pvc` — you should see `web-data-web-0`, `web-data-web-1`, `web-data-web-2` (names follow the pattern `<template-name>-<pod-name>`).
---
![](PVCs%20created%20for%20each%20pod.png)
---
---

## Stable Network Identity


Each StatefulSet pod gets a DNS name: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`

1. Run a temporary busybox pod and use `nslookup` to resolve `web-0.<your-headless-service>.default.svc.cluster.local`
2. Do the same for `web-1` and `web-2`
3. Confirm the IPs match `kubectl get pods -o wide`

---
![](each%20pod%20gets%20separate%20ip.png)
---
---

## Stable Storage — Data Survives Pod Deletion


1. Write unique data to each pod: `kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"`
2. Delete `web-0`: `kubectl delete pod web-0`
3. Wait for it to come back, then check the data — it should still be "Data from web-0"

The new pod reconnected to the same PVC.

---
![](Stable%20Storage%20of%20Pods.png)
---
---

## Ordered Scaling


1. Scale up to 5: `kubectl scale statefulset web --replicas=5` — pods create in order (web-3, then web-4)

2. Scale down to 3 — pods terminate in reverse order (web-4, then web-3)
---
![](Pods%20Scaling.png)
---

3. Check `kubectl get pvc` — all five PVCs still exist. Kubernetes keeps them on scale-down so data is preserved if you scale back up.

---
![](PVCs%20count%20after%20scale%20down.png)
---
---

## Clean Up


1. Delete the StatefulSet and the Headless Service
2. Check `kubectl get pvc` — PVCs are still there (safety feature)
3. Delete PVCs manually


#### *Were PVCs auto-deleted with the StatefulSet?*

No , becoz Kubernetes does not automatically delete PersistentVolumeClaims (PVCs) when we delete a StatefulSet by default. This design choice prioritizes data safety, ensuring that accidental workload deletions, scaling actions, or cluster re-creations do not permanently erase critical stateful data like databases


---

### *Points to Remember*

- `kubectl get sts` is the short name for StatefulSets
- `serviceName` must match an existing Headless Service
- Pod DNS: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
- PVC naming: `<template-name>-<statefulset-name>-<ordinal>`
- Pods create in order (0, 1, 2) and terminate in reverse order (2, 1, 0)
- Scaling down does not delete PVCs — data is preserved
- Deleting a StatefulSet does not delete PVCs — clean up separately
