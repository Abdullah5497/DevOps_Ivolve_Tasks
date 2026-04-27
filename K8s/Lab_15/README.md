# Lab 15: Node.js Application Deployment with ClusterIP Service

## Objective
The objective of this lab is to deploy a Node.js application on Kubernetes using a Deployment with 2 replicas, environment variables from ConfigMap and Secret, persistent storage via PVC, a toleration for the tainted worker node, and a ClusterIP service to expose the application internally.







### Step 1: Create ClusterIP Service for MySQL
A regular ClusterIP service is needed so the Node.js app can reach MySQL via DNS.

```bash
kubectl apply -f mysql-clusterip-service.yaml
kubectl get svc mysql-clusterip
```



---

### Step 2: Create ConfigMap

```bash
vim configmap.yaml
kubectl apply -f configmap.yaml
kubectl get configmap nodejs-config
```


---

### Step 3: Create Secret

Encode the password first:

```bash
echo -n "MyStrongPass123" | base64
```

```bash
vim secret.yaml
kubectl apply -f secret.yaml
kubectl get secret nodejs-secret
```


---

### Step 4: Create PVC

```bash
vim pvc.yaml
kubectl apply -f pvc.yaml
kubectl get pvc nodejs-pvc
```

Expected status: `Bound`


---

### Step 5: Create Deployment

```bash
vim deployment.yaml
kubectl apply -f deployment.yaml
kubectl get pods -w
```


---

### Step 6: Create ClusterIP Service

```bash
vim service.yaml
kubectl apply -f service.yaml
kubectl get svc nodejs-service
```


---

### Step 7: Verify All Resources

```bash
kubectl get all
kubectl get pvc
```





