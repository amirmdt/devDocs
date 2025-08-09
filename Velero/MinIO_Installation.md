## ✅ MinIO Installation on Kubernetes
To deploy MinIO in a Kubernetes cluster, we used Helm, a popular package manager for Kubernetes.
#### 🔍 Chart Source
The MinIO Helm chart was sourced from Artifact Hub. We searched for `minio` and selected the one provided by Bitnami, a trusted and well-maintained chart provider.
#### 📦 Installation Steps
The following steps were used to install MinIO in the `a-dehghani` namespace:
```
# Add the Bitnami Helm repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Install MinIO using a specific chart version
helm install my-minio bitnami/minio --version 17.0.16 -n a-dehghani
```
#### 📝 Notes
- Replace my-minio with a custom release name if needed.
- Make sure the namespace a-dehghani exists beforehand.
- The version 17.0.16 was used at the time of deployment, but you may use a newer version depending on your cluster compatibility or feature needs.