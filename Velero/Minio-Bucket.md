## 🪣 Create MinIO Bucket Internally for Velero to Use
To create the bucket that Velero will use for storing backups, follow these steps from inside the MinIO pod:
_________
#### 🔍 Step 1: Get the MinIO Pod Name
```
kubectl get pods -n a-dehghani | grep minio
```
This helps you identify the exact pod name of the MinIO deployment running in your namespace.
_________
#### 🐚 Step 2: Open a Shell Inside the MinIO Pod
```
kubectl exec -it -n a-dehghani minio-5dd7fc5848-q62n8 -- bash
```
Replace the pod name with the actual one from Step 1.
_________
#### ⚙️ Step 3: Run Commands Inside the Pod
```
mc alias set local http://127.0.0.1:9000 $MINIO_ROOT_USER $MINIO_ROOT_PASSWORD
mc mb local/velero
mc ls local
```
_________
### 💡 Explanation of the Commands
#### 🔧 `1. mc alias set local http://127.0.0.1:9000 $MINIO_ROOT_USER $MINIO_ROOT_PASSWORD`
**✅ What it does:**\
Creates a new alias named local to connect to your internal MinIO server at http://127.0.0.1:9000.
- Uses environment variables:\
  • `$MINIO_ROOT_USER`\
  • `$MINIO_ROOT_PASSWORD`

##### 💡 Tip:
If these environment variables are not set, you can manually use the credentials found in the `credentials-velero` file.
#### 2. `mc mb local/velero`
**✅ What it does:**\
Creates a new bucket named `velero` on the MinIO server defined by the alias `local`.
- `mb` = make bucket
- `local/velero` = "make a bucket called velero on the MinIO server named local"
##### 📁 Result:
A new bucket named `velero` is created — this will be used to store Velero backups.

#### 📁 3. mc ls local
**✅ What it does:**\
Lists all existing buckets on the MinIO server connected via alias `local`.\
🔍 This helps verify that the velero bucket was successfully created.



