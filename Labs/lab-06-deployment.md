In this lab we will create deployment and perform upgrade & rollback. 

1. Create a nginx deployment named nginx-deployment. 
```yaml
cloud_user_p_bed41efd@kind:~$ cat nginx-deploy.yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      deploy: example
  template:
    metadata:
      labels:
        deploy: example
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9
cloud_user_p_bed41efd@kind:~$ kubectl create -f nginx-deploy.yaml 
deployment.apps/nginx-deployment created
cloud_user_p_bed41efd@kind:~$
```
2. Check the status of the K8s objects. 
```yaml
cloud_user_p_bed41efd@kind:~$ kubectl get deployments 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           61s
cloud_user_p_bed41efd@kind:~$
cloud_user_p_bed41efd@kind:~$ kubectl get po 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-569558bf55-5hktb   1/1     Running   0          99s
nginx-deployment-569558bf55-ccz7j   1/1     Running   0          99s
nginx-deployment-569558bf55-qvfwb   1/1     Running   0          99s
cloud_user_p_bed41efd@kind:~$
```
3. Scale the replicas to 5. 
```yaml
cloud_user_p_bed41efd@kind:~$ kubectl scale deployment nginx-deployment --replicas=5
deployment.apps/nginx-deployment scaled
cloud_user_p_bed41efd@kind:~$ kubectl get po 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-569558bf55-76k8p   1/1     Running   0          6s
nginx-deployment-569558bf55-ccz7j   1/1     Running   0          4m31s
nginx-deployment-569558bf55-l7mv5   1/1     Running   0          6s
nginx-deployment-569558bf55-q4bk5   1/1     Running   0          2m31s
nginx-deployment-569558bf55-qvfwb   1/1     Running   0          4m31s
cloud_user_p_bed41efd@kind:~$
```
4. Update the nginx version from 1.7.9 to 1.8 by editing the deployment.
```yaml
cloud_user_p_bed41efd@kind:~$ kubectl edit deployment nginx-deployment
deployment.apps/nginx-deployment edited
cloud_user_p_bed41efd@kind:~$
cloud_user_p_bed41efd@kind:~$ kubectl rollout status deployment.v1.apps/nginx-deployment 
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
cloud_user_p_bed41efd@kind:~$
```
5. Check the rollout history. 
```yaml
cloud_user_p_bed41efd@kind:~$ kubectl rollout history deployment/nginx-deployment 
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/nginx-deployment nginx=nginx:broken --record=true

cloud_user_p_bed41efd@kind:~$
```
6. Perform a rollback. 
```yaml
cloud_user_p_bed41efd@kind:~$ kubectl rollout undo deployment/nginx-deployment 
deployment.apps/nginx-deployment rolled back
cloud_user_p_bed41efd@kind:~$
```
7. Verify that the rollback is successful by checking the version of nginx image in the running pod. 
```yaml
cloud_user_p_bed41efd@kind:~$ kubectl describe pod nginx-deployment-569558bf55-5hktb | grep image
image: nginx:1.7.9
cloud_user_p_bed41efd@kind:~$
```
8. Cleanup 
```yaml
cloud_user_p_bed41efd@kind:~$ kubectl delete -f nginx-deploy.yaml 
deployment.apps/nginx-deployment deleted
cloud_user_p_bed41efd@kind:~$
```
