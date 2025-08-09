# ☁️ Kubernetes Velero Backup & Restore with MinIO

This repository provides a step-by-step guide on how to install and configure Velero for Kubernetes backups using MinIO as the object storage provider.

It includes clear instructions for setting up each component, client installation, and a real-world NGINX backup/restore scenario using the `a-dehghani` namespace.

---

## 📚 Sections Overview

### 1. 🗃️ MinIO Installation on Kubernetes
Install MinIO using Helm as the S3-compatible storage backend for Velero.

📄 [MinIO_Installation](./MinIO_Installation.md)

---

### 2. 🖥️ Velero Client Installation (Local Machine)
Install the Velero CLI client on your local machine for interacting with the Velero server in your Kubernetes cluster.

📄 [Velero_Client_Installation](./Velero_Client_Installation.md)

---

### 3. ☁️ Velero Installation on Kubernetes
Install the Velero server component in your cluster using Helm, and configure it to use the MinIO backend.

📄 [Velero_Server_Installation](./Velero_Server_Installation.md)

---

### 4. 📦 Creating the MinIO Bucket for Velero
After installing MinIO, you need to create the bucket (e.g., `velero`) that Velero will use to store its backups. This is done from inside the MinIO pod using `mc` (MinIO client) commands.

📄 [Minio-Bucket](./Minio-Bucket.md)

---

### 5. 🧪 Backup & Restore Scenario: NGINX Pod
A full walkthrough demonstrating Velero in action — backing up, deleting, and restoring a test NGINX pod in the `a-dehghani` namespace.

📄 [Pod_Backup_Scenario](./Pod_Backup_Scenario.md)

---

## ✅ Prerequisites
Before you begin, ensure the following:
- A working Kubernetes cluster.
- Helm installed and configured.
- MinIO installed as S3 storage (see section 1).
- Velero CLI installed on your local machine (see section 2).

---

## 📦 Backup Strategy in Action
Velero enables:
- Scheduled or on-demand backups.
- Namespace- or resource-specific backup granularity.
- Disaster recovery via restore functionality.
- Easy integration with S3-compatible storage systems.

---
