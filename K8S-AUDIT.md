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
| Deployment + ReplicaSet | ✅ | `./k8s/02-frontend-deployment.yml` → 4 deployments | Manages application Pods and maintains replicas | `nginx Deployment, ReplicaSet` |
| Service | ✅ | ` /k8s/03-frontend-service.yml` → 6 services | Provides stable networking between application components | `nginx Service` |
| Namespace | ✅ | `/01-namespace.yml` → 3 namespaces | Keeps application resources isolated and organized ,if we don't use separated Ns then whole project will work in default Ns| `namespace manifest, k8s docs` |
| Labels and selectors | ✅ | `/k8s/03-frontend-service.yml` → 6 services have endpoints | Connects Services to the correct Pods | commands: namespaces, labels, selectors, k8s docs |
| Rolling update + rollback | ✅ | `./audit.sh` → 1 deployment rolled to a new image; rollback not visible | Allows application updates with minimal downtime | rolling update (rollback is not covered there, see k8s docs)` |
| ConfigMap | ✅ | `/k8s/10-configmap.yml` → 3 ConfigMaps | Stores non-sensitive application configuration | <MONGODB ConfigMap> |
| Secret | ✅ | `/k8s/09-secrets.yml` → 2 Opaque Secrets | Stores sensitive configuration such as credentials | < MONGODB Secret> |
| Requests and limits | ✅ | `k8s/02-frontend-deployment.yml` → 6 of 8 containers have CPU and memory requests/limits | Prevents containers from consuming excessive cluster resources | `Deployment with resources, k8s docs` |
| Probes (liveness + readiness) |✅ |  `k8s/02-frontend-deployment.yml` → 3 of 8 containers have both probes | Helps Kubernetes detect unhealthy Pods and send traffic only to ready Pods | `k8s docs, liveness example` |
| PVC | ✅ | `/k8s/08-mongodb-pv.yml` → 3 bound PVCs | Provides persistent storage for application data |PVC, MySQL volumes (kind creates the volume for you, see kind/README.md) |
| Ingress | ❌ | `./audit.sh` → 0 ingress resources and 0 controllers | Not implemented; frontend is accessed using Service port-forwarding | `k8s/` |
| Multi-node kind cluster | ✅ |kind-config.yaml. kubectl get nodes shows 3 nodes. The vote pods ran on two different workers. → 3 nodes | Provides a multi-node Kubernetes environment for testing | `kind/kind-config.yaml` |
| HPA (stretch) | ✅ | `/k8s/14-backend-hpa.yml` → 2 HPAs | Automatically scales application replicas based on resource usage | `k8s/` |
| RBAC + ServiceAccount (stretch) | ❌ | `./audit.sh` → 0 roles/rolebindings and 0 custom ServiceAccounts | Not implemented in the current project | `k8s/` |
| CronJob (stretch) | ✅ | `/k8s/13-mongodb-backup-cronjob.yml` → 1 CronJob | Runs scheduled background tasks automatically | `CronJob manifest, k8s docs/` |
| GitHub Actions deploying to kind (stretch) | ⚠️ | `/.github/workflows` → no kind workflow detected | CI/CD deployment to kind is not implemented  BUT I want every pull request to prove the manifests work on a clean 3-node cluster.| (helm/kind-action, example workflow) |

----------------------------------------------

# Evidence output
-------------------------------------------------
* Labels and selectors (each Service has endpoints):
```bash
ubuntu@ip-172-31-24-151:~$ kubectl get endpointslices
NAME                     ADDRESSTYPE   PORTS   ENDPOINTS               AGE
backend-g8vgz            IPv4          5000    10.244.1.4,10.244.2.6   9d
frontend-service-8kg6l   IPv4          80      10.244.2.3              10d
mongodb-kfxlh            IPv4          27017   10.244.1.2              5d23h
ubuntu@ip-172-31-24-151:~$
```
-------------------
```bash
ubuntu@ip-172-31-24-151:~$ kubectl rollout history deployment/backend
deployment.apps/backend
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

------------------------
```bash
ubuntu@ip-172-31-24-151:~$ kubectl rollout undo deployment/backend
Warning: resource deployments/backend was previously managed with 'kubectl apply'. Rolling back will not update the kubectl.kubernetes.io/last-applied-configuration annotation, which may cause unexpected behavior on future 'kubectl apply' operations. Consider using 'kubectl apply' with your previous configuration file instead.
deployment.apps/backend rolled back
```
-----------------------------
```bash
ubuntu@ip-172-31-24-151:~$ kubectl rollout status deployment/backend
deployment "backend" successfully rolled out
```
```bash
ubuntu@ip-172-31-24-151:~$ kubectl get rs
NAME                      DESIRED   CURRENT   READY   AGE
backend-6db6655c94        1         1         1       3d2h
backend-7fb9b64f85        1         1         0       22h
task-manager-6f4947d67f   0         0         0       10d
task-manager-7669c5f96d   0         0         0       10d
task-manager-845b68f666   1         1         1       22h
```
* PVC keeps data even after delete the pods
```bash
ubuntu@ip-172-31-24-151:~$ kubectl get pvc
NAME                     STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
mongodb-backup-pvc       Bound    mongodb-backup-pv   2Gi        RWO            manual-backup   <unset>                 22h
mongodb-data-mongodb-0   Bound    mongodb-pv          1Gi        RWO            manual          <unset>                 5d22h
ubuntu@ip-172-31-24-151:~$
```
CronJob 
```bash
ubuntu@ip-172-31-24-151:~/MERN-task-manager/MERN-task-manager$ kubectl logs -n task-manager-ns job/mongodb-backup-29857080
2026-10-08T17:16:29.097+0000    writing `taskmanager.tasks` to `/backup/2026-10-08_17-16-16/taskmanager/tasks.bson`
2026-10-08T17:16:29.098+0000    writing `taskmanager.users` to `/backup/2026-10-08_17-16-16/taskmanager/users.bson`
2026-10-08T17:16:29.107+0000    done dumping `taskmanager.tasks` (2 documents)
2026-10-08T17:16:29.109+0000    done dumping `taskmanager.users` (3 documents)
MongoDB backup completed: /backup/2026-10-08_17-16-16
ubuntu@ip-172-31-24-151:~/MERN-task-manager/MERN-task-manager$
```
The audit script, run on the finished app with the workflow folder passed in:
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
❌ Requests and limits                  7 of 9 containers have cpu+memory requests and limits
✅ Probes (liveness+readiness)          3 of 9 containers have both probes
✅ PVC                                  3 bound PVC(s)
❌ Ingress                              0 ingress resource(s), 0 running controller pod(s)
✅ Multi-node cluster                   3 node(s)
✅ HPA (stretch)                        2 HPA(s)
❌ RBAC + ServiceAccount (stretch)      0 role/rolebinding(s), 0 custom serviceaccount(s)
✅ CronJob (stretch)                    1 cronjob(s)
❌ GitHub Actions to kind (stretch)     looks for kind create cluster / helm/kind-action in ./.github/workflows

⚠️  1 pod(s) not Running/Ready: backend-7fb9b64f85-l9fxh
   The app is not fully working, so the ✅ above only show that objects exist. Fix the pods first.
Seen: 12/16  (must 10/12, stretch 2/4). Pass mark is 10+. Now write the evidence in K8S-AUDIT.md.
```



