command to get clusters are installed:
```
kubectl config get-clsuters
```
## delete cluster
***on all nodes***
```
kubeadm reset -f
rm -rf .kube/
rm -rf /etc/cni/ /opt/cni/ /var/lib/cni/
rm -rf /var/lib/etcd/
#ip addr show
sudo ip link delete tunl0 || true
sudo ip link delete cali07a9f2e3b57 || true
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
#dnf list --installed | grep kube
dnf remove kubectl.x86_64 kubelet.x86_64 kubeadm.x86_64 kubernetes-cni.x86_64 cri-tools.x86_64
rm -f /usr/local/bin/kubelet /usr/local/bin/kubectl /usr/local/bin/kubeadm
rm -rf /etc/systemd/system/kubelet.service.d/
systemctl stop containerd.service
rm -rf /etc/systemd/system/containerd.service.d/
rm /usr/local/bin/containerd /usr/local/bin/containerd-stress /usr/local/bin/containerd-shim-runc-v2 /usr/local/bin/ctr -f
rm /usr/lib/systemd/system/containerd.service -f
rm /usr/local/sbin/runc -f
rm -f /usr/local/bin/crictl
rm -rf /var/lib/calico/
rm -rf /var/lib/containerd/
rm -rf /etc/cni/net.d/
systemctl daemon-reload
rm -rf /opt/containerd/
reboot
```
## install kubernetes with kubeadm and containerd


