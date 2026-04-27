# Lab 15: Node.js Application Deployment with ClusterIP Service

## Objective
The objective of this lab is to deploy a Node.js application on Kubernetes using a Deployment with 2 replicas, environment variables from ConfigMap and Secret, persistent storage via PVC, a toleration for the tainted worker node, and a ClusterIP service to expose the application internally.

---

## Environment

* **Kubernetes Cluster:** Minikube
* **Kubernetes Version:** v1.35.1
* **Container Runtime:** containerd
* **Docker Image:** abdullah911/kubernetes-app:lab9

---

## Resources Overview

| Resource | Name | Purpose |
|---|---|---|
| ConfigMap | nodejs-config | Stores DB_HOST and DB_USER |
| Secret | nodejs-secret | Stores DB_PASSWORD (base64 encoded) |
| PVC | nodejs-pvc | Persistent storage mounted into the app |
| Deployment | nodejs-app | Runs 2 replicas of the Node.js app |
| Service | nodejs-service | ClusterIP service on port 80 → 3000 |

---

## Steps

### Step 1: Verify Cluster and Taint

```bash
kubectl get nodes
kubectl describe node minikube-m02 | grep Taint
```

![Get Nodes](Get_Nodes.png)

---

### Step 2: Create ClusterIP Service for MySQL
A regular ClusterIP service is needed so the Node.js app can reach MySQL via DNS.

```bash
kubectl apply -f mysql-clusterip-service.yaml
kubectl get svc mysql-clusterip
```

![MySQL Service](MySQL_Service.png)

---

### Step 3: Create ConfigMap

```bash
vim configmap.yaml
kubectl apply -f configmap.yaml
kubectl get configmap nodejs-config
```

![ConfigMap](ConfigMap.png)

---

### Step 4: Create Secret

Encode the password first:

```bash
echo -n "MyStrongPass123" | base64
```

```bash
vim secret.yaml
kubectl apply -f secret.yaml
kubectl get secret nodejs-secret
```

![Secret](Secret.png)

---

### Step 5: Create PVC

```bash
vim pvc.yaml
kubectl apply -f pvc.yaml
kubectl get pvc nodejs-pvc
```

Expected status: `Bound`

![PVC](PVC.png)

---

### Step 6: Create Deployment

```bash
vim deployment.yaml
kubectl apply -f deployment.yaml
kubectl get pods -w
```

![Deployment](Deployment.png)

---

### Step 7: Create ClusterIP Service

```bash
vim service.yaml
kubectl apply -f service.yaml
kubectl get svc nodejs-service
```

![Service](Service.png)

---

### Step 8: Verify All Resources

```bash
kubectl get all
kubectl get pvc
```

Expected:
* Pods: `nodejs-app-xxx` → `Running`
* Deployment: `2/2`
* Service: `nodejs-service` → `ClusterIP`
* PVC: `nodejs-pvc` → `Bound`

![Verify All](Verify_All.png)

---

### Step 9: Check Application Logs

```bash
kubectl logs $(kubectl get pods | grep nodejs | awk '{print $1}' | head -1)
```

Expected: `✅ Connected to MySQL`

![App Logs](App_Logs.png)

---

### Step 10: Test the Application

```bash
kubectl port-forward svc/nodejs-service 8080:80
```

Open browser at `http://localhost:8080`

![App Running](App_Running.png)

---

## Push to GitHub

```bash
cd ~/DevOps_Ivolve_Tasks
git add K8s/Lab_15/
git commit -m "Add K8s Lab 15: Node.js Application Deployment with ClusterIP Service"
git pull origin main --rebase
git push origin main
```
