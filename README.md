# Simple webapp deployment using docker 

This setup is done on an Ubuntu LTS 22.04 server in Google Cloud 

```
root@k8s:~# cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.4 LTS"
root@k8s:~#
```

Install docker using the below script  
``
root@k8s:~# cat docker-install.sh 
#!/bin/bash

echo "Running update and installing dependecies...."
sudo apt update
sudo apt install ca-certificates curl apt-transport-https -y

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg


echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "Ïnstalling Docker..."
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose -y

sudo systemctl start docker

sudo systemctl enable docker

[ `! grep -q 'docker:' /etc/group` ] && sudo newgrp docker
sudo usermod -aG docker $USER

echo "Docker Installed successfully. Run 'docker version' to verify"
root@k8s:~# 

root@k8s:~# ./docker-install.sh 
Running update and installing dependecies....
Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu jammy-backports InRelease [127 kB]
Get:4 http://security.ubuntu.com/ubuntu jammy-security InRelease [129 kB]     
.... OUTPUT TRUNCATED......
...
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable docker
Docker Installed successfully. Run 'docker version' to verify
root@k8s:~# 
```

Verify docker
```
root@k8s:~# docker version 
Client: Docker Engine - Community
 Version:           27.0.2
 API version:       1.46
 Go version:        go1.21.11
 Git commit:        912c1dd
 Built:             Wed Jun 26 18:47:16 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          27.0.2
  API version:      1.46 (minimum version 1.24)
  Go version:       go1.21.11
  Git commit:       e953d76
  Built:            Wed Jun 26 18:47:16 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.18
  GitCommit:        ae71819c4f5e67bb4d5ae76a6b735f29cc25774e
 runc:
  Version:          1.7.18
  GitCommit:        v1.1.13-0-g58aa920
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
root@k8s:~# systemctl status docker 
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-06-29 16:19:59 UTC; 34s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 3215 (dockerd)
      Tasks: 9
     Memory: 22.5M
        CPU: 489ms
     CGroup: /system.slice/docker.service
             └─3215 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jun 29 16:19:58 k8s systemd[1]: Starting Docker Application Container Engine...
Jun 29 16:19:58 k8s dockerd[3215]: time="2024-06-29T16:19:58.375829086Z" level=info msg="Starting up"
Jun 29 16:19:58 k8s dockerd[3215]: time="2024-06-29T16:19:58.377528924Z" level=info msg="detected 127.0.0.53 nameserver, assuming systemd-resolved, so usin>
Jun 29 16:19:59 k8s dockerd[3215]: time="2024-06-29T16:19:59.215277664Z" level=info msg="Loading containers: start."
Jun 29 16:19:59 k8s dockerd[3215]: time="2024-06-29T16:19:59.694773365Z" level=info msg="Loading containers: done."
Jun 29 16:19:59 k8s dockerd[3215]: time="2024-06-29T16:19:59.723767476Z" level=info msg="Docker daemon" commit=e953d76 containerd-snapshotter=false storage>
Jun 29 16:19:59 k8s dockerd[3215]: time="2024-06-29T16:19:59.724163944Z" level=info msg="Daemon has completed initialization"
Jun 29 16:19:59 k8s dockerd[3215]: time="2024-06-29T16:19:59.800024421Z" level=info msg="API listen on /run/docker.sock"
Jun 29 16:19:59 k8s systemd[1]: Started Docker Application Container Engine.
root@k8s:~# 
```
Create a repository in Docker Hub.

![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/adaa1b87-9913-41f9-b96b-48e4db9465ce)


![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/2bcfbbcb-5b15-4acd-878d-b806c8b78d7a)


The files required to build the application are located in the directory hello-world-python.  
```
root@k8s:~/hello-world-python# tree 
.
├── Dockerfile-blue
├── Dockerfile-green
├── Dockerfile-red
├── app-blue.py
├── app-green.py
├── app-red.py
└── requirements.txt

0 directories, 7 files
root@k8s:~/hello-world-python#
```


Application version 1.0.0 will be built using the app-red.py

```root@k8s:~/hello-world-python# docker build -t kubernetes2024/webapp:1.0.0 -f Dockerfile-red .
[+] Building 24.3s (4/8)                                                                                                                     docker:default
 => [internal] load build definition from Dockerfile-red                                                                                               0.0s
 => => transferring dockerfile: 180B                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/python:3.8.5                                                                                        0.8s
 => [internal] load .dockerignore
....OUTPUT TRUNCATED....
 => exporting to image                                                                                                                                 0.5s 
 => => exporting layers                                                                                                                                0.5s
 => => writing image sha256:68bc0d4841007d173c9016a091c0e6ad646b22ee710e814e35b12a91edfea86e                                                           0.0s
 => => naming to docker.io/kubernetes2024/webapp:red                                                                                                   0.0s
root@k8s:~/hello-world-python#
```

Verify the image 
```
root@k8s:~/hello-world-python# docker image ls 
REPOSITORY              TAG       IMAGE ID       CREATED         SIZE
kubernetes2024/webapp   1.0.0     68bc0d484100   3 minutes ago   892MB
root@k8s:~/hello-world-python# 
```

Push the image to repository  
```root@k8s:~/hello-world-python# docker login
Log in with your Docker ID or email address to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com/ to create one.
You can log in with your password or a Personal Access Token (PAT). Using a limited-scope PAT grants better security and is required for organizations using SSO. Learn more at https://docs.docker.com/go/access-tokens/

Username: kubernetes2024
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credential-stores

Login Succeeded
root@k8s:~/hello-world-python# docker push kubernetes2024/webapp:1.0.0 
The push refers to repository [docker.io/kubernetes2024/webapp]
ee9c7427c541: Pushed 
5e5de371ec56: Pushed 
546a97461f45: Pushed 
dd275046203b: Layer already exists 
48fd0662e424: Layer already exists 
817923b47cba: Layer already exists 
0fb2e27dc3b8: Layer already exists 
a995c5106335: Layer already exists 
17bdf5e22660: Layer already exists 
d37096232ed8: Layer already exists 
6add0d2b5482: Layer already exists 
4ef54afed780: Layer already exists 
1.0.0: digest: sha256:f8aa9c0a642edd2e5d5e5b745fe06de583abb2c6f63bf10e9a163f9eb16a5d87 size: 2841
root@k8s:~/hello-world-python#
```
Verify that the image is uploaded to repository 

![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/c1e15ca5-a8fa-4b10-af97-5ea5cc4ccc05)

Start the application container 
```
root@k8s:~/hello-world-python# docker run -d --rm -p 5000:5000 kubernetes2024/webapp:1.0.0 
92478ba4676dd0e05ab0c7d5e05e87a2fc4e1b76d6e3b8eb3dd88ec3ed96cdcc
root@k8s:~/hello-world-python# docker ps 
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
92478ba4676d   kubernetes2024/webapp:1.0.0   "flask run --host 0.…"   3 seconds ago   Up 3 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   vigorous_noether
root@k8s:~/hello-world-python# 
```

Access the application 

![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/d2b5a051-0598-4038-aae4-aa1450c0795b)










