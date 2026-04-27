# Lab 15: Node.js Application Deployment with Kubernetes

## Objective
Deploy a Node.js application on Kubernetes using:
- Deployment with 2 replicas
- PersistentVolumeClaim (PVC)
- ConfigMap and Secret
- ClusterIP Service
- Node.js server listening on 0.0.0.0 for Port-forward

---

## Environment

* **Kubernetes Cluster:** Minikube
* **Kubernetes Version:** v1.35.1
* **Container Runtime:** containerd
* **Docker Image:** abdullah911/kubernetes-app:lab9

---

## Steps

### Step 1: Kubernetes Resources

**ConfigMap**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nodejs-config
data:
  DB_HOST: mysql-clusterip
  DB_USER: root
```

**Secret**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nodejs-secret
type: Opaque
data:
  DB_PASSWORD: TXlTdHJvbmdQYXNzMTIz   # base64 of: MyStrongPass123
```

**PersistentVolumeClaim**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nodejs-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs
  template:
    metadata:
      labels:
        app: nodejs
    spec:
      tolerations:
        - key: node
          operator: Equal
          value: worker
          effect: NoSchedule
      containers:
        - name: nodejs
          image: abdullah911/kubernetes-app:lab9
          ports:
            - containerPort: 3000
          envFrom:
            - configMapRef:
                name: nodejs-config
            - secretRef:
                name: nodejs-secret
          volumeMounts:
            - name: nodejs-data
              mountPath: /usr/src/app/data
      volumes:
        - name: nodejs-data
          persistentVolumeClaim:
            claimName: nodejs-pvc
```

**ClusterIP Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
spec:
  type: ClusterIP
  selector:
    app: nodejs
  ports:
    - port: 80
      targetPort: 3000
```

---

### Step 2: Deploy to Kubernetes

```bash
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

![Apply Resources](Apply_Resources.png)

---

### Step 3: Verify Deployment

```bash
# Check pods status
kubectl get pods -w

# Describe pods if needed
kubectl describe pod <pod-name>
```

![Verify Pods](Verify_Pods.png)

---

### Step 4: Verify All Resources

```bash
kubectl get all
kubectl get pvc
```

Expected:
* Pod: `nodejs-app-xxx` → `Running`
* Deployment: `2/2`
* Service: `nodejs-service` → `ClusterIP`
* PVC: `nodejs-pvc` → `Bound`

![Verify All](Verify_All.png)

---

### Step 5: Check Application Logs

```bash
kubectl logs $(kubectl get pods | grep nodejs | awk '{print $1}' | head -1)
```

Expected:
```
✅ Connected to MySQL and 'ivolve' DB found.
Server started on http://0.0.0.0:3000
```

![App Logs](App_Logs.png)

---

### Step 6: Access the Application

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
