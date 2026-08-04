# Day 59 – Helm — Kubernetes Package Manager

## Task
Over the past eight days we have written Deployments, Services, ConfigMaps, Secrets, PVCs, and more — all as individual YAML files. For a real application we might have dozens of these. Helm is the package manager for Kubernetes, like apt for Ubuntu. Today we will install charts, customize them, and create our own.

---

## Install Helm


1. Install Helm (brew, curl script, or chocolatey depending on your OS)
2. Verify with `helm version` and `helm env`

Three core concepts:
- **Chart** — a package of Kubernetes manifest templates
- **Release** — a specific installation of a chart in your cluster
- **Repository** — a collection of charts (like a package repo)

---
![](helm%20installed%20succesfully.png)
---

## Add a Repository and Search


1. Add the Bitnami repository: `helm repo add bitnami https://charts.bitnami.com/bitnami`
2. Update: `helm repo update`
3. Search: `helm search repo nginx` and `helm search repo bitnami`

---
![](helm%20repo%20and%20search.png)
---

## Install a Chart


1. Deploy nginx: `helm install my-nginx bitnami/nginx`
2. Check what was created: `kubectl get all`
3. Inspect the release: `helm list`, `helm status my-nginx`, `helm get manifest my-nginx`


One command replaced writing a Deployment, Service, and ConfigMap by hand.

---
![](installing%20helm%20chart.png)
---
![](list%20of%20services%20ran%20by%20one%20command.png)
---
![](status%20of%20nginx%20helm%20app.png)
---

## Customize with Values


1. View defaults: `helm show values bitnami/nginx`

2. Install a custom release with `--set replicaCount=3 --set service.type=NodePort`
---
![](installing%20a%20release%20with%20custom%20values%20as%20args.png)
---
![](passed%20values%20as%20args%20worked.png)
---

3. Create a `custom-values.yaml` file with replicaCount, service type, and resource limits
4. Install another release using `-f custom-values.yaml`
5. Check overrides: `helm get values <release-name>`
---
![](creating%20and%20verifying%20custom%20value%20.png)
---



---

## Upgrade and Rollback


1. Upgrade: `helm upgrade my-nginx bitnami/nginx --set replicaCount=5`
---
![](helm%20upgrade%20release%20command.png)
---
![](release%20upgraded%20succesfully.png)
---


2. Check history: `helm history my-nginx`
3. Rollback: `helm rollback my-nginx 1`
---
![](helm%20rollaback%20command.png)
---

4. Check history again — rollback creates a new revision (3), not overwriting revision 2
---
![](rollback%20done%20succesfully.png)
---


**Note :** Same concept as Deployment rollouts from Day 52, but at the full stack level.

---

## Create Your Own Chart


1. Scaffold: `helm create my-app`
2. Explore the directory: `Chart.yaml`, `values.yaml`, `templates/deployment.yaml`
---
![](exploring%20our%20own%20chart%20directory.png)
---


3. Look at the Go template syntax in templates: `{{ .Values.replicaCount }}`, `{{ .Chart.Name }}`
---
![](deployment%20manifest%20file%20template.png)
---


4. Edit `values.yaml` — set replicaCount to 3 and image to nginx:1.25
5. Validate: `helm lint my-app`
---
![](helm%20lint.png)
---


6. Preview: `helm template my-release ./my-app`
---
![](template%20of%20our%20own%20chart.png)
---

7. Install: `helm install my-release ./my-app`
---
![](Installing%20custom%20app%20via%20our%20own%20chart.png)
---

8. Upgrade: `helm upgrade my-release ./my-app --set replicaCount=5`
---
![](helm%20upgrading%20our%20own%20chart.png)
---
![](Own%20chart%20upgraded%20succesfully.png)
---


---


## *Points to Remember*

- `helm show values <chart>` — see what you can customize
- `--set key=value` for single overrides, `-f values.yaml` for multiple overrides using custom-value file
- Nested values use dots: `--set service.type=NodePort`
- `helm get values <release>` shows overrides, `--all` for everything
- `helm template` renders without installing — great for debugging
- `helm lint` validates chart structure before installing
- Use `--keep-history` if you want to retain release history for auditing
    - `helm uninstall <RELEASE_NAME> --keep-history -n <NAMESPACE>`

---
