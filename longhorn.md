## Install Longhorn on kubernetes cluster
### Installing open-iscsiV on all Nodes
```
yum --setopt=tsflags=noscripts install iscsi-initiator-utils
echo "InitiatorName=$(/sbin/iscsi-iname)" > /etc/iscsi/initiatorname.iscsi
systemctl enable iscsid
systemctl start iscsid
```
Please ensure iscsi_tcp module has been loaded before iscsid service starts. Generally, it should be automatically loaded along with the package installation.
```
modprobe iscsi_tcp
```
### Installing NFSv4 client on all Nodes
Check `NFSv4.1` support is enabled in kernel
```
cat /boot/config-`uname -r`| grep CONFIG_NFS_V4_1
```
Check `NFSv4.2` support is enabled in kernel
```
cat /boot/config-`uname -r`| grep CONFIG_NFS_V4_2
```
now install:
```
yum install nfs-utils
```
### Installing Cryptsetup and LUKS on all Nodes
```
yum install cryptsetup
```
### Installing Device Mapper Userspace Tool
```
yum install device-mapper
```
### create the `/var/lib/longhorn` directory
```
mkdir -p /var/lib/longhorn
```

### `bash`, `curl`, `findmnt`, `grep`, `awk`, `blkid`, `lsblk` must be installed.

these where **Installation Requirements**

### add nexus repository to containerd for all nodes:
```
mkdir -p /etc/containerd/certs.d/192.168.6.14:8668
vim /etc/containerd/certs.d/192.168.6.14\:8668/hosts.toml
```
and add below lines into the `hosts.toml` file:
```
server = "https://192.168.6.14:8668"

[host."https://192.168.6.14:8668"]
  capabilities = ["pull", "resolve"]
  skip_verify = true
```
which `192.168.6.14:8668` is the pull address of nexus repository.\
then edit the `/etc/containerd/config.toml` file and add the `/etc/containerd/certs.d` path into this file like below:
```
    [plugins.'io.containerd.cri.v1.images'.registry]
      config_path = '/etc/containerd/certs.d'
```
then run commands below:
```
systemctl daemon-reload
systemctl restart containerd
```
### get and edit the `values-longhorn.yaml` file
go to artifachub website, search for longhorn, select the main one (https://artifacthub.io/packages/helm/longhorn/longhorn):\
in CHART VERSIONS click on the version which we have images on the nexus repository:(v1.8.1)\
click on `DEFAULT VALUES` and download the file:\
in this file some images are named which when we install longhorn using helm and this values file, they are going to be pulled. we edit their name and add the address of our repository and path to these images in our registry in the start of the image names. for example:
```
repository: longhornio/longhorn-engine
```
change to:
```
repository: 192.168.6.14:8668/cluster-images/docker.io/longhornio/longhorn-engine
```
then save the file in the master node.

now in nexus repository find the `longhorn-version.tgz (longhorn-1.8.1.tgz)` , click on it , right click on its Path on the right side menu, Copy link address, and download it in the master node using command below:
```
curl -O https://192.168.6.14:8448/repository/cluster-files/kubernetes/charts/longhorn-1.8.1.tgz -k
```
then using command below install longhorn on your cluster:
```
helm install longhorn -n longhorn-system --create-namespace longhorn-1.8.1.tgz -f values-longhorn.yaml
```
now check the pods in longhorn-system namespace, see if all are running completely and correctly.

_____________________________

## Deploy NGINX Pod with Longhorn Volume and Access it Externally
1. create a `pvc.yaml` like below (for example):
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
   name: pvc-test
spec:
   storageClassName: "longhorn"
   accessModes:
   - ReadWriteMany
   resources:
      requests:
         storage: 1Gi
```
then apply this file:
```
kubectl apply -f pvc.yaml
```
this command will create a pvc in kubernetes, you can see this using command:
```
kubectl get pvc
```
then create a pod manifest to create a nginx pod which this pvc is mounted:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-longhorn
  labels:
    app: nginx-longhorn
spec:
  volumes:
    - name: web-content
      persistentVolumeClaim:
        claimName: pvc-test
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: web-content
          mountPath: /usr/share/nginx/html
```
and save this file with name like `nginx.yaml`.\
now using command below apply this pod manifest and create the nginx pod:
```
kubectl apply -f nginx.yaml
```
after the pod is created completely and it is running (`kubectl get po`), exec into this pod:
```
kubectl exec -it nginx-longhorn -- bash
```
and inside this pod go to path `/usr/share/nginx/html` and create a file named `index.html` and write something in it like `hello welcome`.\
then exit the pod. and create a service manifest to create a Node Port service, like below:
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-longhorn
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

and save it to file nginx-service.yaml and apply this manifest:
```
kubectl apply -f nginx-service.yaml
```
now in the browser you can access to `192.168.6.122:30080` (`192.168.6.122` is the worker node which nginx is installed on it) and see the `hello welcome`

now you can create another pod using another manifest like below and mount the same pvc to it:
```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  volumes:
    - name: web-content1
      persistentVolumeClaim:
        claimName: pvc-test
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: web-content1
          mountPath: /content
```
if you apply this pod manifest and exec into it and go to path `/content` you can see the index.html file.(if the pod has no `/content` path on its own, the path will be created) and if you change it you can see the changes in web and in other nginx pod.






