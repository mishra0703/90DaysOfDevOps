# Day 60 – Capstone: Deploy WordPress + MySQL on Kubernetes

## Task
Ten days of Kubernetes — clusters, Pods, Deployments, Services, ConfigMaps, Secrets, storage, StatefulSets, resource management, autoscaling, and Helm. Today we will put it all together and Deploy a real WordPress + MySQL application using every major concept we have learned.

---

## Create the Namespace (Day 52)


1. Create a `capstone` namespace
2. Set it as your default: `kubectl config set-context --current --namespace=capstone`

---

## Deploy MySQL (Days 54-56)


1. Create a Secret with `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, and `MYSQL_PASSWORD` using `stringData`
2. Create a Headless Service (`clusterIP: None`) for MySQL on port 3306
3. Create a StatefulSet for MySQL with:
   - Image: `mysql:8.0`
   - `envFrom` referencing the Secret
   - Resource requests (cpu: 250m, memory: 512Mi) and limits (cpu: 500m, memory: 1Gi)
   - A `volumeClaimTemplates` section requesting 1Gi of storage, mounted at `/var/lib/mysql`
4. Verify MySQL works: `kubectl exec -it mysql-0 -- mysql -u <user> -p<password> -e "SHOW DATABASES;"`

---
![](mysql%20added.png)
---
---

## Deploy WordPress (Days 52, 54, 57)


1. Create a ConfigMap with `WORDPRESS_DB_HOST` set to `mysql-0.mysql.capstone.svc.cluster.local:3306` and `WORDPRESS_DB_NAME`
2. Create a Deployment with 2 replicas using `wordpress:latest` that:
   - Uses `envFrom` for the ConfigMap
   - Uses `secretKeyRef` for `WORDPRESS_DB_USER` and `WORDPRESS_DB_PASSWORD` from the MySQL Secret
   - Has resource requests and limits
   - Has a liveness probe and readiness probe on `/wp-login.php` port 80
3. Wait until both pods show `1/1 Running`

---
![](wordpress-app%20deployed.png)
---

## Expose WordPress (Day 53)


1. Create a NodePort Service on port 30080 targeting the WordPress pods
2. Access WordPress in your browser:
   - Minikube: `minikube service wordpress -n capstone`
   - Kind: `kubectl port-forward svc/wordpress 8080:80 -n capstone`
3. Complete the setup wizard and create a blog post


---
![](wordpress%20app%20is%20deployed.png)
---
![](We%20can%20use%20it%20without%20any%20issue.png)
---

## Test Self-Healing and Persistence


1. Delete a WordPress pod — watch the Deployment recreate it within seconds. Refresh the site.
2. Delete the MySQL pod: `kubectl delete pod mysql-0 -n capstone` — watch the StatefulSet recreate it
3. After MySQL recovers, refresh WordPress — your blog post should still be there


*Even after deleting both pods, our blog post is still there !*
---
![](Pods%20will%20be%20created%20automatically%20even%20after%20deleting.png)
---
![](Even%20after%20deleting%20mysql%20our%20pod%20get%20recreated%20autom.%20and%20our%20data%20is%20also%20safe.png)
---


## Set Up HPA (Day 58)


1. Write an HPA manifest targeting the WordPress Deployment with CPU at 50%, min 2, max 10 replicas
2. Apply and check: `kubectl get hpa -n capstone`
3. Run `kubectl get all -n capstone` for the complete picture


---
![](hpa%20implemented.png)
---
![](HPA%20Working%20Fine.png)
---
![](New%20Pods%20Created%20Automatically.png)
---

## (Bonus) Compare with Helm (Day 59)


1. Install WordPress using `helm install wp-helm bitnami/wordpress` in a separate namespace
2. Compare: how many resources did each approach create? Which gives more control?
3. Clean up the Helm deployment


---
![](get%20all%20-n%20capstone.png)
---
![](get%20all%20-n%20helm-wp.png)
---

## Clean Up and Reflect


1. Take a final look: `kubectl get all -n capstone`
2. Count the concepts you used: Namespace, Secret, ConfigMap, PVC, StatefulSet, Headless Service, Deployment, NodePort Service, Resource Limits, Probes, HPA, Helm — twelve concepts in one deployment
3. Delete the namespace: `kubectl delete namespace capstone`
4. Reset default: `kubectl config set-context --current --namespace=default`


---

## *Point to Remeber*

- If MySQL takes long to start, check: `kubectl logs mysql-0 -n capstone`
- `WORDPRESS_DB_HOST` must match the StatefulSet DNS pattern: `<pod>.<service>.<namespace>.svc.cluster.local`
- WordPress probes may fail initially — `initialDelaySeconds` gives it time to boot
- If PVC stays Pending, check `kubectl get storageclass`
- `nodePort` must be in range 30000-32767
- The Bitnami chart uses MariaDB instead of MySQL — compatible but not identical


---

## Concepts → Learning Day Map

| Concept | Day Learned | Used For |
|---|---|---|
| Namespace | Day 52 | Isolating all capstone resources |
| Secret | Days 54–56 | MySQL root/user credentials, WordPress DB credentials |
| Headless Service | Days 54–56 | Stable per-pod DNS for MySQL (`mysql-app-0.mysql-svc...`) |
| StatefulSet | Days 54–56 | Running MySQL with a stable identity and dedicated storage |
| PersistentVolumeClaim (`volumeClaimTemplates`) | Days 54–56 | Durable storage for `/var/lib/mysql` |
| ConfigMap | Days 52, 54, 57 | Non-sensitive WordPress config (`WORDPRESS_DB_HOST`, `WORDPRESS_DB_NAME`) |
| Deployment | Days 52, 54, 57 | Running 2 replicas of WordPress |
| Resource Requests/Limits | Days 54–56 | CPU/memory guarantees and caps for both tiers |
| Liveness/Readiness Probes | Days 52, 54, 57 | Health checks on `/wp-login.php` |
| NodePort Service | Day 53 | Exposing WordPress outside the cluster |
| HorizontalPodAutoscaler | Day 58 | Scaling WordPress 2→10 replicas on CPU |
| Helm | Day 59 | Comparing a packaged chart install against hand-written manifests |
