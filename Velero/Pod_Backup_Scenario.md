## Now, here’s a full scenario for you using the a-dehghani namespace:
### 📌 Step-by-step Velero Backup & Restore Scenario
#### 👣 1. Deploy a simple NGINX pod
first create a Pod manifest named nginx.yaml:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: a-dehghani
  labels:
    app: nginx-app
spec:
  containers:
  - name: ng1
    image: nginx:latest
    ports:
    - containerPort: 80
```
then using command below install the NGINX pod in a-dehghani namespace:
```
kubectl apply -f nginx.yaml
```
make sure it's running:
```
kubectl get pod nginx -n a-dehghani
```
#### 💾 2. Create a Velero backup
```
velero backup create nginx-backup \
  --include-namespaces a-dehghani \
  --selector run=nginx-app \
  -n a-dehghani
```
`--selector run=nginx-app` assumes your pod has that label. If unsure, run:
```
kubectl get pod nginx-test -n a-dehghani --show-labels
```
Check backup status:
```
velero backup get -n a-dehghani
```
#### ❌ 3. Delete the NGINX pod
```
kubectl delete pod nginx-test -n a-dehghani
```
Confirm it's gone:
```
kubectl get pod nginx -n a-dehghani
```
#### ♻️ 4. Restore from backup
```
velero restore create nginx-restore \
  --from-backup nginx-backup \
  -n a-dehghani
```
Monitor restore progress:
```
velero restore get -n a-dehghani
```
#### ✅ 5. Verify the NGINX pod is back
```
kubectl get pod -n a-dehghani
```
You should see nginx-test recreated.




