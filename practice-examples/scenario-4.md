# Scenario 4 — YAML + Namespace

1. Create a namespace called staging
2. Write a single YAML file called staging-app.yaml containing:
 - Deployment name: frontend
 - image: nginx 
 - replicas: 3 in staging namespace
3. Service name: frontend-svc 
 - type: NodePort 
 - port: 80 
 - nodePort: 30090 in staging namespace

4. Apply the file
5. Verify pods and service in staging namespace
6. Test with curl localhost:30090

## Solution

```
root@controlplane:~$ kubectl create namespace staging
namespace/staging created
root@controlplane:~$ kubectl get namespaces staging 
NAME      STATUS   AGE
staging   Active   13s
root@controlplane:~$ nano staging-app.yaml
root@controlplane:~$ kubectl apply -f staging-app.yaml 
deployment.apps/frontend created
service/frontend-service created
root@controlplane:~$ kubectl get deployments.apps 
No resources found in default namespace.
root@controlplane:~$ kubectl get deployments.apps -n staging 
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   3/3     3            3           29s
root@controlplane:~$ kubectl get pods -n staging 
NAME                        READY   STATUS    RESTARTS   AGE
frontend-64fcfd56f8-l4ztk   1/1     Running   0          47s
frontend-64fcfd56f8-sv64g   1/1     Running   0          47s
frontend-64fcfd56f8-wbxsj   1/1     Running   0          47s
root@controlplane:~$ kubectl get svc    
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   11d
root@controlplane:~$ kubectl get svc -n staging 
NAME               TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
frontend-service   NodePort   10.99.76.74   <none>        80:30090/TCP   67s
root@controlplane:~$ curl localhost:30090       
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
root@controlplane:~$ 
```

```
In MiniKube-

minikube service list
minikube service frontend-service -n staging
```