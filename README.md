
# Tutorial: Deploying a Static Website on DigitalOcean Managed Kubernetes Cluster


Containerize & Deploy a static website on DigitalOcean's managed Kubernetes cluster. This repository provides a step-by-step tutorial, complete with code examples and configuration files, to help you get started with containerization and Kubernetes.

# Prerequisite
 - Any Linux virtual machine running Ubuntu.
 - Docker installed and running. If not already, refer [here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
 - DockerHub id. if not already, please refer [here](https://docs.docker.com/accounts/create-account/)
 - git client installed
 - static website files. Feel free to use to sample in this repo [website-files](https://github.com/vinayvtiwari/simple-web-app-do/tree/main/website-files)
 - DigitalOcean login id with relevant permissions.


# Table of Content

01. [Static Website Containerization with Docker](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-01---static-website-containerization-with-docker)
02. [Publishing Your Docker Image to Docker Hub](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-02----publishing-your-docker-image-to-docker-hub)
03. [Creating a Managed Digital Ocean Kubernetes Cluster(DOKS)](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-03---creating-a-managed-digital-ocean-kubernetes-clusterdoks)
04. [Install & configure Digital Ocean CLI(DOCTL), API Token & Kubernetes CLI (Kubectl)](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-04---install--configure-digital-ocean-clidoctl-api-token--kubernetes-cli-kubectl)
05. [Kubernetes Fundamentals: Deploying Pods, ReplicaSets, Declarative definitions, Deployments, and LoadBalancer Service Type](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-05---kubernetes-fundamentals-deploying-pods-replicasets-declarative-definitions-deployments-and-loadbalancer-service-type)
06. [Monitoring Your Cluster: Installing Metrics Server on DOKS](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-06---monitoring-your-cluster-installing-metrics-server-on-doks)
07. [Scaling Your Application: Understanding and Deploying Horizontal Pod Autoscaler (HPA)](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-07---scaling-your-application-understanding-and-deploying-horizontal-pod-autoscaler-hpa)
08. [Architechture Diagram Depicting the above configuration](https://github.com/vinayvtiwari/simple-web-app-do/blob/main/README.md#step-08---architechture-diagram-depicting-the-above-configuration)

# Step 01 - Static Website Containerization with Docker

 - **Clone this repository locally**

```shell
git clone https://github.com/vinayvtiwari/simple-web-app-do.git
```
output:
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
Except Dockerfile , all the remaining files are website related which is what we want to dockerize. Lets understand it first.

Dockerfile --> A Dockerfile is a text file that contains instructions for building a Docker image. It's a blueprint for creating a Docker image, specifying  the base image, dependencies, and commands to run.

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
docker build -t vinayti/simple-website-demo .
```

output:
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
- **Check whether the image is build using the below docker command**

```shell
sudo docker images
```
output:
```shell

vinayti@osboxes:~/simple-web-app-do/website-files$ sudo docker images
REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
vinayti/simple-website-demo   latest    3ec382e7dd14   28 seconds ago   193MB
vinayti@osboxes:~/simple-web-app-do/website-files$
```

# Step 02 -  Publishing Your Docker Image to Docker Hub
- **Authenticate to DockerHub using docker id**

```shell
docker login -u <dockerhub username>
```

output:
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
sudo docker push vinayti/simple-website-demo:latest
```

output:
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

![image](https://github.com/user-attachments/assets/d9f248ef-8360-4f48-afe8-bd3c6db8183b)


# Step 03 - Creating a Managed Digital Ocean Kubernetes Cluster(DOKS)

- **Login to DigitalOcean [here](https://cloud.digitalocean.com/login)**

- **Click Create Button on top right and select Kubernetes.**
  
  ![image](https://github.com/user-attachments/assets/122a739d-9302-4d02-9de7-4e125cc2dd58)

- **Select the latest version. Select appropriate datacenter region & keep the VPC Setting as is.**

  ![image](https://github.com/user-attachments/assets/726e0271-49b4-4648-b0ee-d0c7bc225252)

- Under Cluster Capacity, Provide a Node pool name and change number of Nodes to 2. There is "Set Node Pool to Autoscale" which will increase the number on Nodes, based on utilization. There is also a high avaialbility option for controlplane, which will run control plane components in HA mode. Because this is not production we will keep them unchecked.

  ![image](https://github.com/user-attachments/assets/51660cce-cd07-4fcb-a926-86cc25b1c61d)

- **Under Finalize section, provide a name for your cluster or leave it at default. Click Create Cluster.**
  
  ![image](https://github.com/user-attachments/assets/6a741582-2d9d-4248-80ed-90a9d16af748)

- **After some time (5-10min), Along with the Cluster, you will also see your two nodes also up and running.**

  ![image](https://github.com/user-attachments/assets/3644f373-281f-471d-85c8-12570b396e99)

- **Under the section "Connecting and managing this cluster". make a note of the below command, where the long string at the end is the cluster id. For security reason, i have provided a wrong id.** 

```shell
doctl kubernetes cluster kubeconfig save <cluster id from the Digital Ocean Control Panel>
```

output:
```shell
doctl kubernetes cluster kubeconfig save 12345678-1234-5678-9012-123456789012
```
  ![image](https://github.com/user-attachments/assets/ede40925-3dcb-467d-b44f-a3523396a793)

**Kubernetes Cluster is now ready to run your own container images.**

# Step 04 - Install & configure Digital Ocean CLI(DOCTL), API Token & Kubernetes CLI (Kubectl)

- **This step is required to connect the local linux instance to DOs kubernetes cluster. Run the below command.**

```shell
sudo snap install doctl
```
output:
```shell
vinayti@osboxes:~/simple-web-app-do/website-files$ sudo snap install doctl
doctl v1.124.0 from DigitalOcean✓ installed
vinayti@osboxes:~/simple-web-app-do/website-files$
```

- **Grant additional permission to doctl to integrate with kubectl.**

```shell
sudo snap connect doctl:kube-config
```
output:
```shell
vinayti@osboxes:~/simple-web-app-do/website-files$ sudo snap connect doctl:kube-config
vinayti@osboxes:~/simple-web-app-do/website-files$
```
- **create a API Token from the DigitalOcean Control Panel. On the left hand panel, select API**

  ![image](https://github.com/user-attachments/assets/9e38c30d-d1ae-4b4d-a56b-1bd3bf3c214c)

- **Click on generate new token.**

  ![image](https://github.com/user-attachments/assets/1d202f72-d1a0-4c7e-a2b1-0b7764d21373)

- **Give a meaningful name. Also select the expiry period. For simplicty. under scope, select full access. Click generate token**

  ![image](https://github.com/user-attachments/assets/67a1b4aa-29df-441b-9171-da8193eba729)

- **Copy the token and keep it in a notepad. We will need this to authenticate doctl.**

  ![image](https://github.com/user-attachments/assets/9ecbe2fc-538c-4ba9-9f8d-43bcc053a50e)

- **Use the API token to grant doctl access to your DigitalOcean account. Pass in the token string when prompted by doctl auth init, and give this authentication context a name.**

```shell
doctl auth init
```
output:
```shell
vinayti@osboxes:~/simple-web-app-do/website-files$ doctl auth init
Please authenticate doctl for use with your DigitalOcean account. You can generate a token in the control panel at https://cloud.digitalocean.com/account/api/tokens

❯ Enter your access token:  ●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

Validating token... ✔

vinayti@osboxes:~/simple-web-app-do/website-files$ doctl account get
User Email                    Team       Droplet Limit    Email Verified    User UUID                               Status
vinayvinodtiwari@gmail.com    My Team    10               true              c93d21dc-47f3-409a-8d19-64ea0b64eccb    active
vinayti@osboxes:~/simple-web-app-do/website-files$
```
- **To install kubectl , follow the document [here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management)**

output:
```shell
vinayti@osboxes:~/simple-web-app-do/website-files$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
vinayti@osboxes:~/simple-web-app-do/website-files$ sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
vinayti@osboxes:~/simple-web-app-do/website-files$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
vinayti@osboxes:~/simple-web-app-do/website-files$ sudo apt-get update
Hit:1 http://security.ubuntu.com/ubuntu oracular-security InRelease
Hit:2 https://download.docker.com/linux/ubuntu oracular InRelease
Hit:3 http://archive.ubuntu.com/ubuntu oracular InRelease
Hit:5 http://archive.ubuntu.com/ubuntu oracular-updates InRelease
Hit:6 http://archive.ubuntu.com/ubuntu oracular-backports InRelease
Get:4 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  InRelease [1,186 B]
Get:7 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  Packages [6,406 B]
Fetched 7,592 B in 5s (1,508 B/s)
Reading package lists... Done
vinayti@osboxes:~/simple-web-app-do/website-files$ sudo apt-get install -y kubectl
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following package was automatically installed and is no longer required:
  python3-netifaces
Use 'sudo apt autoremove' to remove it.
The following NEW packages will be installed:
  kubectl
0 upgraded, 1 newly installed, 0 to remove and 95 not upgraded.
Need to get 11.2 MB of archives.
After this operation, 57.4 MB of additional disk space will be used.
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.32/deb  kubectl 1.32.3-1.1 [11.2 MB]
Fetched 11.2 MB in 2s (6,647 kB/s)
Selecting previously unselected package kubectl.
(Reading database ... 156760 files and directories currently installed.)
Preparing to unpack .../kubectl_1.32.3-1.1_amd64.deb ...
Unpacking kubectl (1.32.3-1.1) ...
Setting up kubectl (1.32.3-1.1) ...
vinayti@osboxes:~/simple-web-app-do/website-files$
```
- **Set the kubectl context to point to DOs cluster. Please note that the clusterid i am using is wrong. Please use the one you see on the control manager. Also run the get nodes command as shown below, the output will confirm that you are now connected.**

```shell
doctl kubernetes cluster kubeconfig save <your cluster id>
```
output:
```shell  
vinayti@osboxes:~/simple-web-app-do/website-files$ doctl kubernetes cluster kubeconfig save 12345678-1234-5678-9012-123456789012
Notice: Adding cluster credentials to kubeconfig file found in "/home/vinayti/.kube/config"
Notice: Setting current-context to do-nyc1-vinayti-k8s-cluster01
vinayti@osboxes:~/simple-web-app-do/website-files$
vinayti@osboxes:~/simple-web-app-do/website-files$
vinayti@osboxes:~/simple-web-app-do/website-files$ kubectl get nodes
NAME                STATUS   ROLES    AGE   VERSION
node-pool01-6hce7   Ready    <none>   8h    v1.32.2
node-pool01-6hcem   Ready    <none>   8h    v1.32.2
vinayti@osboxes:~/simple-web-app-do/website-files$

```
# Step 05 - Kubernetes Fundamentals: Deploying Pods, ReplicaSets, Declarative definitions, Deployments, and LoadBalancer Service Type

- **Before proceeding further, lets understand some basic concept of Kubernetes.**

PODs: In Kubernetes, a Pod is the smallest and most basic execution unit that can be created and managed. It's a logical host for one or more containers. The container image that we created and uploaded to docker hub will run in the POD.

ReplicaSets: A Kubernetes ReplicaSet (RS) ensures a specified number of replicas (identical Pods) are running at any given time. It's a way to maintain a desired state for an application. This makes sure that the define number of pods are always running.

Declarative Definitions: Declarative definitions in Kubernetes often use YAML files to define resources.  These YAML files specify the desired state of resources like Pods, ReplicaSets, Deployments, and more.

Deployment: Kubernetes Deployments manage the rollout of new versions or configurations of an application. They provide a declarative way to describe the desired state of an application. Within a deployment, we can define the PODs, Replica Set, and many more.

Load Balancer Service Type: In Kubernetes, a LoadBalancer service type exposes a service to the outside world by provisioning a load balancer from a cloud provider. In our case, it is DigitalOcean.

- **Within the git cloned repository, there is another folder called k8s-files. cd into it**

output:
```shell
vinayti@osboxes:~/simple-web-app-do$ cd k8s-files/
vinayti@osboxes:~/simple-web-app-do/k8s-files$ ls
web-app-deployment.yml  web-app-hpa.yml  web-app-service.yml
vinayti@osboxes:~/simple-web-app-do/k8s-files$
```
These files are very well commented for anyone to understand the meaning of the lines. Lets use the kubectl create commands to deploy the resources.  In the below output, we can see that the PODs are deployed on seperate nodes. Kubernetes has a component called scheduler which takes care of the placement of the nodes, making sure they are placed on different nodes for resiliency. The count is 2, because in th file web-app-deployment.yaml, we have defined replica setting at 2.

```shell
kubectl create -f web-app-deployment.yaml
```
```shell
kubectl get pods -o wide
```
output:
```shell
vinayti@osboxes:~/k8s-files$ kubectl create -f web-app-deployment.yaml
deployment.apps/website-deployment created
vinayti@osboxes:~/k8s-files$ kubectl get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE   IP             NODE                NOMINATED NODE   READINESS GATES
website-deployment-847f9949b-c9477   1/1     Running   0          87s   10.108.0.184   node-pool01-6hcem   <none>           <none>
website-deployment-847f9949b-mk8q2   1/1     Running   0          87s   10.108.0.54    node-pool01-6hce7   <none>           <none>

vinayti@osboxes:~/k8s-files$
```
As the PODs are running, lets now make it public facing by deploying a load balancer service. In the below output, we are asking DigitalOcean to deploy a load balancer. This load balancer will serve the application running on PODs to the outside world. If you notice, the External IP section for the LoadBalancer Service type is pending. It will appear once the load balancer is deployed within the DigitalOcean Account. 

```shell
kubectl create -f web-app-service.yml
```
```shell
kubectl get svc
```
output:
```shell
vinayti@osboxes:~/k8s-files$ kubectl create -f web-app-service.yml
service/website-service created
vinayti@osboxes:~/k8s-files$ kubectl get svc
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP      10.109.0.1      <none>        443/TCP        25m
website-service   LoadBalancer   10.109.13.213   <pending>     80:30348/TCP   4s
vinayti@osboxes:~/k8s-files$ kubectl get svc
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP                                      PORT(S)        AGE
kubernetes        ClusterIP      10.109.0.1      <none>                                           443/TCP        28m
website-service   LoadBalancer   10.109.13.213   134.199.177.228,2604:a880:400:d1:0:1:7f21:2001   80:30348/TCP   2m34s
```

- **You can now match the IP address in the LoadBalance External IP section with the Load Balancer IP in Digital Oceans control panel.**

  ![Screenshot 2025-04-20 145214](https://github.com/user-attachments/assets/1515c585-e11e-4b24-90bc-23926c29f1f7)

- **Using the Load Balancer IP, you can now browse the Website we deployed on the POD.**

  ![image](https://github.com/user-attachments/assets/6d01a801-950e-4110-8b01-0b3349005710)

# Step 06 - Monitoring Your Cluster: Installing Metrics Server on DOKS

Kubernetes Metrics Server: It provides essential metrics for Kubernetes clusters, enabling better resource management and scaling decisions. As per the latest Kubernetes Version, it is not a part of the Kubernetes Package. Having said that, we will install it manually. It can also be deployed via definition file. The file is not included in this repo, to make sure, we always get the latest version. 

- **Run the below commnad to download the metric server.**

```shell
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
output:
```shell
vinayti@osboxes:~/k8s-files$ wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
--2025-04-20 05:33:08--  https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
Resolving github.com (github.com)... 20.207.73.82
Connecting to github.com (github.com)|20.207.73.82|:443... connected.
2025-04-20 05:33:10 (13.4 MB/s) - ‘components.yaml’ saved [4307/4307]
```
- **Run the below commnad to install the metric server.**

```shell
 kubectl create -f components.yaml
```
output:
```shell
vinayti@osboxes:~/k8s-files$ kubectl create -f components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```
- **In few minutes, we shoud see metrics server capturing the nodes as well as pods metrics.**

```shell
 kubectl top node; kubectl top pods
```
output:
```shell
vinayti@osboxes:~/k8s-files$ kubectl top nodes
NAME                CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
node-pool01-6hce7   33m          1%       808Mi           26%
node-pool01-6hcem   45m          2%       756Mi           24%
vinayti@osboxes:~/k8s-files$
vinayti@osboxes:~/k8s-files$
vinayti@osboxes:~/k8s-files$
vinayti@osboxes:~/k8s-files$ kubectl top pods
NAME                                 CPU(cores)   MEMORY(bytes)
website-deployment-847f9949b-c9477   0m           3Mi
website-deployment-847f9949b-mk8q2   0m           3Mi
```

# Step 07 - Scaling Your Application: Understanding and Deploying Horizontal Pod Autoscaler (HPA)

Horizontal POD Autoscaler: In earlier steps, we talked about replica which spins up the defined number of PODs, however, it is static. In case the load increases or decreases, we cannnot modify the replica count again and again. We need a mechanism, which can look at the current load and accordingly scale up or scale down the number of PODs. This is where HPA comes handy. Horizontal Pod Autoscaling (HPA) automatically scales the number of replicas of a pod based on observed CPU utilization or other custom metrics. Inour example, it gets the metrics from the metric server, we deployed in the last step.

- **There is another YAML file in the k8s-files folder web-app-hpa. Lets use it to deploy the (HPA).**

```shell
 kubectl create -f web-app-hpa.yaml
```

output:
```shell
vinayti@osboxes:~/k8s-files$ kubectl create -f web-app-hpa.yaml
horizontalpodautoscaler.autoscaling/website-hpa created
```

- **lets check the stats of the HPA.**

```shell
 kubectl get hpa
```
output:
```shell
vinayti@osboxes:~/k8s-files$ kubectl get hpa
NAME          REFERENCE                       TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
website-hpa   Deployment/website-deployment   cpu: 0%/50%   2         10        2          18s
```
The above output means, that the HPA will monitor CPU percentage of the PODs, and if reached 50%, it will scale the number of pods. The maximum scale size is 10. Once the load decreases. it can scale down to a minimum of 2. Also, the current CPU usage of the PODs is 0%

# Step 08 - Architechture Diagram Depicting the above configuration








