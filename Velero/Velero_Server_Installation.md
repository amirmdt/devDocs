## 🚀 Velero Server Installation on Kubernetes
To install the Velero server in Kubernetes, I used Helm.
#### 🔍 Step 1: Find the Helm Chart
I searched for velero on `ArtifactHub`, clicked on Velero Helm, and went to the Install section. There, I found a link to the Helm package:
#### 📦 Download Velero Helm Chart:
```
curl -LO https://github.com/vmware-tanzu/helm-charts/releases/download/velero-10.0.11/velero-10.0.11.tgz
```
#### ⚙️ Step 2: Prepare Custom Values File
I customized the `values.yaml` to use MinIO (which was previously installed in the `a-dehghani` namespace) as the backup storage provider.

**📄 custom-values.yaml**
```
configuration:
  backupStorageLocation:
  - bucket: velero
    name: velero
    provider: aws
    prefix: backups
    default: true
    config:
      region: minio
      s3ForcePathStyle: true
      s3Url: http://minio.a-dehghani.svc:9000
  volumeSnapshotLocation:
  - config:
      region: minio
      s3ForcePathStyle: true
      s3Url: http://minio.a-dehghani.svc:9000
    provider: aws

initContainers:
- name: velero-plugin-for-aws
  image: velero/velero-plugin-for-aws:v1.8.1
  volumeMounts:
  - mountPath: /target
    name: plugins

credentials:
  useSecret: true
  secretContents:
    cloud: ./credentials-velero

```
#### ✅ Adjustments made:
- Fixed YAML indentation.
- Set correct namespace for MinIO (`a-dehghani`).
- Ensured AWS plugin for MinIO is included.
#### 🔐 Step 3: Create MinIO Credentials File
**📄 credentials-velero**
```
[default]
aws_access_key_id = admin
aws_secret_access_key = BSl6nT8snd
```
#### 📦 Step 4: Extract and Prepare Helm Directory
```
tar -xvf velero-10.0.11.tgz
cp custom-values.yaml credentials-velero ./velero/
cd velero
```
#### 🚀 Step 5: Install Velero on Kubernetes
**📍 Namespace: `a-dehghani`**
```
helm install velero -n a-dehghani -f custom-values.yaml .
```
  🔐 Note: I didn’t have permission to install Helm charts, so I asked Pouya (admin) to install it for me.

  ✅ Result: Velero server is now successfully installed in the a-dehghani namespace and is configured to use MinIO as its backup storage provider.

