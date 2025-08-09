## ✅ Velero Client Installation on Local Machine
To interact with the Velero server running in Kubernetes, we installed the Velero CLI on the local machine (Linux amd64 architecture).
#### 📥 Download Velero CLI
The latest release was downloaded from the official GitHub releases page:\
👉 https://github.com/vmware-tanzu/velero/releases\
Choose the appropriate tarball for your system. In our case, it was:
```
velero-<VERSION>-linux-amd64.tar.gz
```
#### 📦 Installation Steps
```
# Extract the tarball
tar -xvf velero-<VERSION>-linux-amd64.tar.gz

# Change to the extracted directory
cd velero-<VERSION>-linux-amd64

# Ensure the binary is executable
chmod +x velero

# Move the binary to a directory in your system's PATH
sudo mv velero /usr/local/bin/
```
You can verify the installation using:
```
velero version
```
#### 📝 Notes
- Replace <VERSION> with the actual version you downloaded (e.g., v1.16.2).
- If you don’t have permission to move the binary to /usr/local/bin, either use sudo or add the directory to your PATH environment variable.

