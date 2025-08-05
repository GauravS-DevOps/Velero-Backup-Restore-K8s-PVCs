# Velero Backup and Restore (K8s + PVCs) Using AWS S3

## 📌 Overview

This project demonstrates how to perform a full backup and restore of Kubernetes workloads — including namespaces, pods, PVCs, and config — using **Velero** with **AWS S3** as the object storage backend.

The project covers:
- Backing up two namespaces (`ns1`, `ns2`) from a production cluster
- Storing the backup in an S3 bucket (`velero-bucket`)
- Restoring it into a local `minikube` cluster under new namespace names (`ns1-local`, `ns2-local`)

---

## ⚙️ Tools & Technologies

- Kubernetes (v1.32.5 on prod, Minikube locally)
- Velero v1.16.x
- AWS S3 (for backup storage)
- Ubuntu/Kali Linux for local machine

---

## 🧪 Step-by-Step Instructions

### 1️⃣ Configure AWS S3 Bucket

- Go to AWS Console and create a public/private bucket named:

```bash
velero-bucket
```

- Ensure the region is `us-east-1` or of your choice
- Create an IAM user or role with s3 read access and the following policies:
  - `AmazonS3FullAccess`

### 2️⃣ Prepare Velero Credentials

Create a file called `credentials-velero` with the following content:

```
[default]
aws_access_key_id = YOUR_AWS_ACCESS_KEY
aws_secret_access_key = YOUR_AWS_SECRET_KEY
```

### 3️⃣ Install Velero on Prod Cluster

Run this from the master node of your Kubernetes production cluster:

```bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.2 \
  --bucket velero-bucket \
  --secret-file ./credentials-velero \
  --backup-location-config region=us-east-1 \
  --namespace velero
```

### 4️⃣ Create the Backup

```bash
velero backup create namespace-full-backup \
  --include-namespaces ns1,ns2 \
  --wait
```

✅ **Status: Completed**

---

### 5️⃣ Install Velero on Local (Minikube)

```bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.9.2 \
  --bucket velero-bucket \
  --secret-file ./credentials-velero \
  --backup-location-config region=us-east-1 \
  --namespace velero
```

Confirm:
```bash
velero backup-location get
```

Output:
```
NAME      PROVIDER   BUCKET/PREFIX      PHASE       ACCESS MODE   DEFAULT
default   aws        velero-bucket   Available   ReadWrite     true
```

Then list all backups:
```bash
velero backup get
```

---

### 6️⃣ Perform Restore

Create new namespaces to map old ones:
- `ns1` ➝ `ns1-local`
- `ns2` ➝ `ns2-local`

Run restore:

```bash
velero restore create local-restore-ns1-ns2 \
  --from-backup namespace-full-backup \
  --namespace-mappings ns1:ns1-local,ns2:ns2-local \
  --wait
```

✅ **Restore Status: Completed**

---

## ✅ Validation

Verify:
```bash
kubectl get all -n ns1-local
kubectl get all -n ns2-local
```

Check PVCs, pods, deployments, services are successfully restored.

---

## 🧠 Architecture Diagram

```
+-------------------------+        S3 Bucket        +-------------------------+
|                         |   AWS S3 (us-east-1)   |                         |
|  Prod K8s Cluster       |----------------------->|   velero-bucket      |
|  Namespaces:            |   Velero Backup        |                         |
|   - ns1                 |                        +-------------------------+
|   - ns2           |
|                         |
+-----------+-------------+
            |
            v
+-----------+-------------+
| Local K8s (Minikube)    |
| Velero Restore          |
| Namespace Mapping:      |
|   ns1 ➝ ns1-local        |
|   ns2 ➝ ns2-local |
+-------------------------+
```

---

## 📁 Folder Structure

```
Velero-Backup-Restore-K8s-PVCs/
├── credentials-velero
├── README.md
└── (Backup and restore commands)
```

---

## 🏁 Outcome

- Successfully backed up and restored workloads from prod to local
- Used AWS S3 as the persistent, scalable backup backend
- Demonstrated namespace remapping and data integrity verification
