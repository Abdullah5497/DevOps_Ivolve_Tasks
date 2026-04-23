# Lab 13: Persistent Storage Setup for Application Logging

## Objective
The objective of this lab is to set up persistent storage for application logs using Persistent Volumes (PV) and Persistent Volume Claims (PVC) in Kubernetes.


## Check That the Cluster is Running

```bash
kubectl get nodes
```

![GetNodes](GetNodes.png)

---

## Steps

### Step 1: Prepare Node Directory
SSH into the worker node and create the directory that will be used as the hostPath for the PV:


```bash
sudo mkdir -p /mnt/app-logs
sudo chmod 777 /mnt/app-logs
ls -ld /mnt/app-logs
```

Expected output:
```
drwxrwxrwx 2 root root 4096 ... /mnt/app-logs
```

![Dir](Dir.png)


### Step 2: Create Persistent Volume (PV)

```bash
vim pv.yaml
```

Content of `pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-logs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/app-logs
```

Apply the PV:

```bash
kubectl apply -f pv.yaml
```

Verify the PV:

```bash
kubectl get pv
kubectl describe pv app-logs-pv
```

Expected status: `Available`

![PV](PV.png)

---

### Step 3: Create Persistent Volume Claim (PVC)

```bash
vim pvc.yaml
```

Content of `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-logs-pvc
  namespace: ivolve
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

Apply the PVC:

```bash
kubectl apply -f pvc.yaml
```

Verify the PVC:

```bash
kubectl get pvc -n ivolve
kubectl describe pvc app-logs-pvc -n ivolve
```

Expected status: `Bound`

![PVC](PVC.png)

---

### Final Verification
Confirm the PV and PVC are bound to each other:

```bash
kubectl get pv
kubectl get pvc -n ivolve
```

Expected output:
```
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  
app-logs-pv    1Gi        RWX            Retain           Bound    ivolve/app-logs-pvc    
```

```
NAME            STATUS   VOLUME         CAPACITY   ACCESS MODES   
app-logs-pvc    Bound    app-logs-pv    1Gi        RWX            
```

![Final](Final.png)


