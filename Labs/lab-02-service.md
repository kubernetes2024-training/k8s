In this lab, we will create a nginx pod and a service to access the application. 

1. Create a deployment named nginxsvc using the image nginx 
```yaml
$ kubectl create deploy nginxsvc --image=nginx 
deployment.apps/nginxsvc created
$ 
```

2. Scale the number of pods to 3 
```yaml 
$ kubectl scale deploy nginxsvc --replicas=3 
deployment.apps/nginxsvc scaled
$
```

3. List all the objects using 'kubectl get all'
```yaml
$ kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/nginxsvc-6f45cc47b4-2qcbb   1/1     Running   0          20s
pod/nginxsvc-6f45cc47b4-6cw7z   1/1     Running   0          47s
pod/nginxsvc-6f45cc47b4-l7x4k   1/1     Running   0          20s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   13h

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginxsvc   3/3     3            3           47s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginxsvc-6f45cc47b4   3         3         3       47s
$
```

4. Expose the service  
```yaml
$ kubectl expose deploy nginxsvc --port=80 
service/nginxsvc exposed
$
```

5. View the details of the service. The IPs may be different in your system.
```yaml
$ kubectl describe svc nginxsvc 
Name:              nginxsvc
Namespace:         default
Labels:            app=nginxsvc
Annotations:       <none>
Selector:          app=nginxsvc
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.24.47
IPs:               10.96.24.47
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.3:80,10.244.1.4:80,10.244.2.3:80
Session Affinity:  None
Events:            <none>
$ 
```

6. Get the yaml output of the service.  
```yaml
$ kubectl get svc nginxsvc -o yaml 
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2023-11-16T09:33:08Z"
  labels:
    app: nginxsvc
  name: nginxsvc
  namespace: default
  resourceVersion: "39621"
  uid: a76667f4-b058-44d4-bd02-796ad6d23278
spec:
  clusterIP: 10.96.24.47
  clusterIPs:
  - 10.96.24.47
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginxsvc
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
$ 
```

7. Gather the endpoint and service IP. 
```yaml
$ kubectl get endpoints 
NAME         ENDPOINTS                                   AGE
kubernetes   172.18.0.4:6443                             14h
nginxsvc     10.244.1.3:80,10.244.1.4:80,10.244.2.3:80   2m41s
$ 
$ kubectl get svc 
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   14h
nginxsvc     ClusterIP   10.96.24.47   <none>        80/TCP    2m51s
$ 
```

9. Edit the service and update the type as NodePort. After editing the contents should be similar to step 10.
```yaml
$ kubectl edit svc nginxsvc 
service/nginxsvc edited
$ 
```

10. Get the yaml form of service configuration. Additions highlighted in bold.  
```yaml
$ kubectl get svc nginxsvc -o yaml 
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2023-11-16T09:33:08Z"
  labels:
    app: nginxsvc
  name: nginxsvc
  namespace: default
  resourceVersion: "40350"
  uid: a76667f4-b058-44d4-bd02-796ad6d23278
spec:
  clusterIP: 10.96.24.47
  clusterIPs:
  - 10.96.24.47
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - **nodePort: 32000**
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginxsvc
  sessionAffinity: None
  **type: NodePort**
status:
  loadBalancer: {}
$
```

11. Find the node in which application pod is running.  
```yaml
$ kubectl get po -o wide 
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
nginxsvc-6f45cc47b4-2qcbb   1/1     Running   0          8m51s   10.244.2.3   kind-worker2   <none>           <none>
nginxsvc-6f45cc47b4-6cw7z   1/1     Running   0          9m18s   10.244.1.3   kind-worker    <none>           <none>
nginxsvc-6f45cc47b4-l7x4k   1/1     Running   0          8m51s   10.244.1.4   kind-worker    <none>           <none>
$ 
```

12. Get the IP of the worker node. 
```yaml
$ docker inspect kind-worker | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    **"IPAddress": "172.18.0.3",**
$ 
```

13. Access the application using node IP and nodePort. 
```yaml
$ curl http://172.18.0.3:32000
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
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
$ 
```

14. Cleanup the svc and deployment
```yaml
$ kubectl delete svc nginxsvc 
$ kubectl delete deployment nginxsvc 
$ 
```

