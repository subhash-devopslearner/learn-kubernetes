# Scenario 5 — ConfigMap + Secret
In the staging namespace:

Create a ConfigMap called app-config with:

APP_ENV=staging
LOG_LEVEL=debug
MAX_CONNECTIONS=100


Create a Secret called app-secret with:

DB_PASSWORD=staging123
API_KEY=stg-key-xyz


Write a YAML file config-deploy.yaml with a Deployment that:

Name: config-app
Image: nginx
Replicas: 2
Injects both ConfigMap and Secret as environment variables
Namespace: staging


Verify by running:

kubectl exec -it <pod-name> -n staging -- env | grep -E "APP_ENV|LOG_LEVEL|MAX_CONNECTIONS|DB_PASSWORD|API_KEY"

## Solution

```
root@controlplane:~$ kubectl get namespaces 
NAME                 STATUS   AGE
cilium-secrets       Active   11d
default              Active   11d
kube-node-lease      Active   11d
kube-public          Active   11d
kube-system          Active   11d
local-path-storage   Active   11d
staging              Active   28m

root@controlplane:~$ kubectl config set-context --current -n staging 
Context "kubernetes-admin@kubernetes" modified.

root@controlplane:~$ kubectl create configmap app-config --from-literal=APP_ENV=staging --from-literal=LOG_LEVEL=debug --from-literal=MAX_CONNECTIONS=100
configmap/app-config created

root@controlplane:~$ kubectl get configmaps app-config 
NAME         DATA   AGE
app-config   3      17s

root@controlplane:~$ kubectl describe configmaps app-config 
Name:         app-config
Namespace:    staging
Labels:       <none>
Annotations:  <none>

Data
====
APP_ENV:
----
staging

LOG_LEVEL:
----
debug

MAX_CONNECTIONS:
----
100


BinaryData
====

Events:  <none>

root@controlplane:~$ kubectl create secret generic app-secret \
> --from-literal=DB_PASSWORD=staging123 \
> --from-literal=API_KEY=stg-key-xyz
secret/app-secret created

root@controlplane:~$ kubectl get secrets app-secret    
NAME         TYPE     DATA   AGE
app-secret   Opaque   2      13s

root@controlplane:~$ kubectl describe secrets app-secret    
Name:         app-secret
Namespace:    staging
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
API_KEY:      11 bytes
DB_PASSWORD:  10 bytes

root@controlplane:~$ kubectl get secrets app-secret -o yaml
apiVersion: v1
data:
  API_KEY: c3RnLWtleS14eXo=
  DB_PASSWORD: c3RhZ2luZzEyMw==
kind: Secret
metadata:
  creationTimestamp: "2026-05-28T11:21:38Z"
  name: app-secret
  namespace: staging
  resourceVersion: "9712"
  uid: 30523edb-a714-4b41-83cf-736e09643fc5
type: Opaque

root@controlplane:~$ echo -n "c3RnLWtleS14eXo=" | base64 --decode
stg-key-xyz

root@controlplane:~$ echo -n "c3RhZ2luZzEyMw==" | base64 --decode
staging123

root@controlplane:~$ nano config-deploy.yaml

root@controlplane:~$ kubectl apply -f config-deploy.yaml 
deployment.apps/config-app created

root@controlplane:~$ kubectl get deployments.apps config-app 
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
config-app   2/2     2            2           15s

root@controlplane:~$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
config-app-59886b5995-5gpmk   1/1     Running   0          23s
config-app-59886b5995-68crt   1/1     Running   0          23s
frontend-64fcfd56f8-l4ztk     1/1     Running   0          45m
frontend-64fcfd56f8-sv64g     1/1     Running   0          45m
frontend-64fcfd56f8-wbxsj     1/1     Running   0          45m

root@controlplane:~$ kubectl exec -it config-app-59886b5995-5gpmk -- env | grep -E "APP_ENV|LOG_LEVEL|MAX_CONNECTIONS|DB_PASSWORD|API_KEY"
APP_ENV=staging
LOG_LEVEL=debug
MAX_CONNECTIONS=100
DB_PASSWORD=staging123
API_KEY=stg-key-xyz
root@controlplane:~$ 
```