# Scenario 9 — RBAC
In staging namespace:

1. Create a ServiceAccount called developer
2. Create a Role called developer-role that allows:

    - get, list, watch on pods
    - get, list, watch on deployments
    - create, update on configmaps


3. Create a RoleBinding called developer-binding that binds
developer-role to developer ServiceAccount
4. Verify permissions with kubectl auth can-i:

    - get pods ✅
    - delete pods ❌
    - get deployments ✅
    - create configmaps ✅
    - delete secrets ❌
    - get services ❌

## Solution

```
root@controlplane:~$ kubectl create namespace staging
namespace/staging created

root@controlplane:~$ kubectl config set-context --current -n staging 
Context "kubernetes-admin@kubernetes" modified.

root@controlplane:~$ nano rbac-policy.yaml

root@controlplane:~$ kubectl apply -f rbac-policy.yaml 
serviceaccount/developer created
role.rbac.authorization.k8s.io/developer-role created
rolebinding.rbac.authorization.k8s.io/developer-binding created

root@controlplane:~$ kubectl get serviceaccounts 
NAME        AGE
default     85s
developer   10s

root@controlplane:~$ kubectl get role            
NAME             CREATED AT
developer-role   2026-05-30T18:36:46Z

root@controlplane:~$ kubectl get rolebindings.rbac.authorization.k8s.io 
NAME                ROLE                  AGE
developer-binding   Role/developer-role   24s

root@controlplane:~$ kubectl auth can-i get pods --as=system:serviceaccount:staging:developer
yes

root@controlplane:~$ kubectl auth can-i delete pods --as=system:serviceaccount:staging:developer
no

root@controlplane:~$ kubectl auth can-i get deployments --as=system:serviceaccount:staging:developer
yes

root@controlplane:~$ kubectl auth can-i create configmaps --as=system:serviceaccount:staging:developer
yes

root@controlplane:~$ kubectl auth can-i delete secrets --as=system:serviceaccount:staging:developer
no

root@controlplane:~$ kubectl auth can-i get services --as=system:serviceaccount:staging:developer
no
```

```
To get list of apiGroups
kubectl api-resources | grep deployments
kubectl explain deployments

To check resource permissions
kubectl auth can-i <verb> <resource>
```
