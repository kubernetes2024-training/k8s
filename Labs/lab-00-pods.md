
# Kubernetes Fundamentals: Pods

There are about 55 standard resources in Kubernetes that you can control. The first and primary resource you will most likely encounter is a Pod. Pods provide context—such as access to the filesystem and the network on a hosting node—for one or more containers. Because Kubernetes is a container orchestration system, it makes sense that its most important and feature-rich resource is a Pod.

This lab introduces you to Pods and explores some important Pod usage techniques and features.

In this lab, you will learn how to:

    ☐ Create, read, update, and delete a Pod
    ☐ Define a container in a Pod
    ☐ Access the network and filesystem context of a Pod
    ☐ Fix some common problems with starting a Pod

### Creating a Pod

The simplest way to create a Pod is via the imperative kubectl run command. For example, to run our kuard server, you can use:
```
kubectl run kuard-<YOUR NAME> --image=gcr.io/kuar-demo/kuard-amd64:blue
```
You can see the status of this Pod by running:
```
kubectl get pods
```
For now, you can delete this Pod by running:
```
kubectl delete pods/kuard-<YOUR NAME>
```
### Creating a Pod Manifest

You can write a Pod manifest using YAML or JSON, but YAML is generally preferred because it is slightly more human editable and provides the ability to add comments. Pod manifests (and other Kubernetes API objects) should really be treated in the same way as source code, and things like comments help explain the Pod to new team members.

Pod manifests include a couple of key fields and attributes: namely, a metadata section for describing the Pod and its labels, a spec section for describing volumes, and a list of containers that will run in the Pod.

Without Kubernetes, the kuard application can be started using the following docker command:
```
docker run -d \
  --name kuard \
  --publish 8080:8080 \
  gcr.io/kuar-demo/kuard-amd64:blue
```

On Kubernetes, you can achieve a similar result by writing a Pod manifest to a file. The following command will output the YAML shown to the file kuard-pod.yaml:
```
cat << 'EOF' > kuard-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: kuard-<YOUR NAME>
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard-<YOUR NAME>
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
EOF
```

The file is a Kubernetes YAML manifest that represents a Pod resource:
```
less kuard-pod-<YOUR NAME>.yaml
```
It may initially seem more cumbersome to manage your application in this manner. However, this written record of desired state is the best practice in the long run, especially for large teams with many applications. These manifests help you follow the notion of Infrastructure as Code (IaC) and the practices around GitOps.

### Running Pods
So far in this step of the lab, you've created a Pod manifest that can be used to start a Pod running kuard. Use the kubectl apply command to launch a single instance of kuard:
```
kubectl apply -f kuard-pod.yaml
```
The Pod manifest will be submitted to the Kubernetes API server. The Kubernetes system will then schedule that Pod to run on a healthy node in the cluster, where the kubelet daemon will monitor it.

### Listing Pods
Now that you have a Pod running, let’s go find out some more about it. Using the kubectl command-line tool, list all Pods running in the cluster in the default namespace:
```
kubectl get pods
```
For now, this should only be the single Pod that was created earlier:

NAME       READY     STATUS    RESTARTS   AGE
kuard-<YOUR NAME>      1/1       Running   0          44s
You can see the name of the Pod (kuard) that was declared in the YAML file. In addition to the number of ready containers (1/1), the output also shows the status, the number of times the Pod was restarted, and the age of the Pod.

If you ran this command immediately after the Pod was created, you might see:

NAME       READY     STATUS    RESTARTS   AGE
kuard-<YOUR NAME>      0/1       Pending   0          1s
The Pending state indicates that the Pod has been submitted but hasn’t been scheduled yet.

If a more significant error occurs, such as an attempt to create a Pod with a container image that doesn’t exist, it will also be listed in the STATUS field.

By default, the kubectl command-line tool is concise in the information it reports, but you can get more information via command-line flags. Adding -o wide to any kubectl command will print out slightly more information (while still keeping the information to a single line). Adding -o json or -o yaml will print out the complete objects in JSON or YAML, respectively:
```
kubectl get pods -o yaml
```
If you ever want to see an exhaustive, verbose logging of what kubectl is doing, you can add the --v=10 flag for comprehensive logging at the expense of readability:
```
kubectl get pods --v=10
```

### Pod Details
Sometimes the single-line view is insufficient because it is too terse. Additionally, Kubernetes maintains numerous events about Pods that are present in the event stream, not attached to the Pod object.

To find out more information about a Pod (or any Kubernetes object), you can use the kubectl describe command. For example, to describe the Pod previously created, you can run:
```
kubectl describe pods kuard-<YOUR NAME>
```
This outputs a bunch of information about the Pod in different sections. At the top is basic information about the Pod.

### Deleting a Pod
When it is time to delete a Pod, you can do so either by name:
```
kubectl delete pods/kuard-<YOUR NAME>
```
or by using the file that you used to create it:
```
kubectl delete -f kuard-pod-<YOUR NAME>.yaml
```
When a Pod is deleted, it is not immediately killed. Instead, if you run:
```
kubectl get pods
```
you may see that the Pod is in the Terminating state. All Pods have a termination grace period. By default, this is 30 seconds. When a Pod is transitioned to Terminating, it no longer receives new requests. In a serving scenario, the grace period is important for reliability because it allows the Pod to finish any active requests that it may be in the middle of processing before it is terminated.

When you delete a Pod, any data stored in the containers associated with that Pod will be deleted as well. If you want to persist data across multiple instances of a Pod, you need to use PersistentVolumes, described in the lab Kubernetes Fundamentals: Volumes and Mounts.

### Accessing Pods

Now that your Pod is running, you’re going to want to access it for a variety of reasons. You may want to load the web service that is running in the Pod. You may want to view its logs to debug a problem that you are seeing, or even execute other commands inside the Pod to help debug. The following sections detail various ways you can interact with the code and data running inside your Pod.

Using Port Forwarding
To achieve this, you can use the port-forwarding support built into the Kubernetes API and command-line tools. Let's restart the kuard Pod:
```
kubectl apply -f kuard-pod-<YOUR NAME>.yaml
```
When you run:
```
kubectl port-forward kuard-<YOUR NAME> <EPHEMERAL_PORT>:8080 &
```
a secure tunnel is created from your local machine, through the Kubernetes master, to the instance of the Pod running on one of the worker nodes. The & symbol at the end runs the port forwarding as a background process so you can continue with foreground commands.

Note: Replace <EPHEMERAL_PORT> with a random port number in the range of 49152-65535.  


As long as the port-forward command is still running, you can access the Pod (in this case the kuard web interface) at:
```
curl http://localhost:8080
```

### Getting More Info with Logs
When your application needs debugging, it’s helpful to be able to dig deeper than describe to understand what the application is doing. Kubernetes provides two commands for debugging running containers. The kubectl logs command downloads the current logs from the running instance:
```
kubectl logs kuard-<YOUR NAME>
```
Adding the -f flag will cause you to continuously stream logs.

The kubectl logs command always tries to get logs from the currently running container. Adding the --previous flag will get logs from a previous instance of the container. This is useful, for example, if your containers are continuously restarting due to a problem at container startup.

While using kubectl logs is useful for occasional debugging of containers in production environments, it’s generally helpful to use a log aggregation service. There are several open source log aggregation tools, like fluentd and elasticsearch, as well as numerous cloud logging providers. These log aggregation services provide greater capacity for storing a longer duration of logs, as well as rich log searching and filtering capabilities. Many also provide the ability to aggregate logs from multiple Pods into a single view.

### Running Commands in Your Container with exec
Sometimes logs are insufficient, and to truly determine what’s going on, you need to execute commands in the context of the container itself. To do this, you can use:
```
kubectl exec kuard-<YOUR NAME> -- date; ll /
```
You can also get an interactive session by adding the -it flags:
```
kubectl exec -it kuard-<YOUR NAME> -- sh
```
Notice the prompt changed, as you are now inside a shell in the container in the Pod. Not all containers may have a shell available. To exit the shell, type 
```
exit
```

### Copying Files to and from Containers
At times you may need to copy files from a remote container to a local machine for more in-depth exploration. For example, you can use a tool like Wireshark to visualize tcpdump packet captures. Suppose you had a file /captures/capture3.txt inside a container in your Pod. You could securely copy that file to your local machine by running:
```
kubectl cp <pod-name>:/captures/capture3.txt ./capture3.txt
```
Other times you may need to copy files from your local machine into a container. Let’s say you want to copy $HOME/config.txt to a remote container. In this case, you can run:
```
kubectl cp $HOME/config.txt kuard-<YOUR NAME>:/tmp/config.txt
```
With the file copied, confirm it's now in the container:
```
kubectl exec kuard-<YOUR NAME> -- ls -ls /tmp
```
Generally speaking, copying files into a container is an antipattern. You really should treat the contents of a container as immutable. But occasionally it’s the most immediate way to stop the bleeding and restore your service to health, since it is quicker than building, pushing, and rolling out a new image. Once you stop the bleeding, however, it is critically important that you immediately go and do the image build and rollout, or you are guaranteed to forget the local change that you made to your container and overwrite it in the subsequent regularly scheduled rollout.

# Kubernetes: Create and Inspect a Pod

A Pod is the most essential and important primitive in Kubernetes. In this lab, you will practice its creation in a namespace, inspect it, modify it, interact with it, and, later on, delete it. You will practice the relevant kubectl commands by seeing them in action.

In this lab, you will:

Create a new namespace.
Create a new Pod in the namespace.
Inspect the Pod.
Log into the container.
Delete the Pod and namespace.

## Creating a Namespace
Many problems in the exam need to be solved in a specific namespace. Namespaces are intended for use in environments with many users spread across multiple teams, or projects as a means to separate Kubernetes resources. You'll start by creating a new namespace with the name business-<YOUR NAME>.

## Imperative Namespace Creation
To create a new namespace, run the create namespace command. Remember that you can also use the create ns short form to save some typing.
```
kubectl create namespace business-<YOUR NAME>
```

### Viewing Namespaces
Ensure that the namespace was created properly with the get namespace command.
```
kubectl get namespaces
```
You should see the namespace business listed in the terminal output. In the next step, you will create a new Pod in the namespace business.


### Creating a Pod

A Pod is the Kubernetes resource that allows us to run an application in a container.

### Imperative Pod Creation
The Pod you are about to create should have the name mypod, use the image nginx:2.3.5, and expose the container port 80. You should make sure to put the Pod into the namespace business. The run command is a fast and convenient way to create a new Pod because you don't have to edit any YAML definition.
```
kubectl run mypod-<YOUR NAME> --image=nginx:2.3.5 --restart=Never --port=80 --namespace=business-<YOUR NAME>
```
After running the command, the console output should indicate that the Pod was created.

### Viewing Pods
Make sure that the Pod runs without issues by listing all Pods in the namespace. The command below uses the short form -n to indicate the namespace you are querying.
```
kubectl get pod -n business-<YOUR NAME>
```
The list of Pods renders the current status of each Pod available in the namespace. Upon further inspection, you will notice that the status is ErrImagePull or ImagePullBackOff. This status means that the image couldn't be resolved or downloaded from the container registry.

### Describing the Pod
The list of events can give you deeper insight. Use the describe pod command for more information.
```
kubectl describe pod -n business-<YOUR NAME>
```
At the bottom of the terminal output, you will find the message Error response from daemon: manifest for nginx:2.3.5 not found. Open a browser and try to find the image for the specified tag on Docker Hub. Apparently, the tag hasn't been published. In the next step, you will ensure a successful startup of the Pod by setting a different, existent tag.

### Modifying the Pod

Go ahead and edit the existing Pod. For now, we just want to set a different image.

### Setting a Different Image
The kubectl executable provides a specific command for this: set image. The command works on a specific label. In our case, the name of the Pod is mypod and the label as well. The value for the label should be the new tag for the image. Let's set it to nginx, which implies that we are using the latest tag.
```
kubectl set image pod mypod-<YOUR NAME> mypod=nginx --namespace=business-<YOUR NAME>
```
Checking the Pod Status
After setting an image that does exist, the Pod should render the status Running. Give it some time to allow Kubernetes to restart the container.
```
kubectl get pod -n business-<YOUR NAME>
```
Awesome, you fixed the underlying issue that prevented the Pod from starting up properly. Next, you will log into the container and inspect the Pod even further by rendering more details.

### Interacting with the Pod

Congratulations, the Pod is up and running. In this step, you will further interact with the Pod.

### Shelling into the Container
You can shell into a Pod using the exec command to inspect the application runtime environment. Execute the following command to log into the Pod mypod:
```
kubectl exec mypod-<YOUR NAME> -it --namespace=business-<YOUR NAME>  -- /bin/sh
```
After running the command, you can see that the terminal prompt changed. The pound sign (#) signifies that you are now operating inside of the Pod's container.

Let's explore the current directory by listing all of its files and subdirectories.
```
ls
```
To exit the container environment, run the exit command. The terminal prompt changes back to what it was before.
```
exit
```
### Sending a Request to Nginx Running in Container
Ngnix is a proxy web server that we have running inside of the Pod. Let's call the service to ensure it's running properly. Retrieve the IP address of the Pod with the -o wide command-line option. You can find the information under the column labeled IP.
```
kubectl get pods -o wide -n business-<YOUR NAME>
```
This IP address is only accessible from within the Kubernetes cluster. To make a request to nginx, we'll create a temporary Pod running inside of the same node. By using the --rm command-line option, we tell Kubernetes to delete the Pod after exiting the container. The Pod uses the image busybox.
```
kubectl run busybox --image=busybox --rm -it --restart=Never -n business-<YOUR NAME> -- /bin/sh
```
From within the container, run the command wget -O- <ip>:80. Remember to use the IP address from the Pod running nginx. You should see the HTML output of the response rendered in the terminal.

Exit the container by running the exit command. You will see that Kubernetes automatically deletes the busybox Pod.
```
exit
```
Accessing Pod Logs
You can access the log files of the Pod with logs command. The following command renders a line for each time you send a HTTP request to nginx.
```
kubectl logs mypod-<YOUR NAME> -n business-<YOUR NAME>
```
We're all done. In the next step, we'll simply clean up the Kubernetes resources.


### Deleting the Resources

It's time to clean up the Kubernetes resources you created in this lab. First, run the delete command for the Pod. Remember to provide the namespace.
```
kubectl delete pod mypod-<YOUR NAME> --namespace=business-<YOUR NAME>
```
Next, run a similar command to delete the namespace.
```
kubectl delete namespace business-<YOUR NAME>
```
Kubernetes tries to gracefully delete the resources, which can sometimes take a couple of seconds. You can speed up the process by providing the command-line option --grace-period=0 --force. Those command-line options may come in handy if you do not want to waste precious seconds on Kubernetes' deletion process.

Finally, ensure that the Kubernetes resources have been deleted properly.
```
kubectl get pod,namespace
```


