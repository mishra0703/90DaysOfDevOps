# Day 54 – Kubernetes ConfigMaps and Secrets

## Task
Our application needs configuration — database URLs, feature flags, API keys. Hardcoding these into container images means rebuilding every time a value changes. Kubernetes solves this with ConfigMaps for non-sensitive config and Secrets for sensitive data.

---

## Create a ConfigMap from Literals


1. Use `kubectl create configmap` with `--from-literal` to create a ConfigMap called `app-config` with keys `APP_ENV=production`, `APP_DEBUG=false`, and `APP_PORT=8080`
2. Inspect it with `kubectl describe configmap app-config` and `kubectl get configmap app-config -o yaml`
3. Notice the data is stored as plain text — no encoding, no encryption

---
![](creating%20configmap%20from%20literal.png)
---
---

## Create a ConfigMap from a File

    
1. Write a custom Nginx config file that adds a `/health` endpoint returning "healthy"
2. Create a ConfigMap from this file using `kubectl create configmap nginx-config --from-file=default.conf=<your-file>`
3. The key name (`default.conf`) becomes the filename when mounted into a Pod


---
![](creating%20configmap%20from%20file.png)
---
---

## Use ConfigMaps in a Pod


1. Write a Pod manifest that uses `envFrom` with `configMapRef` to inject all keys from `app-config` as environment variables. Use a busybox container that prints the values.
---
![](printing%20configmap%20values%20in%20a%20container.png)
---

2. Write a second Pod manifest that mounts `nginx-config` as a volume at `/etc/nginx/conf.d`. Use the nginx image.
---
![]()
---

3. Test that the mounted config works: `kubectl exec <pod> -- curl -s http://localhost/health`
---
![](hit%20healthy%20endpoint.png)
---
---

## Create a Secret


1. Use `kubectl create secret generic db-credentials` with `--from-literal` to store `DB_USER=admin` and `DB_PASSWORD=s3cureP@ssw0rd`

2. Inspect with `kubectl get secret db-credentials -o yaml` — the values are base64-encoded
---
![](creating%20secrets%20from%20literal.png)
---

3. Decode a value: `echo '<base64-value>' | base64 --decode`
---
![](decoding%20a%20base64%20string.png)
---

**base64 is encoding, not encryption.** Anyone with cluster access can decode Secrets. The real advantages are RBAC separation, tmpfs storage on nodes, and optional encryption at rest.

---

## Use Secrets in a Pod


1. Write a Pod manifest that injects `DB_USER` as an environment variable using `secretKeyRef`
2. In the same Pod, mount the entire `db-credentials` Secret as a volume at `/etc/db-credentials` with `readOnly: true`
3. Verify: each Secret key becomes a file, and the content is the decoded plaintext value


---
![](using%20secrets%20in%20a%20pod.png)
---
---

## Update a ConfigMap and Observe Propagation


1. Create a ConfigMap `live-config` with a key `message=hello`
2. Write a Pod that mounts this ConfigMap as a volume and reads the file in a loop every 5 seconds
3. Update the ConfigMap: `kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}'`
4. Wait 30-60 seconds — the volume-mounted value updates automatically
5. Environment variables from earlier tasks do NOT update — they are set at pod startup only


*What Happened to Us*
- We were not able to access the logs of this pod , couldn't figured out why 

- So , we directly printed the configmap value in exec mode in pod 
    - `kubectl exec print-config-in-loop -- cat /etc/live-config/message`
    - Output : `hello` 
        ![](configmap%20variable%20in%20pod.png)

- Then we tried to ran the patch command to update the value of `message` , but in powershell we were getting error in passing json format value , so we did pass it in git bash terminal and the value got updated finally
    
    ---
    ![](configmap%20variable%20updated%20outside%20pod.png)

- Then after updating the value using patch command ,after some seconds we again printed the configmap value in exec mode in pod , and the value got updated
    
    ---
    ![](configmap%20variable%20update%20inside%20pod.png)

- This is why , Using volume mounts for Kubernetes ConfigMaps and Secrets is significantly better than environment (env) variables due to live dynamic updates, superior data security, and proper file-structure support 

---



## *Points to Remember*

- `--from-literal=KEY=VALUE` for command-line values, `--from-file=key=filename` for file contents
- `envFrom` injects all keys; `env` with `valueFrom` injects individual keys
- `echo -n 'value' | base64` — always use `-n` to avoid encoding a trailing newline
- Volume-mounted ConfigMaps/Secrets auto-update; environment variables do not
- `kubectl get secret <name> -o jsonpath='{.data.KEY}' | base64 --decode` extracts and decodes a value
- Use environment variables for simple key-value settings. Use volume mounts for full config files.

