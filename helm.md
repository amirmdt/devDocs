# on an ubuntu server:
## 1. install docker
## 2. install kubectl
## 3. install kind
```
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```
## 4. Creating a Cluster
```
kind create cluster --name helm
```
(you can pull the desired version of kindest/node image before creating cluster then create the cluster but you should mention the version like: `kind create cluster --name helm --image=...`)
## 5. install helm 
go to helm release page https://github.com/helm/helm/releases and copy the URL of the version you need. then:
```
curl -O https://get.helm.sh/helm-v3.18.1-linux-amd64.tar.gz
tar -xvf helm-v3.18.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```
_______________________________
_______________________________
_______________________________
_______________________________
_______________________________
## create a new CAHRT:
```
helm create <CHART-NAME>
```
this will create a directory with name like your `chart`.\
in this directory there is a directory named "chart" which allow us to embed more charts that our chart may depend on (nest charts).\
there is another directory named `templates` which all the `yaml` files we need to bundle up to form a chart, are in it.\
there is a file named `Chart.yaml` which is a yaml file containing information about the chart.\
and another file named `values.yaml` which consist of default configuration values for this chart. the values file is where we can inject the parameters. it makes out chart generic. we can create more values files, like one value file per application.

now you can copy your `yaml` files to `template` directory. like `deployment.yaml`, `configmap.yaml`, `secret.yaml`, `service.yaml` , etc.\
and for now erase everything in `values.yaml` file.\

### Test the rendring of our template (to make sure it's ok):
```
helm template <chart-name> <chart-folder-name>
```
### Install our app using our Chart:
```
helm install <app-name> <chart-folder-name>
```
### list out apps:
```
helm list
```
### you can see what resources installed on kubernetes using helm bu kubectl, im my cluster:
```
kubectl get all
```
i got:
```
NAME                                  READY   STATUS    RESTARTS   AGE
pod/example-deploy-664d9cf56c-9gtpr   1/1     Running   0          11m
pod/example-deploy-664d9cf56c-w4vsh   1/1     Running   0          11m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/example-service   ClusterIP   10.96.102.101   <none>        80/TCP    11m
service/kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP   87m

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/example-deploy   2/2     2            2           11m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/example-deploy-664d9cf56c   2         2         2       11m
```
and:
```
kubectl get cm
```
i got:
```
NAME               DATA   AGE
example-config     1      13m
kube-root-ca.crt   1      90m
```
and:
```
kubectl get secret
```
i got:
```
NAME                                   TYPE                 DATA   AGE
mysecret                               Opaque               2      14m
sh.helm.release.v1.example-app-01.v1   helm.sh/release.v1   1      14m
```
__________________________
__________________________
__________________________
__________________________
__________________________
## values file (parameter injection):
allows us to inject values to our chart and make it reuseable.
for example in `values.yaml` file we write:
```
deployment:
  image: "aimvector/python"
  tag: "1.0.4"
```
and to inject that into the chart, in `deployment.yaml` file, i replace the:
```
image: aimvector/python:1.0.4
```
with:
```
image: {{ .Values.deployment.image }}:{{ .Values.deployment.tag }}
```
to upgrade the app using the values, use command below:
```
helm upgrade example-app-01 /root/helm/temp/example-app --values /root/helm/temp/example-app/values.yaml
```
which:
`example-app-01` is the app name
`/root/helm/temp/example-app` is the chart directory
`/root/helm/temp/example-app/values.yaml` is the values file path

and if you use `helm list` command you can see that the app `REVISION` changed to `2`

or you can use `--set` flage instead of `--values` to inject the parameters, like:
```
helm upgrade example-app-01 /root/helm/temp/example-app --set deployment.tag=1.0.4
```

## helm rollout new pods when configmap changes
by default in kubernetes when configmap changes, pods are not restarted automatically, and some applications might need a restart to pick up new config files.\
to do so, we need add below to `deployment.yaml` file:
```
annotations:
  checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```
under:
```
spec:
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```
and to trigger that change:
```
helm upgrade <app-name> <app-directory> --values <app-values-file>
```


## Control Flows

using if and else statement, to generate more complex yaml, based on the values we pass in, and it also allowed us to set default.\
let's say we want to enforce things like cpu and memory request and limit.\
so we change what we had before:
```
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```
with:
```
        {{- if .Values.deployment.resources }}
        resources:
          {{- if .Values.deployment.resources.requests }}
          requests:
            memory: {{ .Values.deployment.resources.requests.memory | default "50Mi" | quote }}
            cpu: {{ .Values.deployment.resources.requests.cpu | default "10m" | quote }}
          {{- else}}
          requests:
            memory: "50Mi"
            cpu: "10m"
          {{- end}}
          {{- if .Values.deployment.resources.limits }}
          limits:
            memory: {{ .Values.deployment.resources.limits.memory | default "1024Mi" | quote }}
            cpu: {{ .Values.deployment.resources.limits.cpu | default "1" | quote }}
          {{- else}}  
          limits:
            memory: "1024Mi"
            cpu: "1"
          {{- end }}
        {{- else }}
        resources:
          requests:
            memory: "50Mi"
            cpu: "10m"
          limits:
            memory: "1024Mi"
            cpu: "1"
        {{- end}} 
```







