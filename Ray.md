## KubeRay Operator Installation
```
helm repo add kuberay https://ray-project.github.io/kuberay-helm/
helm repo update
# Install both CRDs and KubeRay operator v1.3.0.
helm install kuberay-operator kuberay/kuberay-operator --version 1.3.0
```
## Validate Installation
```
kubectl get pods
```

# RayCluster Quickstart

*first i tried install raycluster using ray docs:*
```
helm install raycluster kuberay/ray-cluster --version 1.3.0
```
but using this command the pods couldnt be `running` and the raycluster couldnt be `available`\
Reason:\
***Your Ray pods are not starting because:***
1. *Insufficient memory and CPU on 2 nodes.*
2. *Untolerated taint on the control-plane node (master), which prevents it from running regular workloads.*

so i used the following process:

### step 0 : Allow master node to run pods
```
kubectl get nodes  # Find the name of your master node
kubectl taint nodes <master-node-name> node-role.kubernetes.io/control-plane-
```
### Step 1: Create a lightweight RayCluster YAML
***Save the following as `ray-cluster.yaml`:***
```
apiVersion: ray.io/v1
kind: RayCluster
metadata:
  name: raycluster-sample
spec:
  rayVersion: '2.9.3'
  headGroupSpec:
    serviceType: ClusterIP
    rayStartParams:
      dashboard-host: '0.0.0.0'
    template:
      spec:
        containers:
        - name: ray-head
          image: rayproject/ray:2.9.3
          ports:
          - containerPort: 8265  # Dashboard
          - containerPort: 6379  # Redis
          - containerPort: 10001 # Ray client
          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "200m"
              memory: "512Mi"
  workerGroupSpecs:
  - replicas: 1
    minReplicas: 1
    maxReplicas: 2
    groupName: worker-group
    rayStartParams: {}
    template:
      spec:
        containers:
        - name: ray-worker
          image: rayproject/ray:2.9.3
          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "200m"
              memory: "512Mi"
```
### Step 2: Apply the RayCluster YAML
```
kubectl apply -f ray-cluster.yaml
```
### Step 3: Monitor the Pods
```
kubectl get pods
```
*You should see:*\
*1 raycluster-sample-head pod*
*1 raycluster-sample-worker pod*
*Both should become Running.*

## use nodeport instead of clusterIP for `raycluster-sample-head-svc`
to do that i changed the `ray-cluster.yaml` to below:
```
  headGroupSpec:
    serviceType: NodePort
    rayStartParams:
      dashboard-host: '0.0.0.0'
    template:
      spec:
        containers:
        - name: ray-head
          image: rayproject/ray:2.9.3
          ports:
          - name: dashboard
            containerPort: 8265
          - name: redis
            containerPort: 6379
          - name: client
            containerPort: 10001
          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "200m"
              memory: "512Mi"
```
and then i used command `kubectl get raycluster` to see the name of previous `raycluster` and delete that using:
```
kubectl delete raycluster <raycluster-name>
```
and after that, i applied the `ray-cluster.yaml` file again.
```
kubectl apply -f ray-cluster.yaml
```
then the service is like (`kubectl get svc`):
```
raycluster-sample-head-svc   NodePort    10.96.110.100   <none>        10001:31018/TCP,8265:31042/TCP,8080:31723/TCP,6379:32715/TCP   27m
```
so:
|🔧 Service|🌐 NodePort|📋 Purpose|🌍 Access URL (example)|
|-----------|-----------|-----------|-----------------------|
|`raycluster-sample-head-svc`|`31042`|Ray Dashboard|`http://<node-ip>:31042`|
||`31018`|Ray Client|Used by Python clients|
||`32715`|Redis (Ray internal)|Internal only|
||`31723`|Metrics (Prometheus)|Optional|

so i could see the ray dashboard using link below in web browser:
`192.168.163.132:31042`



___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________
___________________________

## install raycluster again (this time on workers):

### restore the original taint on your control-plane node (k8s.master.amir):
*so that regular pods won't run there anymore. This is common when you want to go back to a production-like setup.*
```
kubectl taint nodes k8s.master.amir node-role.kubernetes.io/control-plane=:NoSchedule
```
***Verify:***
```
kubectl describe node k8s.master.amir | grep Taint
```
*you should see:*
```
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```
### increase the worker nodes memory:

as we said before:\
***Your Ray pods are not starting because:***
1. *Insufficient memory and CPU on 2 nodes.*
2. *Untolerated taint on the control-plane node (master), which prevents it from running regular workloads.*

so i did shutdown all nodes and increase their memory. then started them again.

then i uninstalled the `raycluster` and `kuberay-operator` which installed before.
```
helm uninstall raycluster
helm uninstall kuberay-operator
```
### install kuberay operator
```
helm repo add kuberay https://ray-project.github.io/kuberay-helm/
helm repo update
# Install both CRDs and KubeRay operator v1.3.0.
helm install kuberay-operator kuberay/kuberay-operator --version 1.3.0
```
_______________________
_______________________
**Tip:** *first time i tried to install `kuberay operator` i got an error:*
```
Container kuberay-operator failed liveness probe, will be restarted
```
*i restarted the nodes again and reinstalled it and it was ok this time*
_______________________
_______________________
verify:
```
kubectl get pods
```
you should see something like:
```
NAME                                READY   STATUS    RESTARTS   AGE
kuberay-operator-6bc45dd644-gwtqv   1/1     Running   0          24s
```
and if you use command:
```
kubectl get crd | grep ray
```
you should see something like:
```
rayclusters.ray.io                                      2025-05-27T06:28:12Z
rayjobs.ray.io                                          2025-05-27T06:28:12Z
rayservices.ray.io                                      2025-05-27T06:28:14Z
```

### install RayCluster

Once the KubeRay operator is running, you’re ready to deploy a RayCluster. Create a RayCluster Custom Resource (CR) in the default namespace.
```
helm install raycluster kuberay/ray-cluster --version 1.3.0
```
after using this code i saw that the `raycluster-kuberay-head-svc` service is a `ClusterIP` service. i wanted to change it to `NodePort` service. so i did :
```
helm template raycluster kuberay/ray-cluster --version 1.3.0 > raycluster-rendered.yaml
```
then edited the `raycluster-rendered.yaml` file and changed:
```
spec:
  headGroupSpec:
    serviceType: ClusterIP
```
to:
```
spec:
  headGroupSpec:
    serviceType: NodePort
```
and added:
```
ports:
- name: dashboard
  containerPort: 8265
- name: redis
  containerPort: 6379
- name: client
  containerPort: 10001
```
to:
```
        containers:
          -
            volumeMounts:
            - mountPath: /tmp/ray
              name: log-volume
            name: ray-head
            image: rayproject/ray:2.41.0
            ports:
            - name: dashboard
              containerPort: 8265
            - name: redis
              containerPort: 6379
            - name: client
              containerPort: 10001
            imagePullPolicy: IfNotPresent
```
then i deleted the raycluster and applied this file again:
```
kubectl delete raycluster raycluster-kuberay
kubectl apply -f raycluster-rendered.yaml
```
### Install Ray on Your Local Machine
Make sure you have Python 3 and then:
```
pip install "ray[default]" --upgrade
```
### Write a Simple Ray Script
```
import ray
import time

# Connect to Ray head node's NodePort service (adjust IP if needed)
ray.init(address="ray://192.168.163.131:30394")  # Example IP of k8s.worker2.amir

@ray.remote
def square(x):
    time.sleep(1)
    return x * x

start = time.time()
futures = [square.remote(i) for i in range(8)]
results = ray.get(futures)

print("Results:", results)
print("Time taken:", time.time() - start)
```
the `192.168.163.131:30394` is :
* `192.168.163.131` : ip address of the node which head pod installed on it (`raycluster-kuberay-head-cczqf`)
* `30394` : the ray client port `raycluster-kuberay-head-svc   NodePort    10.97.138.50    <none>        10001:30394/TCP,8265:30102/TCP,8080:31887/TCP,6379:31443/TCP   20h`

### Then run the script
```
python3 ray_test.py
```
### Error occured im my case:
```
RuntimeError: Version mismatch: The cluster was started with:
    Ray: 2.41.0
    Python: 3.9.21
This process on Ray Client was started with:
    Ray: 2.46.0
    Python: 3.10.12
```
it means im running this Python code on my local machine with:
```
Ray: 2.46.0
Python: 3.10.12
```
But my Ray cluster (inside the Kubernetes Ray pods) is running with:
```
Ray: 2.41.0
Python: 3.9.21
```
to check versions use commands below in local machine and in head pod (`kubectl exec -it <head-pod-name> -- bash`):
```
ray --version
python3 --version
```

### ***fix this error***:
### Downgrade Ray on your local machine:
To match the version of your Kubernetes Ray cluster (Ray 2.41.0), run this on your local machine:
```
pip uninstall ray -y
pip install "ray[default]==2.41.0"
```

### install Python 3.9.21 on your local machine:
Add the deadsnakes PPA
```
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
```
Install Python 3.9 and venv:
```
sudo apt install -y python3.9 python3.9-venv python3.9-dev
```
Verify Python 3.9 installation:
```
python3.9 --version
```
Create a Python 3.9 virtual environment:
```
python3.9 -m venv ~/ray_py39_env
```
Activate the virtual environment:
```
source ~/ray_py39_env/bin/activate
```
Upgrade pip and install Ray 2.41.0 inside the virtualenv:
```
pip install --upgrade pip
pip install ray==2.41.0
```
Run your Ray script with the virtualenv Python:
```
python ray_test.py
```
### Why use a virtual environment?
* Keeps your system Python 3.10 intact.

* Isolates Ray 2.41.0 and Python 3.9 dependencies.

* Prevents version conflicts.

### another error:
```
ValueError: Ray Client requires pip package ray[client]. If you installed the minimal Ray (e.g. pip install ray), please reinstall by executing pip install ray[client].
```
so i did below:
```
pip uninstall ray -y
pip install ray[client]==2.41.0
```
_____________________________________
_____________________________________
_____________________________________
### see cpu and memory usage of each pods in `Ray Dashboard`

open `Ray Dashboard`(in my case `http://192.168.163.132:30102`) , in `cluster` tab:\
in here you will see a list of pods, includes `head` and `workers`. when you run an application the `cpu` and `memory` usage will change tangibly.

_____________________________________
_____________________________________
_____________________________________

✏️ What is Ray?\
  Ray is an open-source framework for building and running distributed applications. It enables Python code to scale across multiple cores and machines without requiring the developer to manage threads or networking. It supports parallel execution using simple decorators and can be used for machine learning, data processing, and general distributed computing tasks.

✏️ How was Ray deployed on Kubernetes?\
  Ray was deployed on a local Kubernetes cluster using the KubeRay Operator. The operator was installed via Helm, and then a RayCluster resource was created, which launched a head node and one or more worker nodes. NodePort was used to expose the Ray Dashboard and client ports to allow access from outside the cluster.

✏️ How does the Python script work?\
  The script connects to the Ray cluster using the Ray client and defines a simple task (e.g., computing square numbers) using the @ray.remote decorator. It runs the task in parallel using ray.get() and ray.remote() calls. The script demonstrates distributed execution by splitting the computation across multiple Ray workers and collecting the results.

_______________________________
🔹 What Is a "Distributed Application"?\
A distributed application is a program that runs on multiple computers (nodes) at the same time and works together to complete a task.\
Instead of doing all the work on one computer, it distributes the work across several.
_______________________________
🔹 What Does "Scale Across Cores and Machines" Mean?\
Let’s say you have a task: process 1,000 images.\
Case	Description\
🖥️ 1 machine, 1 core	It processes images one by one = slow\
🖥️ 1 machine, 4 cores	It processes 4 at a time = 4x faster\
🖥️🖥️ 3 machines, 4 cores each	12 at a time = 12x faster\

So "scale across cores" = use all CPU cores on one machine.\
"Scale across machines" = use many machines together.

Ray handles all the hard parts of splitting, sending, and collecting tasks from all machines and cores, so you can write Python code like normal, but it runs everywhere in parallel.
________________________________
🔹 Why Use Distributed Apps?\
✅ Speed up big tasks (data processing, AI training, simulations)\
✅ Work with large datasets that don’t fit on one machine\
✅ Use resources more efficiently (like cloud VMs or your full cluster)
________________________________
🔹 Who Does the Work?\
✅ By default, Ray schedules tasks on any available node — head or workers.\
✅ So yes, `raycluster-kuberay-head-cczqf` can also process tasks.\
✅ But most of the heavy lifting is typically done by worker nodes, especially when you scale out.
________________________________
Ray Architecture – Quick Overview\
Ray has a cluster-based architecture designed for distributed computing. It consists of:\

🔹 1. Head Node\
Coordinates the cluster.

Runs:

raylet (resource manager & scheduler)

Dashboard (e.g., on port 8265)

GCS (Global Control Store) for metadata/state

🔹 2. Worker Nodes\
Run Ray workers that execute tasks and actors.

Communicate with the head node for scheduling and task coordination.

🔹 3. Ray Core Components\
Tasks – stateless functions distributed across nodes

Actors – stateful workers that run on a node and hold memory/state

Object Store – in-memory shared data (plasma store) between processes

🔹 4. Scheduler\
Distributes tasks intelligently across the cluster.

Supports task retrying, placement group constraints, etc.











