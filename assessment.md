# Assessment 

1. Create a namespace <YOUR_NAME>

2. Create a deployment manifest file named /tmp/echoserver-<YPUR_NAME>.yaml with the following contents
``
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: hello
  name: hello
  namespace: <YOUR_NAME>
spec:
  replicas: 1
  selector:
    matchLabels:
      run: hello
  template:
    metadata:
      labels:
        run: hello
    spec:
      containers:
      - image: registry.k8s.io/echoserver:1.9
        name: hello
        ports:
        - containerPort: 8080
``

3. Apply the deployment manifest

4. Take screenshot of the deployment status using kubectl command 

5. Take screenshot of the pods running in your namespace 

The echoserver container is running in a Pod. Each Pod in Kubernetes is assigned an internal and virtual IP address at 10.xx.xx.xx. However, from outside of the cluster these IPs are not addressable, and never should be. Even within the cluster other applications normally should not attempt to address these Pods IPs. Instead each replicated Pod is fronted by a single service.  

This service can be referenced by its label, and therefore access with the help of an internal Domain Name System (DNS) that will resolve the URL to the service based on the label. The Service will add a layer of indirection where it will know how to connect to the Pod. All the other applications in the cluster will connect to the service through DNS lookups and the services will connect to the specific Pods.  

6. Expose the pod using the below command 
``
kubectl expose deployment hello -n <YOUR NAMESPACE> --type=LoadBalancer
```
7. Take screenshot of the application by accessing the External IP of the service on port 8080.

Example:
```
root@k8s:~# kubectl get svc -n john 
NAME    TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
hello   LoadBalancer   10.96.18.234   172.18.255.1   8080:31304/TCP   12s
root@k8s:~# curl 172.18.255.1:8080 


Hostname: hello-6d65ff4755-c6cws

Pod Information:
        -no pod information available-

Server values:
        server_version=nginx: 1.13.3 - lua: 10008

Request Information:
        client_address=172.18.0.2
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://172.18.255.1:8080/

Request Headers:
        accept=*/*
        host=172.18.255.1:8080
        user-agent=curl/7.81.0

Request Body:
        -no body in request-

root@k8s:~# 
```
8. Scale the number of pods from 1 to 5. 

9. Take screenshot of the 5 pods running in the cluster 

10. Update the container version to registry.k8s.io/echoserver:1.10. 

11. Take screenshot of the 5 pods running in the cluster 

12. Take screenshot of the pods running with updated version
```
kubectl describe pod hello -n <YOUR NAMESPACE> | grep "Image:"
```

13. Perform the following:  
Schedule a container using the image kubernetes-2024/webapp:2.0.0 in your namespace.  
The pod and container name should be webapp.  
It should be running in your namespace.  
The container should always be running on kind-werker node.  
The application port is 6000.  
Take screenshot of the manifest file.  

Refer the sample manifest and create a manifest for this scenario. Add/modify the configuration in the manifest to fit the requirement.  
```
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
      ports:
        - containerPort: 8080
  nodeName: node02
```

14. Take screenshot of the webapp pod running in kind-worker node.  

15. Expose the pod using below command  
```
kubectl expose po webapp -n <YOUR NAMESPACE> --type=LoadBalancer 
```

16. Take screenshot of webapp application access using curl 


