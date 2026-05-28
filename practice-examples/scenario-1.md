# Scenario 1 — Pod Basics

Create a pod with the following:
1. Name: mypod  
2. Image: nginx  
3. Namespace: default

Then:
1. Verify it's running
2. Shell into it
3. Run curl localhost inside
4. Exit and describe the pod
5. Check which node it's scheduled on

## Solution

```
root@controlplane:~$ kubectl run mypod --image=nginx
pod/mypod created
root@controlplane:~$ kubectl get pods
NAME    READY   STATUS              RESTARTS   AGE
mypod   0/1     ContainerCreating   0          10s
root@controlplane:~$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          17s
root@controlplane:~$ kubectl exec -it mypod -- bash
root@mypod:/# ls
bin                   etc    mnt   sbin  var
boot                  home   opt   srv
dev                   lib    proc  sys
docker-entrypoint.d   lib64  root  tmp
docker-entrypoint.sh  media  run   usr
root@mypod:/# exit
exit
root@controlplane:~$ kubectl describe pod mypod 
Name:             mypod
Namespace:        default
Priority:         0
Service Account:  default
Node:             node01/172.30.2.2
Start Time:       Thu, 28 May 2026 09:44:51 +0000
Labels:           run=mypod
Annotations:      <none>
Status:           Running
IP:               192.168.1.119
IPs:
  IP:  192.168.1.119
Containers:
  mypod:
    Container ID:   containerd://4f87e3172af467b1035b33c4b5aca2bf51a9c612f2c4ca59697a0206b145f5df
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:5aca99593157f4ae539a5dec1092a0ad8762f8e2eb1789085a13a0f5622369f6
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 28 May 2026 09:45:01 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bp6sz (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-bp6sz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  65s   default-scheduler  Successfully assigned default/mypod to node01
  Normal  Pulling    61s   kubelet            spec.containers{mypod}: Pulling image "nginx"
  Normal  Pulled     55s   kubelet            spec.containers{mypod}: Successfully pulled image "nginx" in 6.683s (6.683s including waiting). Image size: 63120520 bytes.
  Normal  Created    55s   kubelet            spec.containers{mypod}: Container created
  Normal  Started    55s   kubelet            spec.containers{mypod}: Container started
root@controlplane:~$ 

```