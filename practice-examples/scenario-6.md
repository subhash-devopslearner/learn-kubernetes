# Scenario 6 — Resource Limits + Health Checks
Write a single YAML file healthy-app.yaml in staging namespace with:

Deployment:
- Name: healthy-app
- Image: nginx
- Replicas: 2
- Resource requests: CPU 100m memory 128Mi
- Resource limits: CPU 200m memory 256Mi
- Readiness probe: HTTP GET / port 80 initialDelay 5s period 10s
- Liveness probe: HTTP GET / port 80 initialDelay 15s period 20s    
    
    
Then:
1. Apply and verify pods are running
2. Describe one pod and show Limits, Requests, Liveness and Readiness sections
3. Delete the nginx index.html inside one pod and watch it restart

## Solution

```
root@controlplane:~$ kubectl create namespace staging
namespace/staging created

root@controlplane:~$ kubectl config set-context --current -n staging 
Context "kubernetes-admin@kubernetes" modified.

root@controlplane:~$ kubectl get pods
No resources found in staging namespace.

root@controlplane:~$ nano healthy-app.yaml

root@controlplane:~$ kubectl apply -f healthy-app.yaml 
deployment.apps/healthy-app created

root@controlplane:~$ kubectl get deployments.apps 
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
healthy-app   0/2     2            0           16s

root@controlplane:~$ kubectl get deployments.apps 
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
healthy-app   1/2     2            1           19s

root@controlplane:~$ kubectl get deployments.apps 
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
healthy-app   2/2     2            2           24s

root@controlplane:~$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
healthy-app-7778b79c46-hwpqg   1/1     Running   0          37s
healthy-app-7778b79c46-mlpjp   1/1     Running   0          37s

root@controlplane:~$ kubectl describe pod healthy-app-7778b79c46-hwpqg 
Name:             healthy-app-7778b79c46-hwpqg
Namespace:        staging
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Thu, 28 May 2026 17:33:21 +0000
Labels:           app=frontend
                  pod-template-hash=7778b79c46
Annotations:      <none>
Status:           Running
IP:               192.168.1.35
IPs:
  IP:           192.168.1.35
Controlled By:  ReplicaSet/healthy-app-7778b79c46
Containers:
  nginx:
    Container ID:   containerd://7f7920273e1c0f48473dbb751e91534f981100ab8749481c8ac5def30683a593
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:5aca99593157f4ae539a5dec1092a0ad8762f8e2eb1789085a13a0f5622369f6
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 28 May 2026 17:33:29 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     200m
      memory:  256Mi
    Requests:
      cpu:        100m
      memory:     128Mi
    Liveness:     http-get http://:80/ delay=15s timeout=1s period=20s #success=1 #failure=3
    Readiness:    http-get http://:80/ delay=5s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2b8nr (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-2b8nr:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  64s   default-scheduler  Successfully assigned staging/healthy-app-7778b79c46-hwpqg to node01
  Normal  Pulling    61s   kubelet            spec.containers{nginx}: Pulling image "nginx"
  Normal  Pulled     56s   kubelet            spec.containers{nginx}: Successfully pulled image "nginx" in 4.748s (4.748s including waiting). Image size: 63120520 bytes.
  Normal  Created    56s   kubelet            spec.containers{nginx}: Container created
  Normal  Started    56s   kubelet            spec.containers{nginx}: Container started

root@controlplane:~$ kubectl exec -it healthy-app-7778b79c46-hwpqg  -- bash

root@healthy-app-7778b79c46-hwpqg:/# cd /usr/share/nginx/html/

root@healthy-app-7778b79c46-hwpqg:/usr/share/nginx/html# ls
50x.html  index.html

root@healthy-app-7778b79c46-hwpqg:/usr/share/nginx/html# rm index.html 

root@healthy-app-7778b79c46-hwpqg:/usr/share/nginx/html# ls
50x.html

root@healthy-app-7778b79c46-hwpqg:/usr/share/nginx/html# exit
exit

root@controlplane:~$ kubectl get pods -w
NAME                           READY   STATUS    RESTARTS   AGE
healthy-app-7778b79c46-hwpqg   1/1     Running   0          3m28s
healthy-app-7778b79c46-mlpjp   1/1     Running   0          3m28s
healthy-app-7778b79c46-hwpqg   0/1     Running   0          3m30s
healthy-app-7778b79c46-hwpqg   0/1     Running   1 (2s ago)   4m2s
healthy-app-7778b79c46-hwpqg   1/1     Running   1 (14s ago)   4m14s
^Croot@controlplane:~$ 
```

```
Later changed deployment name with pod labels-
matchLabels:
  app: healthy-app   ← match the deployment name
```