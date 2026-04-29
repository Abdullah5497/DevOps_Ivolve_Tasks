# Lab16 - Kubernetes Init Container for Pre-Deployment Database Setup

## 🎯 Objective

This lab demonstrates how to configure an Init Container inside a Kubernetes Deployment to perform database preparation before the application starts.

The initialization process includes:

- Creating the `ivolve` database  
- Creating the `ivolve_user` user  
- Granting privileges on the database  
- Using ConfigMap and Secret for database connection settings  

The goal is to ensure the application starts only after MySQL is ready and the database has been prepared.

---

## 📚 Concepts Covered

- Init Containers  
- ConfigMaps  
- Secrets  
- Persistent Volume Claims (PVC)  
- Kubernetes Deployment updates  
- MySQL client container  

---

## 📝 Lab Overview

Deployment flow:

### Step 1
Run the existing Node.js application deployment from the previous lab.

---

### Step 2
Before the application container starts:

The Init Container will:

- Connect to MySQL  
- Wait until MySQL is reachable  
- Create the `ivolve` database  
- Create `ivolve_user`  
- Grant privileges  

---

### Step 3
After initialization succeeds:

- Init Container exits successfully  
- Main Node.js container starts  

---

# 🏗 Components Used

## 1️⃣ Init Container

Container image:

```yaml
mysql:5.7
```

Purpose:

- Prepares the database before app startup  
- Uses MySQL client commands  
- Reads environment variables from:

  - ConfigMap  
  - Secret

---

## 2️⃣ Node.js Application Pod

Container image:

```yaml
fatmaahassan/kubernets-app:lab9
```

Uses:

- Existing Persistent Volume Claim:

```yaml
nodejs-pvc
```

Mounted path:

```bash
/usr/src/app/data
```

Starts only after Init Container finishes.

---

## 3️⃣ Persistent Volume Claim

PVC was created previously and reused in this lab.

---

# ⚙️ Steps Followed

## 1️⃣ Update Deployment with Init Container

Edit:

```bash
K8s/Lab16/deployment.yaml
```

Add:

- `initContainers` section  
- MySQL client image  
- DB initialization commands  
- Secret reference for root password  

Configuration includes:

- DB Host via `DB_HOST`  
- Root password from `mysql-secret`  
- SQL commands for database and user creation

---

## 2️⃣ Apply Updated Deployment

```bash
kubectl apply -f K8s/Lab16/deployment.yaml
```

---

## 3️⃣ Check Pod Status

```bash
kubectl get pods
```

Expected lifecycle:

```bash
Init:0/1
Init:Completed
Running
```

The main container should not start before initialization completes.

---

## 4️⃣ Inspect Init Container Logs

```bash
kubectl logs nodejs-app-86db786d44-g46jf -c init-mysql
```

Expected output:

```text
Waiting for MySQL...
Creating database and user...
```

---

## 5️⃣ Verify Database Manually

Connect to MySQL:

```bash
kubectl exec -it mysql-0 -- mysql -u root -p
```

Password:

```bash
MyStrongPass123
```

Check databases:

```sql
SHOW DATABASES;
```

Check privileges:

```sql
SHOW GRANTS FOR 'ivolve_user'@'%';
```

Confirm:

- `ivolve` exists  
- `ivolve_user` exists  
- User has expected privileges

---

## 6️⃣ Verify Node.js Application

Check pods:

```bash
kubectl get pods
```

Check deployment:

```bash
kubectl get deployment
```

Verify:

- Init completed  
- Pod is healthy  
- Deployment available

---

## 📂 Files Used

```text
K8s/Lab16/
├── deployment.yaml
├── configmap.yaml
├── secret.yaml
└── README.md
```

---

## ✅ Lab16 Complete

Completed successfully:

- Database created automatically  
- User created automatically  
- Privileges granted  
- Init Container completed  
- Node.js application started  
- Manual validation performed

---

## 🚀 Result

The Kubernetes deployment now performs automatic database initialization during startup using an Init Container.

This removes manual setup steps and ensures the application starts only when all prerequisites are ready.
