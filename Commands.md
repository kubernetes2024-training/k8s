### Pods 

Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace  
```
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

Create the pod that was just described using YAML  
```
kubectl run nginx --image=nginx --restart=Never --dry-run=client -n mynamespace -o yaml > pod.yaml

cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: mynamespace
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

kubectl create -f pod.yaml
```

Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output  
```
kubectl run busybox --image=busybox --command --restart=Never -it --rm -- env
```

Create a pod with image nginx called nginx and expose traffic on port 80  
```
kubectl run nginx --image=nginx --restart=Never --port=80
```

Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'  
```
kubectl get po -o wide # get the IP, will be something like '10.1.1.131'
# create a temp busybox pod
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Get information about the pod, including details about potential issues (e.g. pod hasn't started)  
```
kubectl describe po nginx
```

Get pod logs  
```
kubectl logs nginx
```

Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod  
```
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe po nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
# or
kubectl run nginx --image nginx --restart=Never --env=var1=val1 -it --rm -- sh -c 'echo $var1'
```

### Labels and Annotations
Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1  
```
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```

Show all labels of the pods  
```
kubectl get po --show-labels
```

Change the labels of pod 'nginx2' to be app=v2  
```
kubectl label po nginx2 app=v2 --overwrite
```

Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value  
```
kubectl annotate po nginx1 nginx2 nginx3 description='my description'
```

Check the annotations for pod nginx1  
```
kubectl annotate pod nginx1 --list
```

Remove the annotations for these three pods  
```
kubectl annotate po nginx{1..3} description- owner-
```

### Pod Placement
Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'  
```
kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels

apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
  nodeSelector: # add this
    accelerator: nvidia-tesla-p100 # the selection label
```

### Deployments
Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)  
```
kubectl create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

Check how the deployment rollout is going  
```
kubectl rollout status deploy nginx
```

Update the nginx image to nginx:1.19.8  
```
kubectl set image deploy nginx nginx=nginx:1.19.8
```

Check the rollout history and confirm that the replicas are OK  
```
kubectl rollout history deploy nginx
```

Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)  
```
kubectl rollout undo deploy nginx
```

Do an on purpose update of the deployment with a wrong image nginx:1.91  
```
kubectl set image deploy nginx nginx=nginx:1.91
```

Verify that something's wrong with the rollout
```
kubectl rollout status deploy nginx
```

Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8  
```
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx
```

Scale the deployment to 5 replicas  
```
kubectl scale deploy nginx --replicas=5
```

### Resource requests and limits
Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi  
```
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

vi pod.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:    
        memory: "512Mi"
        cpu: "200m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### ServiceAccounts

Create a new serviceaccount called 'myuser'  
```
kubectl create sa myuser
```




