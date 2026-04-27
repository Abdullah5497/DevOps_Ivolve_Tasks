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


---

### Step 3: Verify Deployment

```bash
# Check pods status
kubectl get pods -w

# Describe pods if needed
kubectl describe pod <pod-name>
```


---

### Step 4: Verify All Resources

```bash
kubectl get all
kubectl get pvc
```

