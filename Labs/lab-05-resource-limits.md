This lab will demonstrate how resource limits are enforced on kubernetes Pods. 

1. Create a pod manifest named limit.yml
```yaml
cloud_user_p_d9227775@cloudshell:~$ cat limit.yml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limit
spec:
  containers:
  - name: resource-limit
    image: nginx
    resources:
      limits:
        cpu: "1"
        memory: "8192M"
      requests:
        cpu: "0.5"
        memory: "4096M"
cloud_user_p_d9227775@cloudshell:~$
```
2. Create the pod
```yaml
cloud_user_p_d9227775@cloudshell:~$ kubectl apply -f limit.yml
pod/resource-limit created
cloud_user_p_d9227775@cloudshell:~$
```
3. Verify the status
```yaml
cloud_user_p_d9227775@cloudshell:~$ kubectl get po | grep resource-limit 
resource-limit   0/1     Pending   0          22s
cloud_user_p_d9227775@cloudshell:~$
```
4. Check why the pod is in pending state.
```yaml
cloud_user_p_d9227775@cloudshell:~$ kubectl get events | grep resource-limit
3m44s       Warning   FailedScheduling    pod/resource-limit                          0/3 nodes are available: 2 Insufficient cpu, 3 Insufficient memory. preemption: 0/3 nodes are available: 3 No preemption victims found for incoming pod..
3m44s       Normal    NotTriggerScaleUp   pod/resource-limit                          pod didn't trigger scale-up:
cloud_user_p_d9227775@cloudshell:~$ 
```
5. Modify the pod spec and reduce resources allocated to the pod
```yaml
cloud_user_p_d9227775@cloudshell:~$ kubectl delete -f limit.yml 
pod "resource-limit" deleted
cloud_user_p_d9227775@cloudshell:~$ vi limit.yml 
cloud_user_p_d9227775@cloudshell:~$ cat limit.yml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limit
spec:
  containers:
  - name: resource-limit
    image: nginx
    resources:
      limits:
        cpu: 0.3
        memory: "200M"
      requests:
        cpu: 0.3
        memory: "100M"
cloud_user_p_d9227775@cloudshell:~$ kubectl create -f limit.yml 
pod/resource-limit created
cloud_user_p_d9227775@cloudshell:~$ kubectl get pods 
NAME             READY   STATUS    RESTARTS   AGE
resource-limit   1/1     Running   0          7s
cloud_user_p_d9227775@cloudshell:~$
```

