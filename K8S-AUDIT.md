# K8S Audit

This is a finished audit for the MERN-task-manager. The manifests it points to are in MERN-task-manager/k8s.
```bash
ubuntu@ip-172-31-24-151:~/MERN-task-manager/MERN-task-manager$ ./audit.sh
context: kind-devboard   scope: all non-system namespaces

✅ Deployment + ReplicaSet              4 deployment(s) in app namespaces
✅ Service                              6 service(s)
✅ Namespace                            3 namespace(s) besides default and system ones
✅ Labels and selectors                 6 service(s) have endpoints, so selectors match pod labels
✅ Rolling update + rollback            1 deployment(s) rolled to a new image (rollback itself is not visible, show it in your evidence)
✅ ConfigMap                            3 non-default configmap(s)
✅ Secret                               2 Opaque secret(s)
❌ Requests and limits                  6 of 8 containers have cpu+memory requests and limits
✅ Probes (liveness+readiness)          3 of 8 containers have both probes
✅ PVC                                  3 bound PVC(s)
❌ Ingress                              0 ingress resource(s), 0 running controller pod(s)
✅ Multi-node cluster                   3 node(s)
✅ HPA (stretch)                        2 HPA(s)
❌ RBAC + ServiceAccount (stretch)      0 role/rolebinding(s), 0 custom serviceaccount(s)
✅ CronJob (stretch)                    1 cronjob(s)
❌ GitHub Actions to kind (stretch)     looks for kind create cluster / helm/kind-action in ./.github/workflows

⚠️  1 pod(s) not Running/Ready: backend-7fb9b64f85-zwtv5
   The app is not fully working, so the ✅ above only show that objects exist. Fix the pods first.
Seen: 12/16  (must 10/12, stretch 2/4). Pass mark is 10+. Now write the evidence in K8S-AUDIT.md.
```
------------------------------
# My app

* Name: MERN Task Manager

* Repo link: https://github.com/DevOpsRiya-999/MERN-task-manager

* Tiers:

- Frontend: React application served through Nginx/Docker

- API: Node.js + Express backend

- Database: MongoDB

* Kubernetes manifests are in: k8s/

# How to run it from a fresh machine:

```bash
# install KIND and metrics-server with the commands use official doc Kubernetes.io 

# 1. Clone the repository
git clone https://github.com/DevOpsRiya-999/MERN-task-manager.git
cd MERN-task-manager

# 2. Create the multi-node kind cluster
kind create cluster --name devboard --config kind/kind-config.yaml

# 3. Check the cluster
kubectl get nodes

# 4. Apply the Kubernetes manifests
kubectl apply -f k8s/

# 5. Check namespaces
kubectl get namespaces

# 6. Check all application resources
kubectl get all -n task-manager-ns

# 7. Check PVCs
kubectl get pvc -n task-manager-ns

# 8. Check HPA
kubectl get hpa -n task-manager-ns

# 9. Check CronJob
kubectl get cronjob -n task-manager-ns

# 10. Check pods
kubectl get pods -n task-manager-ns -o wide
```
----------------------------------------
# How to open it:

* The frontend is exposed through a Kubernetes ClusterIP service, so it can be accessed using port-forwarding:
```bash
kubectl port-forward svc/frontend 8081:80 -n task-manager-ns --address 0.0.0.0

```
* Then open:
```bash
http://<EC2-PUBLIC-IP>:3000

```
* Make sure port 8081 is allowed in the EC2 Security Group.

-------------------------------
# Audit

## Audit

| Concept | Status (✅ / ⚠️ / ❌) | Evidence | Why I used it in my app | Where to look |
|---|---|---|---|---|
| Deployment + ReplicaSet | ✅ | `./audit.sh` → 4 deployments | Manages application Pods and maintains replicas | `k8s/` |
| Service | ✅ | `./audit.sh` → 6 services | Provides stable networking between application components | `k8s/` |
| Namespace | ✅ | `./audit.sh` → 3 namespaces | Keeps application resources isolated and organized | `k8s/` |
| Labels and selectors | ✅ | `./audit.sh` → 6 services have endpoints | Connects Services to the correct Pods | `k8s/` |
| Rolling update + rollback | ⚠️ | `./audit.sh` → 1 deployment rolled to a new image; rollback not visible | Allows application updates with minimal downtime | `k8s/` |
| ConfigMap | ✅ | `./audit.sh` → 3 ConfigMaps | Stores non-sensitive application configuration | `k8s/` |
| Secret | ✅ | `./audit.sh` → 2 Opaque Secrets | Stores sensitive configuration such as credentials | `k8s/` |
| Requests and limits | ❌ | `./audit.sh` → 6 of 8 containers have CPU and memory requests/limits | Prevents containers from consuming excessive cluster resources | `k8s/` |
| Probes (liveness + readiness) | ⚠️ | `./audit.sh` → 3 of 8 containers have both probes | Helps Kubernetes detect unhealthy Pods and send traffic only to ready Pods | `k8s/` |
| PVC | ✅ | `./audit.sh` → 3 bound PVCs | Provides persistent storage for application data | `k8s/` |
| Ingress | ❌ | `./audit.sh` → 0 ingress resources and 0 controllers | Not implemented; frontend is accessed using Service port-forwarding | `k8s/` |
| Multi-node kind cluster | ✅ | `./audit.sh` → 3 nodes | Provides a multi-node Kubernetes environment for testing | `kind/kind-config.yaml` |
| HPA (stretch) | ✅ | `./audit.sh` → 2 HPAs | Automatically scales application replicas based on resource usage | `k8s/` |
| RBAC + ServiceAccount (stretch) | ❌ | `./audit.sh` → 0 roles/rolebindings and 0 custom ServiceAccounts | Not implemented in the current project | `k8s/` |
| CronJob (stretch) | ✅ | `./audit.sh` → 1 CronJob | Runs scheduled background tasks automatically | `k8s/` |
| GitHub Actions deploying to kind (stretch) | ❌ | `./audit.sh` → no kind workflow detected | CI/CD deployment to kind is not implemented | `.github/workflows/` |

----------------------------------------------

# Evidence output
-------------------------------------------------
* Labels and selectors (each Service has endpoints):

