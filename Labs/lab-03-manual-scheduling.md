In this lab, we will perform manual scheduling of pod. 

1. Create a pod manifest file named manual-pod.yml. Pass the node name as the value for **nodeName** in the pod spec. 
```yaml
$ cat manual-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels: 
    app: nginx

spec:
  containers:
    - name: nginx
      image: nginx
      ports
        - containerPort: 8080
  nodeName: node02
$
```
2. Create the pod and verify where it is running. 
```yaml
$ kubectl create -f manual-pod.yml
$ kubectl get pods
NAME     READY   STATUS      RESTARTS   AGE   IP             NODE
nginx    1/1     Running     0          12s   10.40.0.15     node02
$
```
4. Cleanup
```yaml
$ kubectl delete -f manual-pod.yml
```


