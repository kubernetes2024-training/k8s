In this lab, we will explore label and selectors in Kubernetes. 

1. Create the following manifest file called label-selector.yml
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metadata-deployment
  labels:
    app: metadata-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: metadata-pod
  template:
    metadata:
      labels:
        app: metadata-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: metadata-service
  labels:
    app: metadata-service
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: metadata-pod
```
3. Notice that the pod label is used in the **matchLabels** section in Deployment and **selector** section in the service manifest.
4. Create the pod and service.
```yaml
cloud_user_p_d9227775@cloudshell:~$ kubectl create -f label-selector.yml
deployment.apps/metadata-deployment created
service/metadata-service created
cloud_user_p_d9227775@cloudshell:~$
```
6. Verify that the IP in the Endpoint in service is the Pod IP. 
```yaml
cloud_user_p_d9227775@cloudshell:~$ kubectl get svc 
NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes         ClusterIP   10.8.0.1     <none>        443/TCP   157m
metadata-service   ClusterIP   10.8.13.98   <none>        80/TCP    8s
cloud_user_p_d9227775@cloudshell:~$ kubectl describe svc metadata-service 
Name:              metadata-service
Namespace:         default
Labels:            app=metadata-service
Annotations:       cloud.google.com/neg: {"ingress":true}
Selector:          app=metadata-pod
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.8.13.98
IPs:               10.8.13.98
Port:              http  80/TCP
TargetPort:        80/TCP
Endpoints:         10.4.2.8:80
Session Affinity:  None
Events:            <none>
cloud_user_p_d9227775@cloudshell:~$ kubectl get po -o wide 
NAME                                   READY   STATUS    RESTARTS   AGE   IP         NODE                                       NOMINATED NODE   READINESS GATES
metadata-deployment-7548848db5-z5l2j   1/1     Running   0          46s   10.4.2.8   gke-cluster-1-default-pool-4b825c88-wdd5   <none>           <none>
cloud_user_p_d9227775@cloudshell:~$
```
8. Cleanup
```yaml
cloud_user_p_d9227775@cloudshell:~$ kubectl delete -f label-selector.yml
deployment.apps "metadata-deployment" deleted
service "metadata-service" deleted
cloud_user_p_d9227775@cloudshell:~$
$
```
