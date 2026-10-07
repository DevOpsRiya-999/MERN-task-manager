# Kubernetes Production-Style Additions

## New/updated components

- Resource requests and limits: frontend, backend, MongoDB, backup CronJob
- Liveness probes: frontend, backend, MongoDB
- Readiness probes: frontend, backend, MongoDB
- Backend health endpoints: `/healthz` and `/readyz`
- Backend HPA: 1–5 replicas
- Frontend HPA: 1–4 replicas
- MongoDB daily backup CronJob: 02:00 UTC
- Dedicated backup PV/PVC with 7-day local retention

## File order

```text
01-namespace.yml
02-frontend-deployment.yml
03-frontend-service.yml
04-mongodb.yml
05-mongodb-service.yml
06-backend-deployment.yml
07-backend-service.yml
08-mongodb-pv.yml
09-secrets.yml
10-configmap.yml
11-mongodb-backup-pv.yml
12-mongodb-backup-pvc.yml
13-mongodb-backup-cronjob.yml
14-backend-hpa.yml
15-frontend-hpa.yml
```

## Why these were added

### 1. Resource requests and limits
Requests tell Kubernetes the minimum CPU/memory needed for scheduling.
Limits stop a container from consuming unlimited resources.

### 2. Liveness probe
Answers: "Is this container alive?"
If it repeatedly fails, Kubernetes can restart the container.

### 3. Readiness probe
Answers: "Can this pod safely receive traffic?"
A pod that is not ready is removed from Service endpoints.

### 4. Backend `/healthz` and `/readyz`
`/healthz` checks the Node.js process.
`/readyz` checks that Mongoose is connected to MongoDB.
This avoids sending API traffic to a backend that cannot use its database.

### 5. HPA
HPA automatically changes pod count based on CPU/memory utilization.
Backend: 1–5 pods.
Frontend: 1–4 pods.

### 6. CronJob
Runs a MongoDB dump every day and stores it on a separate backup volume.
Backups older than 7 days are removed.

> Important: this is a practical EC2/local-storage backup for learning/demo purposes.
> For real production, store backups in durable external storage such as S3 and encrypt them.

## Prerequisite for HPA

Your cluster must have Metrics Server installed.

Check:

```bash
kubectl top nodes
kubectl top pods -n task-manager-ns
```

If these return CPU/memory metrics, HPA can calculate utilization.

## Apply everything

From the project root:

```bash
kubectl apply -f k8s/
```

Or apply in order:

```bash
kubectl apply -f k8s/01-namespace.yml
kubectl apply -f k8s/02-frontend-deployment.yml
kubectl apply -f k8s/03-frontend-service.yml
kubectl apply -f k8s/08-mongodb-pv.yml
kubectl apply -f k8s/04-mongodb.yml
kubectl apply -f k8s/05-mongodb-service.yml
kubectl apply -f k8s/09-secrets.yml
kubectl apply -f k8s/10-configmap.yml
kubectl apply -f k8s/06-backend-deployment.yml
kubectl apply -f k8s/07-backend-service.yml
kubectl apply -f k8s/11-mongodb-backup-pv.yml
kubectl apply -f k8s/12-mongodb-backup-pvc.yml
kubectl apply -f k8s/13-mongodb-backup-cronjob.yml
kubectl apply -f k8s/14-backend-hpa.yml
kubectl apply -f k8s/15-frontend-hpa.yml
```

## Verify

```bash
kubectl get pods -n task-manager-ns
kubectl get svc -n task-manager-ns
kubectl get pvc -n task-manager-ns
kubectl get hpa -n task-manager-ns
kubectl get cronjob -n task-manager-ns
kubectl get jobs -n task-manager-ns
```

Check probes:

```bash
kubectl describe pod -n task-manager-ns <backend-pod-name>
kubectl describe pod -n task-manager-ns <frontend-pod-name>
kubectl describe pod -n task-manager-ns <mongodb-pod-name>
```

Check HPA:

```bash
kubectl get hpa -n task-manager-ns
kubectl describe hpa backend-hpa -n task-manager-ns
kubectl describe hpa frontend-hpa -n task-manager-ns
```

Check backup:

```bash
kubectl get cronjob -n task-manager-ns
kubectl get jobs -n task-manager-ns
kubectl logs -n task-manager-ns job/<job-name>
```

To test the CronJob immediately:

```bash
kubectl create job --from=cronjob/mongodb-backup mongodb-backup-test -n task-manager-ns
kubectl get jobs -n task-manager-ns
kubectl logs -n task-manager-ns job/mongodb-backup-test
```

## Important MongoDB storage note

The existing MongoDB StatefulSet uses a `hostPath`-backed PV, so this setup is appropriate for a single-node EC2 learning environment. It is not highly available storage. For production, use a managed database or durable network storage/object storage for backups.

