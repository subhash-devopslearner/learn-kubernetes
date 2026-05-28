# Scenario 2 — Deployment + Scaling

1. Delete mypod
2. Create a Deployment called webserver using image httpd with 2 replicas in namespace default
3. Verify both pods are running
4. Scale up to 5 replicas
5. Delete one pod manually and watch self-healing
6. Scale back down to 3 replicas

## Solution

```
root@controlplane:~$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          13m
root@controlplane:~$ kubectl delete pod mypod 
pod "mypod" deleted from default namespace
root@controlplane:~$ kubectl create deployment webserver --image=httpd --replicas=2
deployment.apps/webserver created
root@controlplane:~$ kubectl get pods -w
NAME                         READY   STATUS    RESTARTS   AGE
webserver-6c765df6c8-8dchj   1/1     Running   0          17s
webserver-6c765df6c8-8wzm2   1/1     Running   0          17s
root@controlplane:~$ kubectl scale deployment webserver --replicas=5
deployment.apps/webserver scaled
root@controlplane:~$ kubectl get pods -w
NAME                         READY   STATUS    RESTARTS   AGE
webserver-6c765df6c8-7hdhh   1/1     Running   0          8s
webserver-6c765df6c8-8dchj   1/1     Running   0          71s
webserver-6c765df6c8-8wzm2   1/1     Running   0          71s
webserver-6c765df6c8-sw9ln   1/1     Running   0          8s
webserver-6c765df6c8-zhdqk   1/1     Running   0          8s
^Croot@controlplane:~$ kubectl delete pod webserver-6c7656c8-zhdqk 
pod "webserver-6c765df6c8-zhdqk" deleted from default namespace
root@controlplane:~$ kubectl get pods -w
NAME                         READY   STATUS    RESTARTS   AGE
webserver-6c765df6c8-7hdhh   1/1     Running   0          37s
webserver-6c765df6c8-8dchj   1/1     Running   0          100s
webserver-6c765df6c8-8wzm2   1/1     Running   0          100s
webserver-6c765df6c8-rvbr2   1/1     Running   0          5s
webserver-6c765df6c8-sw9ln   1/1     Running   0          37s
root@controlplane:~$ kubectl scale deployment webserver --replicas=3
deployment.apps/webserver scaled
root@controlplane:~$ kubectl get pods -w
NAME                         READY   STATUS    RESTARTS   AGE
webserver-6c765df6c8-7hdhh   1/1     Running   0          62s
webserver-6c765df6c8-8dchj   1/1     Running   0          2m5s
webserver-6c765df6c8-8wzm2   1/1     Running   0          2m5s
^Croot@controlplane:~$ 
```