# fecp4-1022
FECP4-1022 Lab 1: Deploying environments in Kubernetes

Below are the instructions on how to create a deployment for production environment.

### 1. Create namespace
```
controlplane $ k create ns production
namespace/production created
```
### 2. Create configmap
```
controlplane $ k create configmap app-config --from-literal=APP_ENV=production -n production 
configmap/app-config created
```
### 3. Create secret
```
controlplane $ k create secret generic app-secret  --from-literal=DB_PASSWORD=prodpassword123 -n production 
secret/app-secret created
```
To verify the secret:
```
controlplane $ k get secret -n production 
NAME         TYPE     DATA   AGE
app-secret   Opaque   1      18s
```
```
controlplane $ k describe secret -n production app-secret 
Name:         app-secret
Namespace:    production
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
DB_PASSWORD:  15 bytes
```
```
controlplane $ k get secret app-secret -n production -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode
prodpassword123
```
### 4. From app-deployment-template.yaml, create a deployment in production:
```
controlplane $ sed -e 's/PLACEHOLDER_NAMESPACE/production/' \
> -e 's/replicas: .*$/replicas: 3/' app-deployment-template.yaml > production-deployment.yaml
controlplane $ k apply -f production-deployment.yaml 
deployment.apps/nginx-app created
```
### 5. Expose service
```
controlplane $ kubectl expose deployment nginx-app \
>   --type=NodePort \
>   --name=nginx-service \
>   --port=80 \
>   --target-port=80 \
>   --namespace=production
service/nginx-service exposed
```
To verify resources:
```
controlplane $ k get all -n production 
NAME                             READY   STATUS    RESTARTS   AGE
pod/nginx-app-65968468b6-2cfxp   1/1     Running   0          79s
pod/nginx-app-65968468b6-fm8ms   1/1     Running   0          79s
pod/nginx-app-65968468b6-wg744   1/1     Running   0          79s

NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/nginx-service   NodePort   10.100.160.53   <none>        80:30877/TCP   4s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-app   3/3     3            3           79s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-app-65968468b6   3         3         3       79s
```
### 6. Verify pods
```
controlplane $ k -n production exec -it nginx-app-65968468b6-2cfxp -- /bin/bash
root@nginx-app-65968468b6-2cfxp:/# echo $APP_ENV
production
root@nginx-app-65968468b6-2cfxp:/# echo $DB_PASSWORD
prodpassword123
```
```
controlplane $ k -n production exec -it nginx-app-65968468b6-fm8ms -- /bin/bash
root@nginx-app-65968468b6-fm8ms:/# echo $APP_ENV
echo $DB_PASSWORD
production
prodpassword123
```
```
controlplane $ k -n production exec -it nginx-app-65968468b6-wg744 -- /bin/bash
root@nginx-app-65968468b6-wg744:/# echo $APP_ENV
echo $DB_PASSWORD
production
prodpassword123
```
