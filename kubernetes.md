
## Preparing and Mounting Dedicated Disk for containerd Storage on Worker Nodes
On both worker nodes,
### 1. Confirm the `vdb` device
run:
```
lsblk
```
Make sure you see something like:
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda      252:0    0   40G  0 disk
├─vda1   252:1    0    1G  0 part /boot
└─vda2   252:2    0   39G  0 part /
vdb      252:16   0   20G  0 disk
```
`vdb` should have no partitions and no mount point.

### 2. Format the disk (`vdb`)
Format it with ext4 (or xfs if you prefer):
```
sudo mkfs.ext4 /dev/vdb
```
### 3. Create a mount point:
Containerd uses /var/lib/containerd by default.
```
mkdir -p /var/lib/containerd
```
Mount it:
```
mount /dev/vdb /var/lib/containerd
```
Make the mount permanent:
Find the disk UUID:
```
blkid /dev/vdb
```
Edit `/etc/fstab`:
```
vim /etc/fstab
```
Add this line (replace UUID=... with your actual UUID):
```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /var/lib/containerd  ext4  defaults  0 2
systemctl daemon-reload
```
Test mount:
```
sudo umount /var/lib/containerd
sudo mount -a
```
_______________________
_______________________
_______________________
_______________________
_______________________
# Install Kubernetes

```
Master Node: 192.168.6.120 (master1)
Worker Nodes: 192.168.6.121 (worker1), 192.168.6.122 (worker2)
```
set hostnames
```
sudo hostnamectl set-hostname master1     # On master1
sudo hostnamectl set-hostname worker1     # On worker1
sudo hostnamectl set-hostname worker2     # On worker2
```
Configure `/etc/hosts` on All Nodes
```
192.168.6.120 master1
192.168.6.121 worker1
192.168.6.122 worker2
```
disable firewall, or:
Port|Protocol|Protocol|Purpose
---|---|---|---
6443|TCP|Master|Kubernetes API server
2379-2380|TCP|Master|etcd server client API
10250|TCP|All Nodes|Kubelet API
10251|TCP|Master|kube-scheduler
10252|TCP|Master|kube-controller-manager
10255|TCP|All (optional)|Read-only Kubelet API
30000-32767|TCP|Workers|NodePort Services
179|TCP|All (if Calico)|BGP (Calico networking)
If you're in a secure environment and want to enable firewalld later, create firewall rules for the above ports and test node-to-node and pod-to-pod communication.

### 1. Preflight Setup (All Nodes)
```
# Disable SELinux
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config

# Disable swap
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
overlay
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set sysctl params
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```
_____________________
### 2. Install containerd (All Nodes)
1. Download the `containerd-<VERSION>-<OS>-<ARCH>.tar.gz` archive from https://github.com/containerd/containerd/releases , verify its sha256sum, and extract it under `/usr/local`:
```
tar Cxzvf /usr/local containerd-1.6.2-linux-amd64.tar.gz
```
download the containerd.service unit file from https://raw.githubusercontent.com/containerd/containerd/main/containerd.service into `/usr/local/lib/systemd/system/containerd.service`, and run the following commands:
```
systemctl daemon-reload
systemctl enable --now containerd
```
2. Installing runc: Download the `runc.<ARCH>` binary from https://github.com/opencontainers/runc/releases , verify its sha256sum, and install it as `/usr/local/sbin/runc`.
```
install -m 755 runc.amd64 /usr/local/sbin/runc
```
3. Installing CNI plugins: Download the `cni-plugins-<OS>-<ARCH>-<VERSION>.tgz` archive from https://github.com/containernetworking/plugins/releases , verify its sha256sum, and extract it under `/opt/cni/bin`:
```
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```
4. install crictl: `crictl` can be downloaded from cri-tools https://github.com/kubernetes-sigs/cri-tools/releases:
```
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
```
change `SystemdCgroup = false` to `SystemdCgroup = true` in `/etc/containerd/config.toml`, if you don't have it add below:
```
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      runtime_type = "io.containerd.runc.v2"

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true
```
under:
```
[plugins.'io.containerd.grpc.v1.cri']
```
section.

_________________
i added all `/usr/local/bin`, `/usr/local/sbin` and `/opt/cni/bin` to $PATH. to do that:\
```
vim ~/.bashrc
```
and add below lines to this file:
```
export PATH="/usr/local/bin:$PATH"
export PATH="/usr/local/sbin:$PATH"
export PATH="/opt/cni/bin:$PATH"
```
then save the file and use command below:
```
source ~/.bashrc
```
_________________
### 3. Install Kubernetes Components (All Nodes)
1. Add the Kubernetes yum repository
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```
2. Install kubelet, kubeadm and kubectl:
```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
***Remember:*** *you have to add proxy for dnf to install these packages using yum*\
to do that:\
add below line to file `/etc/dnf/dnf.conf`:
```
proxy=http://<proxy-server-ip>:<proxy-server-port>
```
______________
i downloaded the `kubectl`, `kubelet` and `kubeadm` binaries using:
```
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubelet"
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubeadm"
```
commands and `chmod +x` them, and moved them to `/usr/local/bin` directory, because i wanted to check if the ones are in `/usr/bin` directory are the same version as these ones.
______________
3. Enable the kubelet service before running kubeadm:
```
systemctl enable --now kubelet
```
_________________
### 4. Initialize Kubernetes Master
**First**: Pull Images on Master:\
add proxy for containerd to pull images, to do that:
```
mkdir -p /etc/systemd/system/containerd.service.d
cat <<EOF | sudo tee /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://<proxy-server-ip>:<proxy-server-port"
Environment="HTTPS_PROXY=http://<proxy-server-ip>:<proxy-server-port"
Environment="NO_PROXY=127.0.0.1,localhost,192.168.6.0/24"
EOF

sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl restart containerd
```
then run command below to pull the images:
```
kubeadm config images pull
```
after all images are successfully pulled, remove the containerd proxy:
```
rm -f /etc/systemd/system/containerd.service.d/http-proxy.conf
systemctl daemon-reexec
systemctl daemon-reload
systemctl restart containerd
```
and now Initialize Kubernetes Master:
```
kubeadm init --node-name=$(hostname)
```
After successful init, first copy the join command somewhere then set up kubectl:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
______________________
### 5. Install CNI Plugin (e.g., Calico)
because we dont want to use proxy while installing Calico:\
first go to `github.com`, search for `calico`, (https://github.com/projectcalico/calico) , select the latest branch (not master), got to `manifests` directory, download the `calico.yaml` file, edit this file and change images names from `docker.io/imageName` to `docker.arvancloud.ir/imageName`\
for example:
```
docker.io/calico/cni:v3.30.2
```
to:
```
docker.arvancloud.ir/calico/cni:v3.30.2
```
then install Calico using command below:
```
kubectl apply -f calico.yaml
```
________________________
### 6. Join Worker Nodes
because while joining, the worker nodes will try to pull some images, so we should add proxy for containerd in worker nodes,then used the join command we copied before:
```
kubeadm join 192.168.6.120:6443 --token pyc7jt.4z2uw2pj0gg89blc \
        --discovery-token-ca-cert-hash sha256:cd0563ed500e386c04442a77ebdef7ba2b16b327915f567b5aabf1902241a4ca
```
after it pulled the images, because they will try to pull some images refer to calico from `arvancloud` i removed the proxy again.\
___________________
***Not Tested:*** *if you lost the join command use command below:*
```
kubeadm token create --print-join-command
```
___________________
### add nexus repository to containerd config file to pull images from it:
for example the nexus repository ip and port is `192.168.6.14:8668`
```
mkdir -p /etc/containerd/certs.d/192.168.6.14:8668
vim /etc/containerd/certs.d/192.168.6.14\:8668/hosts.toml
```
and paste below in it:
```
server = "https://192.168.6.14:8668"

[host."https://192.168.6.14:8668"]
  capabilities = ["pull", "resolve"]
  skip_verify = true
```
the `skip_verify = true` part is to avoid cert errors.
then you should add the `/etc/containerd/certs.d` path in the `/etc/containerd/config.toml` file like below:
```
    [plugins.'io.containerd.cri.v1.images'.registry]
      config_path = ''
```
to:
```
    [plugins.'io.containerd.cri.v1.images'.registry]
      config_path = '/etc/containerd/certs.d'
```

_________________________

## Install helm
1. Download your desired version from https://github.com/helm/helm/releases
2. Unpack it (tar -zxvf helm-version-linux-amd64.tar.gz)
3. Find the helm binary in the unpacked directory, and move it to its desired destination (`mv linux-amd64/helm /usr/local/bin/helm`)

_________________________









