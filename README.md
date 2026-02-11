# 🚀 Kubernetes Multi-Node Setup with AWS EFS (NFS Storage)

## 📌 Project Overview

This project demonstrates a complete Kubernetes multi-node setup on AWS EC2 with:

- 1 Master Node (Ubuntu 24.04)
- 1 Worker Node (Ubuntu 24.04)
- Nginx Deployment (3 replicas)
- HostPath Volume
- AWS EFS (NFS) Integration
- Volume Debugging
- Pod Troubleshooting

---

# 🏗️ Architecture

Master Node  → Controls Cluster  
Worker Node  → Runs Pods  
AWS EFS      → Shared Storage (NFS)  

---

# ☁️ EC2 Setup

## 🔹 Step 1: SSH into Master Node

```bash
ssh -i k8s.pem ubuntu@<MASTER_PUBLIC_IP>
sudo -i
cd /home/ubuntu
```

---

# 🔍 Step 2: Check Cluster Status

```bash
kubectl get pods
kubectl get all
```

If no resources:

```
No resources found in default namespace.
```

---

# 📦 Step 3: HostPath Deployment

## Create hostpath.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: mycontainer
        image: nginx
        volumeMounts:
        - name: myvolume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: myvolume
        hostPath:
          path: /home/ubuntu/bkp
          type: Directory
```

Apply Deployment:

```bash
kubectl apply -f hostpath.yml
```

Verify:

```bash
kubectl get pods
```

---

# 🐳 Step 4: Exec into Pod

```bash
kubectl get pods
kubectl exec -it <pod-name> -- /bin/bash
```

Test:

```bash
curl http://localhost
```

Expected Output:

```
<h1> Helllo from worker node </h1>
```

Exit:

```bash
exit
```

---

# 🗑️ Step 5: Delete Resources

```bash
kubectl delete all --all
```

---

# 🌐 Step 6: Setup AWS EFS on Worker Node

## SSH into Worker Node

```bash
ssh -i k8s.pem ubuntu@<WORKER_PUBLIC_IP>
```

---

## Install NFS Client

```bash
sudo apt update
sudo apt install nfs-common -y
```

---

## Create Mount Directory

```bash
mkdir efs
```

---

## Mount AWS EFS

```bash
sudo mount -t nfs4 \
-o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
fs-0535c15e60524bfcd.efs.us-west-1.amazonaws.com:/ \
/home/ubuntu/efs/
```

Verify:

```bash
du -sh
```

Create Folder in EFS:

```bash
sudo mkdir /home/ubuntu/efs/data
```

---

# 📁 Step 7: NFS Deployment (Master Node)

## Create nfs.yml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: mycontainer
        image: nginx
        volumeMounts:
        - name: myefsvolume
          mountPath: /usr/share/nginx/html
      volumes:
      - name: myefsvolume
        nfs:
          server: fs-0535c15e60524bfcd.efs.us-west-1.amazonaws.com
          path: /data
```

---

## Apply Deployment

```bash
kubectl apply -f nfs.yml
```

---

# 🔎 Verify NFS Mount

```bash
kubectl get all
kubectl describe pod <pod-name>
```

Look for:

```
Type: NFS
Server: fs-0535c15e60524bfcd.efs.us-west-1.amazonaws.com
Path: /data
```

---

# 🐞 Errors Faced & Fixes

## ❌ Error 1:

```
unknown field "hostpath"
```

✅ Fix:
Use correct field:

```
hostPath:
```

---

## ❌ Error 2:

```
volumeMounts[0].name: Not found
```

✅ Fix:
`volumeMounts.name` and `volumes.name` must match exactly.

---

## ❌ Exec Pod Error:

Wrong pod name used.

✅ Fix:
Use exact name from:

```bash
kubectl get pods
```

---

# 🧠 Skills Learned

- Kubernetes Deployment
- ReplicaSet
- Pod Exec
- HostPath Volume
- NFS Volume
- AWS EFS Integration
- YAML Debugging
- VolumeMount Matching
- Cluster Troubleshooting
- Resource Cleanup

---

# 🎯 Real DevOps Concepts Covered

- Storage in Kubernetes
- Persistent Volumes (Basic NFS)
- Multi-node Cluster
- Container Orchestration
- Cloud + Kubernetes Integration

---

# 🚀 Future Improvements

- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)
- StorageClass
- LoadBalancer Service
- Ingress Controller
- CI/CD Integration

---

# 👨‍💻 Author

Rohit Bhusare  
DevOps Learner | Kubernetes | AWS | Docker  

---

⭐ If you like this project, give it a star!
