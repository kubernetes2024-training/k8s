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

#### Install docker using the below script  
```
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
.... OUTPUT TRUNCATED......
Docker Installed successfully. Run 'docker version' to verify
root@k8s:~# 
```

#### Verify docker
```
root@k8s:~/hello-world-python# systemctl status docker | grep running 
     Active: active (running) since Sat 2024-06-29 16:19:59 UTC; 22min ago
root@k8s:~/hello-world-python# 
```
#### Create a repository in Docker Hub.

![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/adaa1b87-9913-41f9-b96b-48e4db9465ce)


![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/2bcfbbcb-5b15-4acd-878d-b806c8b78d7a)


#### The files required to build the application are located in the directory hello-world-python.  
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

#### File contents 
```
root@k8s:~/hello-world-python# cat Dockerfile-red 
FROM python:3.8.5

WORKDIR /src

RUN pip install flask

COPY app-red.py ./app.py

EXPOSE 5000

CMD ["flask", "run", "--host", "0.0.0.0"]
root@k8s:~/hello-world-python#
```

```
root@k8s:~/hello-world-python# cat app-red.py 
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return '<p style="font-size: 50px; color: red;">Hello, World from Docker</p>'
root@k8s:~/hello-world-python#
```
```
root@k8s:~/hello-world-python# cat Dockerfile-blue 
FROM python:3.8.5

WORKDIR /src

RUN pip install flask

COPY app-blue.py ./app.py

EXPOSE 5000

CMD ["flask", "run", "--host", "0.0.0.0"]
root@k8s:~/hello-world-python#
```

```
root@k8s:~/hello-world-python# cat app-blue.py 
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return '<p style="font-size: 50px; color: blue;">Hello, World from Docker</p>'
root@k8s:~/hello-world-python#
```
```
root@k8s:~/hello-world-python# cat Dockerfile-green 
FROM python:3.8.5

WORKDIR /src

RUN pip install flask

COPY app-green.py ./app.py

EXPOSE 5000

CMD ["flask", "run", "--host", "0.0.0.0"]
root@k8s:~/hello-world-python# 
```
```
root@k8s:~/hello-world-python# cat app-green.py 
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return '<p style="font-size: 50px; color: green;">Hello, World from Docker</p>'
root@k8s:~/hello-world-python#
```

#### Application version 1.0.0 will be built using the app-red.py

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

#### Verify the image 
```
root@k8s:~/hello-world-python# docker image ls 
REPOSITORY              TAG       IMAGE ID       CREATED         SIZE
kubernetes2024/webapp   1.0.0     68bc0d484100   3 minutes ago   892MB
root@k8s:~/hello-world-python# 
```
```
root@k8s:~/hello-world-python# cat requirements.txt 
flask
root@k8s:~/hello-world-python#
```

#### Push the image to repository  
```root@k8s:~/hello-world-python# docker login
Username: kubernetes2024
Password: 
Login Succeeded
root@k8s:~/hello-world-python# docker push kubernetes2024/webapp:1.0.0 
The push refers to repository [docker.io/kubernetes2024/webapp]
ee9c7427c541: Pushed 
5e5de371ec56: Pushed 
546a97461f45: Pushed 
.... 
4ef54afed780: Layer already exists 
1.0.0: digest: sha256:f8aa9c0a642edd2e5d5e5b745fe06de583abb2c6f63bf10e9a163f9eb16a5d87 size: 2841
root@k8s:~/hello-world-python#
```
#### Verify that the image is uploaded to repository 

![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/c1e15ca5-a8fa-4b10-af97-5ea5cc4ccc05)

#### Start the application container 
```
root@k8s:~/hello-world-python# docker run -d --rm -p 5000:5000 kubernetes2024/webapp:1.0.0 
92478ba4676dd0e05ab0c7d5e05e87a2fc4e1b76d6e3b8eb3dd88ec3ed96cdcc
root@k8s:~/hello-world-python# docker ps 
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
92478ba4676d   kubernetes2024/webapp:1.0.0   "flask run --host 0.…"   3 seconds ago   Up 3 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   vigorous_noether
root@k8s:~/hello-world-python# 
```

#### Access the application 

![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/7bab2872-04a3-41f5-9150-847eb8bb3d3c)


Now our application is up & running, it is time for an upgrade.  

Upgrade from 1.0.0 (red) to 2.0.0 (green)  

#### Create 2.0.0 image and push to repository 
```
root@k8s:~/hello-world-python# docker build -t kubernetes2024/webapp:2.0.0 -f Dockerfile-green . 
[+] Building 0.6s (10/10) FINISHED                                                                                                           docker:default
 => [internal] load build definition from Dockerfile-green                                                                                             0.0s
root@k8s:~/hello-world-python# docker image ls | grep '2.0.0' 
kubernetes2024/webapp   2.0.0     e7c30819f0c1   18 seconds ago   892MB
root@k8s:~/hello-world-python# docker push kubernetes2024/webapp:2.0.0
The push refers to repository [docker.io/kubernetes2024/webapp]
c4f42028fc42: Pushed 
5e5de371ec56: Layer already exists 
2.0.0: digest: sha256:6626c840327c7fa87d818e0ddb6bdbc5a7d407d4f838111a3b4aa21a275ae6f2 size: 2841
root@k8s:~/hello-world-python#
```

#### Verify in repository  
![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/f4a8ba34-d3ca-439b-ae86-3ff7cf43de03)

#### Upgrade the application (stop/remove older version of application & start with new version)
```
root@k8s:~/hello-world-python# docker ps | grep 'kubernetes2024/webapp:1.0.0' 
92478ba4676d   kubernetes2024/webapp:1.0.0   "flask run --host 0.…"   18 minutes ago   Up 18 minutes   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   vigorous_noether
root@k8s:~/hello-world-python# docker stop 92478ba4676d 
92478ba4676d
root@k8s:~/hello-world-python# docker ps | grep 'kubernetes2024/webapp:1.0.0' 
root@k8s:~/hello-world-python#

root@k8s:~/hello-world-python# docker run -d --rm -p 5000:5000 kubernetes2024/webapp:2.0.0 
7592c979e27467e03542c1dbd72510b83784e2c3e84e13582b24202b9dd62879
root@k8s:~/hello-world-python# docker ps | grep 'kubernetes2024/webapp:2.0.0' 
7592c979e274   kubernetes2024/webapp:2.0.0   "flask run --host 0.…"   8 seconds ago   Up 8 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   festive_hermann
root@k8s:~/hello-world-python#
```

#### Access the application
![image](https://github.com/kubernetes2024-training/k8s/assets/174179573/f93c4ad9-6293-4a71-bb31-1f76ab88edb8)















