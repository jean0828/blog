---
title: Adding the Docker Build and Publish Stage in a Pipeline
date: 2023-04-04 21:20:00 -0500
categories: [devsecops]
tags: [kubernetes,google cloud,devsecops,jenkins,docker]     # TAG names should always be lowercase
---

This is part from the course of Linux Foundation called Implementing DevSecOps and it’s given by [initcron](https://gist.github.com/initcron).

In a development environment, you can build an image with Docker. However, in a Continuous Integration environment such as Jenkins, it's not prudent to use Docker daemon, as it would need privileged access. You can however use tools such as **Kaniko** to build an image in a secure, non-privileged environment. Follow the steps in this section to achieve that.

The step-by-step process to set up an automated container image build and publish process involves the following:

1. Setting up the credentials to connect to the container registry. Kaniko will read these credentials while being launched as part of a pipeline run by Jenkins.
2. Adding a build agent configuration so that Jenkins knows which container image to use and how to launch a container/pod to run the job with Kaniko.
3. Adding a stage to the Jenkins pipeline to launch Kaniko to build an image using Dockerfile in the source repository and publish it to the registry.


## Adding the Registry Credentials

Kaniko is able to read the credentials store as Kubernetes secrets. Using Docker Hub as the container registry, we can create a secret with the registry credentials:

```
kubectl create secret -n ci docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=user --docker-password=pass --docker-email=user@email.org
```

![secretdocker](https://i.imgur.com/dZ9Ksvl.png)

We can check the secret was added with the command

```
kubectl get secrets -n ci
```

## Create the Jenkins Agent Configuration

Add the build-agent configuration that Jenkins will use to create a container and run the image build job with it. This configuration uses Kaniko, which is an image build tool sitting inside Kubernetes and creates an image without requiring a Docker daemon. This is a secure alternative to using DIND, which is a privileged container, or mounting a Docker socket directly on the Jenkins host.

we should edit the **build-agent.yml** file which is part of the project to add the Kaniko agent configuration, as in:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: spring-build-ci
spec:
  containers:
    - name: maven
      image: maven:alpine
      command:
        - cat
      tty: true
      volumeMounts:
        - name: m2
          mountPath: /root/.m2/
    - name: docker-tools
      image: rmkanda/docker-tools:latest
      command:
        - cat
      tty: true
      volumeMounts:
        - mountPath: /var/run
          name: docker-sock
        - mountPath: /tmp/trivycache/
          name: trivycache
    - name: kaniko
	  image: gcr.io/kaniko-project/executor:v1.6.0-debug
	  imagePullPolicy: Always
	  command: 
	   - sleep
	   args:
	    - 99d
	   volumeMounts:
	    - name: jenkins-docker-cfg
	      mountPath: /kaniko/.docker
    - name: trufflehog
      image: rmkanda/trufflehog
      command:
        - cat
      tty: true
    - name: licensefinder
      image: licensefinder/license_finder
      command:
        - cat
      tty: true
  volumes:
    - name: m2
      hostPath:
        path: /tmp/.m2/
    - name: docker-sock
      hostPath:
        path: /var/run
    - name: trivycache
      hostPath:
        path: /tmp/trivycache/
	- name: jenkins-docker-cfg
	  projected:
	   sources:
	   - secret: 
	       name: regcred
		   items:
		     - key: .dockerconfigjson
		       path: config.json
```

Also, we should add the stage to Jenkinsfile to build an image with Kaniko. When launched, this stage will use the registry credentials set up earlier, read the Dockerfile which is available as part of the source code repo, build an image and publish it to the registry.

```
stage('Docker BnP') {
	steps {
		container('kaniko') {
		 sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/xxxxxx/dsodemo'
		 } 
	} 
}
```

In my case, the Jenkinsfile is:

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

        stage('Docker BnP') {

          steps {

            container('kaniko') {

              sh '/kaniko/executor -- force -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/jean0828/dsodemo'

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

The project is located [here](https://github.com/jean0828/dso-demo-lab2). The pipeline will be execute when there is a new commit or when you add the pipeline as described in the [first part](https://jean0828.github.io/blog/posts/Deploying-a-Cloud-Environment-for-DevSecops/). Then, when it’s completed, we obtain the following image:

![pipeline-completed](https://i.imgur.com/ym9tfES.png)

As we can see, a new step was added to the Package stage and it’s the step to build and publish the image in Docker Hub. Besides, we can confirm that was executed without problems because we can see the image in Docker Hub:

![dockerhub](https://i.imgur.com/9O8HeuD.png)

Now, with simple steps we have a continuos integration pipeline.