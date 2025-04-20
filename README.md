
# Tutorial: Deploying a Static Website on DigitalOcean Kubernetes


Containerize & Deploy a static website on DigitalOcean's managed Kubernetes cluster. This repository provides a step-by-step tutorial, complete with code examples and configuration files, to help you get started with containerization and Kubernetes.

# Prerequisite
 - Any Linux virtual machine running Ubuntu.
 - Docker installed and running. If needed refer [here](https://docs.docker.com/engine/install/ubuntu/)
 - git client installed
 - static website files. Feel free to use to sample in this repo [website-files](https://github.com/vinayvtiwari/simple-web-app-do/tree/main/website-files)


# Table of Content

01. Static Website Containerization with Docker
02. Publishing Your Docker Image to Docker Hub
03. Creating a Managed Digital Ocean Kubernetes Cluster(DOKS)
04. Install & configure Digital Ocean CLI(DOCTL) & Kubernetes CLI (Kubectl)
05. Kubernetes Fundamentals: Deploying Pods, ReplicaSets, Deployments, and LoadBalancer Service Type
07. Monitoring Your Cluster: Installing Metrics Server on DOKS
08. Scaling Your Application: Understanding and Deploying Horizontal Pod Autoscaler (HPA)

# Step 01 - Static Website Containerization with Docker

 - Clone this repository locally

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
- CD into the website-files directory and examine the content.

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
