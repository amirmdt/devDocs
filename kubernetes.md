## Install Kubernetes Cluster on 3 rockyLinux vms 1 for master and 2 for workers:
```
Master Node: 192.168.6.120 (master1)
Worker Nodes: 192.168.6.121 (worker1), 192.168.6.122 (worker2)
```
### 1. set hostnames
```
sudo hostnamectl set-hostname master1     # On master1
sudo hostnamectl set-hostname worker1     # On worker1
sudo hostnamectl set-hostname worker2     # On worker2
```
### 2. Configure `/etc/hosts` on All Nodes
```
192.168.6.120 master1
192.168.6.121 worker1
192.168.6.122 worker2
```
____________
**if you want to use proxy for your vms:**
### 3. Set Proxy (for root and environment) in all nodes
```
sudo tee /etc/profile.d/proxy.sh > /dev/null <<EOF
export http_proxy=http://192.168.6.14:20171
export https_proxy=http://192.168.6.14:20171
export no_proxy=localhost,127.0.0.1,192.168.6.0/24
EOF
source /etc/profile.d/proxy.sh
```
### 4. Disable Swap on all nodes
```
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```
### 5. Load Required Kernel Modules on all nodes
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
overlay
EOF

sudo modprobe br_netfilter
sudo modprobe overlay
```
### 6. Set Kernel Parameters on all nodes
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```
### 7. Open Firewall Ports (optional but recommended if firewalld is active)
#### on master node:
```
sudo firewall-cmd --permanent --add-port=6443/tcp    # Kubernetes API server
sudo firewall-cmd --permanent --add-port=2379-2380/tcp  # etcd
sudo firewall-cmd --permanent --add-port=10250/tcp   # kubelet
sudo firewall-cmd --permanent --add-port=10251/tcp   # kube-scheduler
sudo firewall-cmd --permanent --add-port=10252/tcp   # kube-controller-manager
sudo firewall-cmd --permanent --add-port=179/tcp     # Calico BGP (if enabled)
sudo firewall-cmd --reload
```
on worker nodes:
```
sudo firewall-cmd --permanent --add-port=10250/tcp  # kubelet
sudo firewall-cmd --permanent --add-port=30000-32767/tcp  # NodePort range
sudo firewall-cmd --permanent --add-port=179/tcp    # Calico BGP (if used)
sudo firewall-cmd --reload
```
***this step was for if you are using firewall***
### 8. Install containerd
Download the containerd-<VERSION>-<OS>-<ARCH>.tar.gz archive from https://github.com/containerd/containerd/releases :
```
$ tar Cxzvf /usr/local containerd-1.6.2-linux-amd64.tar.gz
bin/
bin/containerd-shim-runc-v2
bin/containerd-shim
bin/ctr
bin/containerd-shim-runc-v1
bin/containerd
bin/containerd-stress
```




