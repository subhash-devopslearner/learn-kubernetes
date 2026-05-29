# Scenario 7 — Ingress

In staging namespace create a single YAML file ingress-app.yaml with:

1. Deployment frontend — image httpd — 2 replicas
2. Deployment api — image nginx — 2 replicas
3. Service frontend — ClusterIP — port 80
4. Service api — ClusterIP — port 80

Ingress staging-ingress with:

1. host: staging.local
2. / → frontend service
3. /api → api service



Then install nginx ingress controller and test:
1. curl -H "Host: staging.local" localhost:<nodeport>/
2. curl -H "Host: staging.local" localhost:<nodeport>/api


## Solution

```
root@controlplane:~$ helm upgrade -i ingress-nginx ingress-nginx/ingress-nginx     --namespace kube-system --set controller.service.type=NodePort
Release "ingress-nginx" does not exist. Installing it now.
NAME: ingress-nginx
LAST DEPLOYED: Fri May 29 08:22:58 2026
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
Get the application URL by running these commands:
  export HTTP_NODE_PORT=$(kubectl get service --namespace kube-system ingress-nginx-controller --output jsonpath="{.spec.ports[0].nodePort}")
  export HTTPS_NODE_PORT=$(kubectl get service --namespace kube-system ingress-nginx-controller --output jsonpath="{.spec.ports[1].nodePort}")
  export NODE_IP="$(kubectl get nodes --output jsonpath="{.items[0].status.addresses[1].address}")"

  echo "Visit http://${NODE_IP}:${HTTP_NODE_PORT} to access your application via HTTP."
  echo "Visit https://${NODE_IP}:${HTTPS_NODE_PORT} to access your application via HTTPS."

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

root@controlplane:~$ kubectl -n kube-system rollout status deployment ingress-nginx-controller
deployment "ingress-nginx-controller" successfully rolled out

root@controlplane:~$ kubectl get deployment -n kube-system ingress-nginx-controller
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           97s

root@controlplane:~$ kubectl get pod -A
NAMESPACE            NAME                                        READY   STATUS    RESTARTS      AGE
kube-system          cilium-envoy-hrbhs                          1/1     Running   1 (26m ago)   12d
kube-system          cilium-envoy-hwqjz                          1/1     Running   2 (26m ago)   12d
kube-system          cilium-operator-5d8ddcb8d8-8h2jp            1/1     Running   3 (26m ago)   12d
kube-system          cilium-vtmg2                                1/1     Running   2 (26m ago)   12d
kube-system          cilium-vzvsz                                1/1     Running   1 (26m ago)   12d
kube-system          coredns-5f68d5bd7f-2nf9j                    1/1     Running   1 (26m ago)   12d
kube-system          coredns-5f68d5bd7f-7h5gs                    1/1     Running   1 (26m ago)   12d
kube-system          etcd-controlplane                           1/1     Running   2 (26m ago)   12d
kube-system          ingress-nginx-controller-6c7cd85885-t6w78   1/1     Running   0             114s
kube-system          kube-apiserver-controlplane                 1/1     Running   2 (26m ago)   12d
kube-system          kube-controller-manager-controlplane        1/1     Running   2 (26m ago)   12d
kube-system          kube-scheduler-controlplane                 1/1     Running   2 (26m ago)   12d
local-path-storage   local-path-provisioner-644f8b49d7-5xw8x     1/1     Running   2 (26m ago)   12d

root@controlplane:~$ kubectl get deployments.apps -A
NAMESPACE            NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
kube-system          cilium-operator            1/1     1            1           12d
kube-system          coredns                    2/2     2            2           12d
kube-system          ingress-nginx-controller   1/1     1            1           2m29s
local-path-storage   local-path-provisioner     1/1     1            1           12d

root@controlplane:~$ kubectl get svc -A             
NAMESPACE     NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
default       kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                      12d
kube-system   cilium-envoy                         ClusterIP   None             <none>        9964/TCP                     12d
kube-system   ingress-nginx-controller             NodePort    10.104.110.185   <none>        80:31183/TCP,443:30089/TCP   2m41s
kube-system   ingress-nginx-controller-admission   ClusterIP   10.107.184.130   <none>        443/TCP                      2m41s
kube-system   kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       12d

root@controlplane:~$ kubectl create namespace staging
namespace/staging created

root@controlplane:~$ nano ingress-app.yaml
root@controlplane:~$ kubectl apply -f ingress-app.yaml 
deployment.apps/frontend unchanged
deployment.apps/api unchanged
service/frontend unchanged
service/api unchanged
ingress.networking.k8s.io/ingress created

root@controlplane:~$ kubectl config set-context --current -n staging 
Context "kubernetes-admin@kubernetes" modified.

root@controlplane:~$ kubectl get deployments.apps 
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
api        2/2     2            2           2m
frontend   2/2     2            2           2m

root@controlplane:~$ kubectl get svc              
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
api        ClusterIP   10.106.89.94     <none>        80/TCP    2m9s
frontend   ClusterIP   10.107.249.127   <none>        80/TCP    2m9s

root@controlplane:~$ kubectl get ingress
NAME      CLASS   HOSTS           ADDRESS          PORTS   AGE
ingress   nginx   staging.local   10.104.110.185   80      57s

root@controlplane:~$ kubectl get all -n kube-system 
NAME                                            READY   STATUS    RESTARTS      AGE
pod/cilium-envoy-hrbhs                          1/1     Running   1 (34m ago)   12d
pod/cilium-envoy-hwqjz                          1/1     Running   2 (34m ago)   12d
pod/cilium-operator-5d8ddcb8d8-8h2jp            1/1     Running   3 (34m ago)   12d
pod/cilium-vtmg2                                1/1     Running   2 (34m ago)   12d
pod/cilium-vzvsz                                1/1     Running   1 (34m ago)   12d
pod/coredns-5f68d5bd7f-2nf9j                    1/1     Running   1 (34m ago)   12d
pod/coredns-5f68d5bd7f-7h5gs                    1/1     Running   1 (34m ago)   12d
pod/etcd-controlplane                           1/1     Running   2 (34m ago)   12d
pod/ingress-nginx-controller-6c7cd85885-t6w78   1/1     Running   0             9m32s
pod/kube-apiserver-controlplane                 1/1     Running   2 (34m ago)   12d
pod/kube-controller-manager-controlplane        1/1     Running   2 (34m ago)   12d
pod/kube-scheduler-controlplane                 1/1     Running   2 (34m ago)   12d

NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/cilium-envoy                         ClusterIP   None             <none>        9964/TCP                     12d
service/ingress-nginx-controller             NodePort    10.104.110.185   <none>        80:31183/TCP,443:30089/TCP   9m32s
service/ingress-nginx-controller-admission   ClusterIP   10.107.184.130   <none>        443/TCP                      9m32s
service/kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       12d

NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/cilium         2         2         2       2            2           kubernetes.io/os=linux   12d
daemonset.apps/cilium-envoy   2         2         2       2            2           kubernetes.io/os=linux   12d

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cilium-operator            1/1     1            1           12d
deployment.apps/coredns                    2/2     2            2           12d
deployment.apps/ingress-nginx-controller   1/1     1            1           9m32s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/cilium-operator-5d8ddcb8d8            1         1         1       12d
replicaset.apps/coredns-5f68d5bd7f                    2         2         2       12d
replicaset.apps/coredns-7d764666f9                    0         0         0       12d
replicaset.apps/ingress-nginx-controller-6c7cd85885   1         1         1       9m32s

root@controlplane:~$ kubectl get svc -n kube-system 
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
cilium-envoy                         ClusterIP   None             <none>        9964/TCP                     12d
ingress-nginx-controller             NodePort    10.104.110.185   <none>        80:31183/TCP,443:30089/TCP   9m56s
ingress-nginx-controller-admission   ClusterIP   10.107.184.130   <none>        443/TCP                      9m56s
kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       12d

root@controlplane:~$ kubectl get pods -n staging
NAME                       READY   STATUS    RESTARTS   AGE
api-74f9d65454-mt2jj       1/1     Running   0          6m5s
api-74f9d65454-xfs8j       1/1     Running   0          6m5s
frontend-8ff69fb94-ctfdb   1/1     Running   0          6m5s
frontend-8ff69fb94-jvwbm   1/1     Running   0          6m5s

root@controlplane:~$ kubectl get svc -n staging     
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
api        ClusterIP   10.106.89.94     <none>        80/TCP    6m25s
frontend   ClusterIP   10.107.249.127   <none>        80/TCP    6m25s

root@controlplane:~$ kubectl get ingress -n staging 
NAME      CLASS   HOSTS           ADDRESS          PORTS   AGE
ingress   nginx   staging.local   10.104.110.185   80      5m15s

root@controlplane:~$ curl -H "Host: staging.local" localhost:31183/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<title>It works! Apache httpd</title>
</head>
<body>
<p>It works!</p>
</body>
</html>

root@controlplane:~$ curl -H "Host: staging.local" localhost:31183/api
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

