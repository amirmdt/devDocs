## Install openfaas on kubernetes

***creating two namespaces, one for the OpenFaaS core services and one for the functions.***
```
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```

***Add the OpenFaaS helm chart:***
```
helm repo add openfaas https://openfaas.github.io/faas-netes/
```
***Deploy CE from the helm chart repo directly:***
```
helm repo update \
 && helm upgrade openfaas \
  --install openfaas/openfaas \
  --namespace openfaas
```
***Install arkade:***
```
curl -sSL https://get.arkade.dev | sudo -E sh
```
***Install faas-cli:***
```
arkade get faas-cli
cp ~/.arkade/bin/faas-cli /usr/local/bin/faas-cli
```
***Retrieve the OpenFaaS credentials with:***
```
PASSWORD=$(kubectl -n openfaas get secret basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode) && \
echo "OpenFaaS admin password: $PASSWORD"
```
## Verify the installation
***Once all the services are up and running, log into your gateway using the OpenFaaS CLI. This will cache your credentials into your `~/.openfaas/config.yml` file.***\
***Fetch your public IP or NodePort via `kubectl get svc -n openfaas gateway-external -o wide` and set it as an environmental variable as below:***
```
export OPENFAAS_URL=http://192.168.163.132:31112
```
***Now log in with the CLI and check connectivity:***
```
echo -n $PASSWORD | faas-cli login -g $OPENFAAS_URL -u admin --password-stdin
```
***install faas-netes (controller)***\
***Clone the faas-netes repository***
```
git clone https://github.com/openfaas/faas-netes.git
cd faas-netes

```
***Render the chart to a Kubernetes manifest called `openfaas.yaml`****
```
helm template \
  openfaas chart/openfaas/ \
  --namespace openfaas \
  --set basic_auth=true \
  --set functionNamespace=openfaas-fn > openfaas.yaml
```
***Install the components using `kubectl`***
```
kubectl apply -f namespaces.yml,openfaas.yaml
```
## in master node install Docker
***how? : explained in Docker Docs***\
***why? : faas-cli build cannot use crictl. It specifically expects Docker or a Docker-compatible engine (like podman) because it needs the full Docker build process (Dockerfile, image tagging, etc.).***\
***crictl is a low-level container runtime CLI for Kubernetes (like kubectl is for the API), and it does not support building images from a Dockerfile.***

## run docker registry container in master node
***how? : explained in docker docs***\
***why? : we need it to push images into it and workers can pull the images***

## give access to master node (docker) to registry:
***add below to `/etc/docker/daemon.json`***
```
{
  "insecure-registries": ["192.168.163.132:5000"]
}
```
## give access to master node (containerd) to registry:
```
sudo mkdir -p /etc/containerd/certs.d/192.168.163.132:5000
sudo nano /etc/containerd/certs.d/192.168.163.132:5000/hosts.toml
```
***Paste the following:***
```
server = "http://192.168.163.132:5000"

[host."http://192.168.163.132:5000"]
  capabilities = ["pull", "resolve"]
  skip_verify = true
```
***Edit `/etc/containerd/config.toml` to use config_path***
```
[plugins."io.containerd.grpc.v1.cri".registry]
  config_path = "/etc/containerd/certs.d"
```
***sudo systemctl restart containerd***
```
sudo systemctl restart containerd
```
***now we can pull images from the docker registry using ctr into workers***
```
ctr image pull --plain-http 192.168.163.132:5000/pytdict:latest
```

## How OpenFaaS Uses Docker Images
*Each function you build with faas-cli gets turned into a Docker image (or OCI container image), which is then pushed to a container registry (like Docker Hub, GitHub Container Registry, or a private registry), and deployed by OpenFaaS.*

## Image Naming Pattern
*You define the image name in your `stack.yaml` under the function like this:*
```
functions:
  myfunction:
    lang: python3-flask
    handler: ./myfunction
    image: your-registry/your-image-name:tag
```
*Example for a local private registry at `192.168.163.132:5000`:*
```
    image: 192.168.163.132:5000/myfunction:latest
```
## What Images to Use for Different Languages?
*You don't choose the image manually — `faas-cli build` creates it from the template you selected with `faas-cli new`. But you do specify the image name in `stack.yaml`.*

## Quick Checklist for Your Own Function
*Let’s say you wrote a Python function:*
```
faas-cli new myadder --lang python3-http
```
*Then in `stack.yaml`:*
```
functions:
  myadder:
    lang: python3-http
    handler: ./myadder
    image: 192.168.163.132:5000/myadder:latest
```
*Now build and push:*
```
faas-cli build -f stack.yaml
faas-cli push -f stack.yaml
faas-cli deploy -f stack.yaml
```

## example : create a function which get a string and uppercase it:
*first see the list of templates we can use:*
```
faas-cli template store list
```
*then download a template (in my case i download the python3-flask):*
```
faas-cli template store pull python3-flask
```
*then start to create a new function using `python3-flask` template:*
```
faas-cli new uppercase --lang python3-flask
```
*then edit the file `stack.yml`:(in my case my master node ip is 192.168.163.132 and im using docker registry in the master node)*
```
version: 1.0
provider:
  name: openfaas
  gateway: http://192.168.163.132:31112
functions:
  uppercase:
    lang: python3-flask
    handler: ./uppercase
    image: 192.168.163.132:5000/uppercase:latest
```
*then edit the file `handler.py` in `uppercase` directory:*
```
def handle(req):
    x = req.upper()

    return x
```
*then use commands below to `build` image , `push` the image to registry and `deploy` the function:*
```
faas-cli build -f stack.yaml
faas-cli push -f stack.yaml
faas-cli deploy -f stack.yaml
```
*then use the command below to test it:*
```
curl -X POST http://192.168.163.132:31112/function/uppercase -d "hello world"
```
*or you can use the web ui to test it*


__________________________________

## Lens

### install lens:
*in my case i have openfaas installed on kubernetes cluster on a ubuntu vm. but i want to install `lens` on my host which is windows11. so i downloaded the lens install `.exe` file from https://k8slens.dev/  then i installed the `lens` on my windows 11.*\
*to access the kubernetes on my vm from this lens i copied the `~/.kube/config` file in the vm to `C:\Users\<YourWindowsUser>\.kube\config`*\
*for that first create a `.kube` directory in path `C:\Users\<YourWindowsUser>\` if it is not exist. then use command below to copy the file:*
```
# in my case
scp root@192.168.163.132:/root/.kube/config C:\Users\Amir\.kube\config
```
*and at the end i edited the file config and changed the:*
```
server: https://127.0.0.1:6443
```
*to:*
```
server: https://192.168.163.132:6443
```
***if i want to be honest, i didnt change anything in the file config, because it was like:***
```
server: https://192.168.163.132:6443
```
***from the begining*** :|

*and now the `lens` should auto-detect the kubeconfig file, if not, you can add the cluster manually.*

### see how long it takes to invoke a function:
* first invoke a function in the openfaas web ui
* Go to Lens → Workloads → Pods → openfaas-fn namespace.
* Click your function pod.
* Click "Logs".
_____________________________________
_____________________________________
_____________________________________

OpenFaaS is a serverless framework that runs on Kubernetes. It uses a Gateway to expose a REST API for deploying and invoking functions. When you deploy a function, a controller (faas-netes) creates Kubernetes resources for it. Invoking a function sends a request through the gateway to the function pod, and metrics can be collected using Prometheus. Async invocations use a queue-worker component for background processing.









