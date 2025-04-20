
# Tutorial: Deploying a Static Website on DigitalOcean Kubernetes


Containerize & Deploy a static website on DigitalOcean's managed Kubernetes cluster. This repository provides a step-by-step tutorial, complete with code examples and configuration files, to help you get started with containerization and Kubernetes.

# Prerequisite
 - Any Linux virtual machine running Ubuntu.
 - Docker installed and running. If needed refer [here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
 - DockerHub id. if not already, please refer [here](https://docs.docker.com/accounts/create-account/)
 - git client installed
 - static website files. Feel free to use to sample in this repo [website-files](https://github.com/vinayvtiwari/simple-web-app-do/tree/main/website-files)
 - DigitalOcean login id with relevant permissions.


# Table of Content

01. Static Website Containerization with Docker
02. Publishing Your Docker Image to Docker Hub
03. Creating a Managed Digital Ocean Kubernetes Cluster(DOKS)
04. Install & configure Digital Ocean CLI(DOCTL) & Kubernetes CLI (Kubectl)
05. Kubernetes Fundamentals: Deploying Pods, ReplicaSets, Deployments, and LoadBalancer Service Type
07. Monitoring Your Cluster: Installing Metrics Server on DOKS
08. Scaling Your Application: Understanding and Deploying Horizontal Pod Autoscaler (HPA)

# Step 01 - Static Website Containerization with Docker

 - **Clone this repository locally**

 ```shell
 vinayti@osboxes:~$ git clone https://github.com/vinayvtiwari/simple-web-app-do.git
Cloning into 'simple-web-app-do'...
remote: Enumerating objects: 34, done.
remote: Counting objects: 100% (34/34), done.
remote: Compressing objects: 100% (30/30), done.
remote: Total 34 (delta 7), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (34/34), 132.57 KiB | 1.56 MiB/s, done.
Resolving deltas: 100% (7/7), done.
vinayti@osboxes:~$
vinayti@osboxes:~$ cd simple-web-app-do/
vinayti@osboxes:~/simple-web-app-do$ ls
k8s-files  LICENSE  README.md  website-files
vinayti@osboxes:~/simple-web-app-do$

```
- **CD into the website-files directory and examine the content.**

```shell
vinayti@osboxes:~/simple-web-app-do$ cd website-files/
vinayti@osboxes:~/simple-web-app-do/website-files$ ls -l
total 140
-rw-rw-r-- 1 vinayti vinayti     86 Apr 20 10:54 Dockerfile
-rw-rw-r-- 1 vinayti vinayti 111884 Apr 20 10:27 do-kube-image.png
-rw-rw-r-- 1 vinayti vinayti  16512 Apr 20 10:27 do-logo.png
-rw-rw-r-- 1 vinayti vinayti   3839 Apr 20 10:27 index.html
vinayti@osboxes:~/simple-web-app-do/website-files$
```
Except Dockerfile , all the remaining files are website related files which is what we want to dockerize. Lets understand it first.

Dockerfile --> A Dockerfile is a text file that contains instructions for building a Docker image. It's a blueprint for creating a Docker image, specifying the base image, dependencies, and commands to run.

```shell
vinayti@osboxes:~/simple-web-app-do/website-files$ cat Dockerfile
FROM nginx:latest
COPY index.html do-kube-image.png do-logo.png /usr/share/nginx/html
vinayti@osboxes:~/simple-web-app-do/website-files$
```

- **Understanding both the lines**  
```shell 
FROM nginx:latest
``` 
Tells us to use nginx as a base image. It will be downloaded from DockerHub. Its a cloud-based registry service provided by Docker that allows users to store, manage, and share Docker images. It provides a centralized location for Docker users to find and share container images. More more info refer [here](https://hub.docker.com/_/nginx)

```shell 
COPY index.html do-kube-image.png do-logo.png /usr/share/nginx/html
```
Copies index.html, do-kube-image.png, and do-logo.png files from the current directory (on the ubuntu machine) into the Docker image


- **Building our own image**
We did not specify the docker file, because the .(dot) at the end means, the Dockerfile is in the current folder. we using the name in the format of **[DOCKER_USERNAME/IMAGE_NAME_YOU_WANT_TO_KEEP]**. It is required because next we will also push it to the dockerhub  so that it can be dowloaded and utilized within the kubernetes environment.

```shell
vinayti@osboxes:/home/vinayti/website-files# docker build -t vinayti/simple-website-demo .
[+] Building 0.5s (7/7) FINISHED                                                     docker:default
 => [internal] load build definition from Dockerfile                                           0.1s
 => => transferring dockerfile: 123B                                                           0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                0.0s
 => [internal] load .dockerignore                                                              0.1s
 => => transferring context: 2B                                                                0.0s
 => [internal] load build context                                                              0.1s
 => => transferring context: 102B                                                              0.0s
 => [1/2] FROM docker.io/library/nginx:latest                                                  0.0s
 => CACHED [2/2] COPY index.html do-kube-image.png do-logo.png /usr/share/nginx/html           0.0s
 => exporting to image                                                                         0.1s
 => => exporting layers                                                                        0.0s
 => => writing image sha256:2e8b08ea0672aa85c5f99e9ecd0513f1d64c569df1b04f2f8512ba76a73d4787   0.0s
 => => naming to docker.io/vinayti/simple-website-demo                                         0.0s
vinaytit@osboxes:/home/vinayti/website-files#
```
- ** Check whether the image is build using the below docker command**

```shell

vinayti@osboxes:~/simple-web-app-do/website-files$ sudo docker images
REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
vinayti/simple-website-demo   latest    3ec382e7dd14   28 seconds ago   193MB
vinayti@osboxes:~/simple-web-app-do/website-files$
```

# Step 02 -  Publishing Your Docker Image to Docker Hub
- **Authenticate to DockerHub using docker id**

```shell
vinayti@osboxes:~/simple-web-app-do/website-files$ docker login -u vinayti

i Info → A Personal Access Token (PAT) can be used instead.
         To create a PAT, visit https://app.docker.com/settings


Password:

WARNING! Your credentials are stored unencrypted in '/home/vinayti/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
vinayti@osboxes:~/simple-web-app-do/website-files$
```

- **Push the image to the DockerHub repository**

```shell
vinayti@osboxes:~/simple-web-app-do/website-files$ sudo docker push vinayti/simple-website-demo:latest
The push refers to repository [docker.io/vinayti/simple-website-demo]
a96335b001f0: Mounted from vinayti/simple-website-demo
d1e3e4dd1aaa: Mounted from vinayti/simple-website-demo
ccc5aac17fc4: Mounted from vinayti/simple-website-demo
8d83f6b79143: Mounted from vinayti/simple-website-demo
9e3c6e8c1e25: Mounted from vinayti/simple-website-demo
9aad78ecf380: Mounted from vinayti/simple-website-demo
bd903131a05e: Mounted from vinayti/simple-website-demo
ea680fbff095: Mounted from vinayti/simple-website-demo
latest: digest: sha256:59e17eea6ca66392f78ce0dedf2edaa6ff35e049aca3c699d9bc1c5566ded450 size: 1988
vinayti@osboxes:~/simple-web-app-do/website-files$
```
The image should now be visible in the docker hub under your account. It can now be utilized by anyone on the internet

# Step 03 - Creating a Managed Digital Ocean Kubernetes Cluster(DOKS)

- Login to DigitalOcean [here](https://cloud.digitalocean.com/login)

- Click Create Button on top right and select Kubernetes.

  ![image](https://github.com/user-attachments/assets/122a739d-9302-4d02-9de7-4e125cc2dd58)

- Select the latest version. Select appropriate datacenter region & keep the VPC Setting as is.

  ![image](https://github.com/user-attachments/assets/726e0271-49b4-4648-b0ee-d0c7bc225252)


  

- 


