# Scenario 3 — Services
Using your existing webserver deployment:

1. Expose it as a ClusterIP service on port 80
2. Verify the service and note the ClusterIP
3. Create a temporary pod and curl the ClusterIP
4. Check the service endpoints
5. Scale deployment to 5 replicas and check if endpoints updated automatically

## Solution

```
root@controlplane:~$ kubectl get deployments.apps 
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
webserver   3/3     3            3           6m14s
root@controlplane:~$ kubectl expose deployment webserver --port=80
service/webserver exposed
root@controlplane:~$ kubectl get svc webserver    
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
webserver   ClusterIP   10.106.202.119   <none>        80/TCP    12s
root@controlplane:~$ kubectl run temppod --image=nginx
pod/temppod created
root@controlplane:~$ kubectl get pod
NAME                         READY   STATUS    RESTARTS   AGE
temppod                      1/1     Running   0          8s
webserver-6c765df6c8-7hdhh   1/1     Running   0          6m43s
webserver-6c765df6c8-8dchj   1/1     Running   0          7m46s
webserver-6c765df6c8-8wzm2   1/1     Running   0          7m46s
root@controlplane:~$ kubectl exec -it temppod -- curl 10.106.202.119
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<title>It works! Apache httpd</title>
</head>
<body>
<p>It works!</p>
</body>
</html>
root@controlplane:~$ kubectl describe svc webserver 
Name:                     webserver
Namespace:                default
Labels:                   app=webserver
Annotations:              <none>
Selector:                 app=webserver
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.106.202.119
IPs:                      10.106.202.119
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                192.168.1.4:80,192.168.0.80:80,192.168.0.162:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
root@controlplane:~$ kubectl scale deployment webserver --replicas=5
deployment.apps/webserver scaled
root@controlplane:~$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
temppod                      1/1     Running   0          2m15s
webserver-6c765df6c8-7hdhh   1/1     Running   0          8m50s
webserver-6c765df6c8-8dchj   1/1     Running   0          9m53s
webserver-6c765df6c8-8wzm2   1/1     Running   0          9m53s
webserver-6c765df6c8-mzrp4   1/1     Running   0          15s
webserver-6c765df6c8-qbr2t   1/1     Running   0          15s
root@controlplane:~$ kubectl describe svc webserver 
Name:                     webserver
Namespace:                default
Labels:                   app=webserver
Annotations:              <none>
Selector:                 app=webserver
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.106.202.119
IPs:                      10.106.202.119
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                192.168.1.4:80,192.168.0.80:80,192.168.0.162:80 + 2 more...
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
root@controlplane:~$ 
```