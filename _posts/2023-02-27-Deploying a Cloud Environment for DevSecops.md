---
title: Deploying a Cloud Environment for DevSecops
date: 2023-02-27 22:20:00 -0500
categories: [devsecops]
tags: [kubernetes,google cloud,devsecops,jenkins]     # TAG names should always be lowercase
---

For this case, we are going to use Google Cloud. There, a kubernetes cluster will be created, Linux environment for development will be also created, and Jenkins will be deployed within Kubernetes.

This is part from the course of Linux Foundation called Implementing DevSecOps and it’s given by [initcron](https://gist.github.com/initcron).

## Create a Kubernetes Cluster with GKE

First, it's recommended to create a new project where all the environment will be deployed. To do that, we just have to click in *New Project*.

![new project](https://i.imgur.com/Q7GabDC.png)

For this case, the name is **DevSecOps**.

![project](https://i.imgur.com/D5roQI2.png)

After creating the project, we should enable the Kubernetes Engine API. To enable it, we have to go to navegation menu →  kubernetes Engine → Clusters &rarr; Enable

![kubernetes-api](https://i.imgur.com/k3JCe4J.png)

![enable-kuber](https://i.imgur.com/KdHQFdi.png)

we have to wait some minutes to have Kubernetes Engine Api activated.

When the Kubernetes, we can create the cluster with the following options.

![create-cluster](https://i.imgur.com/kcnI4a4.png)


* Cluster mode: Standard
* Release Channel: Rappid Channel
* Version: select the newest

As a result, we'll have a kubernetes cluster with 3 nodes. 

![cluster](https://i.imgur.com/lrL0U1H.png)


Next, it's recommended to create a firewall rule to allow connections from your public IP. to crteate the rule, we can do it from Navigation Menu → VPC Network  → Firewall.

![firewall](https://i.imgur.com/FRZGNq9.png)

Then, we have to select the one which has the following naming convention: **gke-cluster-X-XXXXX-all** and add your IP.

![fw-rule](https://i.imgur.com/X38KHzn.png)


![rule](https://i.imgur.com/scmR78N.png)

Now it’s time to connecto to our cluster and the easiest way to do it, it’s using the cloud shell and execute the command suggested.

![cloud-shell](https://i.imgur.com/GaJOEIn.png)

in this environment, this is the command:

```
gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project devsecops-378701
```

After that, we can check the status of the cluster:

```
kubectl get nodes
kubectl config get-contexts
kubectl get pods --all-namespaces
```

![output](https://i.imgur.com/qS0OJD2.png)

Finally, check that we also have helm installed with the command *helm version*.

## Creating a Linux Development Environment

The Linux VM is where our development environmnet will be. Here, we can test our apps with dockers before to put it in the deployment environmnet.

To create the Linux VM, we have to go to VM Instances and click in *create instance* wih those options:

* Size Disk: 30 GB
* Operating System: Ubuntu LTS
* Create or Import SSH Keys
* network tag: dev
* Startup Script: include this script:

```
#!/bin/bash

apt-get update
apt-get install -y git wget

# Install Docker
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

apt-get update

apt-get install -yq docker-ce

cd /root
git clone https://github.com/codespaces-io/codespaces.git
curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

NOTE:  Credits of the script to [initcron](https://gist.github.com/initcron)

Later, we have to create a FW rule to allow ssh connections from our IP.

![disk](https://i.imgur.com/ElvBNgk.png)

![tag](https://i.imgur.com/pPrbC6J.png)

![ssh](https://i.imgur.com/CwQLrb9.png)

![script](https://i.imgur.com/TYv0Yhf.png)

![ruledev](https://i.imgur.com/rkQHiNY.png)

Finally, to connect to the vm, we just use ssh and check docker is installed:

```
ssh user@ip
```

![ssh](https://i.imgur.com/aroP7P1.png)

![inst](https://i.imgur.com/YE7B9Bl.png)

With that, we can confirm that our dev environmnet was successfully deployed.

## Installing Jenkins

The objective of installing Jenkins is to use continous integration. In this case, we’re going to use Helm. Remember that Helm is the package manager of kubernetes.

To install Jenkins, we should follow those steps:

1. add Jenkins repository
```
helm repo add jenkins [https://charts.jenkins.io](https://charts.jenkins.io)
helm repo update
```

2. Create a namespace for CI
```
kubectl create namespace ci
```

3. Create *jenkins.values.yaml* file:
```
controller:
 serviceType: NodePort
 resources:
  requests:
   cpu: "400m"
   memory: "512Mi"
  limits:
    cpu: "2000m"
```

4. Install Jenkins
```
helm install -n ci --values jenkins.values.yaml jenkins jenkins/jenkins
```

![jenkins](https://i.imgur.com/RRMIMRC.png)

When the installation is donde, we have to follow the steps of the output to get the admin password and then login.

```
kubectl exec --namespace ci -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
[output]t3P4I48zb90j7Dbr3aBC8J
export NODE_PORT=$(kubectl get --namespace ci -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
export NODE_IP=$(kubectl get nodes --namespace ci -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT
[output]http://10.128.0.6:31893
```

according to the previous output, we have to access this webpage http://10.128.0.6:319893. However, it’s an internal IP and we have to find the external IP adress. To find the external IP, we have to execute this command:

```
kubectl get nodes -o wide
```

![access](https://i.imgur.com/aq9Ivxt.png)

In this case, the url is: http://35.223.160.114:31893/

![webaccess](https://i.imgur.com/S3Y4oAg.png)

Now, it’s time to configure Jenkins:

1. change admin paswword.
2.  Go to Manage Jenkins → Plugin Manager → available plugins and install:
	1. Blue Ocean
	2.  Configuration as Code
3.  Download now and install after restart

![plugins](https://i.imgur.com/qb9hJjB.png)

![installpl](https://i.imgur.com/Vx19Qu4.png)

## Setting a simple continnuos Integration DevOps pipeline

Fort his case, we’re going to fork this java repo: https://github.com/lfs262/dso-demo. In this repo, we can see the jenkins file and see how the pipeline will be:

```
pipeline {

agent {

kubernetes {

yamlFile 'build-agent.yaml'

defaultContainer 'maven'

idleMinutes 1

}

}

stages {

stage('Build') {

parallel {

stage('Compile') {

steps {

container('maven') {

sh 'mvn compile'

}

}

}

}

}

stage('Test') {

parallel {

stage('Unit Tests') {

steps {

container('maven') {

sh 'mvn test'

}

}

}

}

}

stage('Package') {

parallel {

stage('Create Jarfile') {

steps {

container('maven') {

sh 'mvn package -DskipTests'

}

}

}

}

}

stage('Deploy to Dev') {

steps {

// TODO

sh "echo done"

}

}

}

}
```

After forking the repo, in Jenkins we have to go to Blue Ocean. Blue Ocean is the UI created to run pipelines.

![blueo](https://i.imgur.com/SB1hL54.png)

From there, create a new pipeline, link with github account and follow the steps for Jenkins can pull the code, the jenkins file, create the pipeline, and push status updates to the Git repository on the commit history.

![oceanb2](https://i.imgur.com/e7rPcSb.png)

![github](https://i.imgur.com/P6czXcu.png)

Then, Copy the token and paste in Jenkins

![token](https://i.imgur.com/JRj5BGe.png)

![tokenpaste](https://i.imgur.com/5hP8zZm.png)

After, create pipeline. This action will read the repository, find the jenkins file, create the pipeline, and launch it.

![pipeline](https://i.imgur.com/xY7DeO2.png)

Also, you can confirm that our jenkins jobs will be launched as a Kubernetes pods.

![pods](https://i.imgur.com/mVp3bGr.png)

if you have some error during the CI, it’s because you need to update blue ocean and github plugins.

![success](https://i.imgur.com/zTpz1P5.png)

This is a simple Kubernetes native countinuous integration pipeline.






