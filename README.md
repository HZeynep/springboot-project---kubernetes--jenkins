# springboot-project---kubernetes--jenkins

-Short explanation about what/why you did
- 1-Create springboot project and upload it in GitHub.
- 2-Create .jar file with ```mvn package``` command.
- 3-Create a Dockerfile to build docker image.
- 4-Create deployment.yml,service.yml and ingress.yml files for kubernetes objects.
- 5-Create a Jenkinsfile to build image, push to image to Dcokerhub and deploy app on Kubernetes (AWS EKS).
- Infrastructure:
- 1- Jenkins server with java,maven, Docker, Kubectl and eksctl.
- 2-AWS EKS cluster.


= Hello World with Docker and Kubernetes

This repo consists of a Spring Boot Hello World application. It shows:

. Run & test the application from CLI
. Create a Docker image, run the container and test it
. Create Kubernetes deployment and test it

Pre-requisities

- Jenkins server launched
- Docker, Kubectl, eksctl, maven, java installed.



Let's get started!

Step 1: Creating simple dockerfile
In order to host and orchestrate our application on Kubernetes we have to build a docker image to build and run the application from the repo.
Create a dockerfile with the following contents and place it in the root location of the project repository.
```
FROM openjdk:11-jre
ADD ./target/*.jar /app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```
Step 2: Creating Kubernetes yaml files
Create Kubernetes yaml files (deployment.yml,service.yml, ingress.yaml) with the following contents and place it in the root location of the project repository.
deployment.yml
```
apiVersion: apps/v1 
kind: Deployment 
metadata:
  name: hello-world-deploy
spec:
  replicas: 1 
  selector:  
    matchLabels:
      app: hello-world
  minReadySeconds: 10 
  strategy:
    type: RollingUpdate 
    rollingUpdate:
      maxUnavailable: 1 
      maxSurge: 1 
  template: 
    metadata:
      labels:
        app: hello-world
        env: front-end
    spec:
      containers:
      - name: hello-world-pod
        image: hzeynep/springboot
        ports:
        - containerPort: 8080
 ```
 service.yml
 ```
 apiVersion: v1
kind: Service   
metadata:
  name: hello-world-service
  labels:
    app: hello-world
spec:
  type: NodePort
  ports:
  - port: 8080  
    targetPort: 8080
  selector:
    env: front-end 
 ```   
  ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello-world-service
                port: 
                  number: 8080
 ```

Step 3: For Automated Jenkins Configuration
Create a Jenkinsfile with following content
```
pipeline {
    agent any
    environment {
        PATH=sh(script:"echo $PATH:/var/lib/jenkins/apache-maven-3.8.5/bin", returnStdout:true).trim()
    }
    stages {
        stage('Package Application') {
            steps {
                echo 'Packaging the app into jars with maven'
                sh "mvn clean package"
            }
        }
        stage('Build Docker Images') {
            steps {
                echo 'Building App Staging Images'
                sh "docker build -t hzeynep/springboot ."
                sh 'docker image ls'
            }
        }
        stage('Push Images to Docker Repo') {
            steps {
                echo "Pushing Image to ECR Repo"
                sh "docker push hzeynep/springboot"
            }
        }
        stage('Deploy App on Kubernetes Cluster'){
            steps {
                echo 'Deploying App on K8s Cluster'
                sh "kubectl apply -f k8s"
            }
        }
    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'
        }
    }
}
```

Step-4: Create Jenkins Job
- Go to the Jenkins dashboard and click on ```New Item``` to create a pipeline.
- Enter ```hello-world-pipeline``` then select ```Pipeline``` and click ```OK```.
- Enter ```Hello-world pipeline configured with Jenkinsfile and GitHub Webhook``` in the description field.
- Put a checkmark on ```GitHub Project``` under ```General``` section, enter URL of the project repository.
- Put a checkmark on ```GitHub hook trigger for GITScm polling``` under ```Build Triggers``` section.
- Go to the ```Pipeline``` section, and select ```Pipeline script from SCM``` in the ```Definition``` field.
- Select ```Git``` in the ```SCM``` field.
- Enter URL of the project repository, and let others be default.
```https://github.com/xxxxxxxxxxx/hello-world.git```
- Click ```apply``` and ```save```. Note that the script ```Jenkinsfile``` should be placed under root folder of repo.


Step-5: Create Webhook 

- Go to the `hello-world` repository page and click on `Settings`.

- Click on the `Webhooks` on the left hand menu, and then click on `Add webhook`.

- Copy the Jenkins URL from the AWS Management Console, paste it into `Payload URL` field, add `/github-webhook/` at the end of URL, and click on `Add webhook`.

 Creating Jenkins Pipeline for the Project with GitHub Webhook

Step-1: Github process

- Go to the Jenkins dashboard and click on `New Item` to create a pipeline.

- Enter `hello-world-pipeline` then select `Pipeline` and click `OK`.

- Enter `Hello-world pipeline configured with Jenkinsfile and GitHub Webhook` in the description field.

- Put a checkmark on `GitHub Project` under `General` section, enter URL of the project repository.

- Put a checkmark on `GitHub hook trigger for GITScm polling` under `Build Triggers` section.

- Go to the `Pipeline` section, and select `Pipeline script from SCM` in the `Definition` field.

- Select `Git` in the `SCM` field.

- Enter URL of the project repository, and let others be default.

```text
https://github.com/xxxxxxxxxxx/hello-world.git
```

- Click `apply` and `save`. Note that the script `Jenkinsfile` should be placed under root folder of repo.


== CLI

=== Run App

```
mvn spring-boot:run
```

=== Test Application

```
curl http://localhost:8080
```

== Docker

=== Build Application

```
mvn clean package
```

=== Build Docker Image

```
docker image build -t hzeynep/spring-boot:latest .
```

=== Push Docker Image

```
docker image push hzeynep/spring-boot:latest
```

=== Run Docker Container

```
docker container run -d --name hello-world -p 8080:8080 hzeynep/spring-boot:latest
```

=== Test Application

```
curl http://localhost:8080
```

=== Delete Docker Container

```
docker container rm -f hello-world
```

== Kubernetes

==Create AWS EKS cluster
```
eksctl create cluster --region us-east-1 --zones us-east-1a,us-east-1b,us-east-1c --node-type t2.medium --nodes 1 --nodes-min 1 --nodes-max 1 --name my-cluster
```
==Create ingress controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
```
=== Create Deployment

```
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f ingress.yaml
```

=== Test Application

```
curl http://`kubectl get svc hello-world-service -o jsonpath={.status.loadBalancer.ingress[0].hostname}`
```

=== Delete Deployment

```
kubectl delete -f deployment.yml
kubectl delete -f service.yml
kubectl delete -f ingress.yaml

```
==Delete EKS 
```
./eksctl delete cluster my-cluster
```
