In this lab, we will create namespace and a pod inside the namespace. 

1. Generate the yaml file to create a namespace called production. Notice the use of --dry-run=client flag. This flag doesnt execute the command but instead mimics running the command.  
```yaml
$ kubectl create ns production --dry-run=client -o yaml > lab1.yaml
```

2. Generate the yaml file to create a pod in production namespace.
```yaml 
$ kubectl run nginx-pod --image=nginx -n production --dry-run=client -o yaml >> lab1.yaml 
```

3. Add 3 dashes above apiVersion in lab1.yaml. File should look similar to below content:
```yaml
$ cat lab1.yaml 
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: production
spec: {}
status: {}
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
  namespace: production
spec:
  containers:
  - image: nginx
    name: nginx-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

4. Create the kubernetes objects (pod and ns) using kubectl command. 
```yaml
$ kubectl create -f lab1.yaml 
namespace/production created
pod/nginx-pod created
```

5. Verify of the namespace is created.
```yaml
$ kubectl get ns 
NAME                 STATUS   AGE
default              Active   11h
kube-node-lease      Active   11h
kube-public          Active   11h
kube-system          Active   11h
local-path-storage   Active   11h
production           Active   6s
$
```

6. Verify if pod is created. 
```yaml
$ kubectl get po 
No resources found in default namespace.
$ 
```
It wont be showing up because we are checking in the default namespace. 

7. Verify if pod is created in production namespace. 
```yaml
$ kubectl get po -n production 
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          19s
$ 
```

8. Delete the pod and namespace. Note that deleting the namespace removes all kubernetes objects inside the namespace. Be extra careful while deteing namespace. 
```yaml
$ kubectl delete ns production 
namespace "production" deleted
$
```
