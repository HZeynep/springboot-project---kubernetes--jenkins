# springboot-project---kubernetes--jenkins
1. bir mvn projesi bulduk, github reposuna koyduk
2. mvn clean package komutuyla .jar file oluşturduk
3. Dockerfile ile image oluşturduk
4. Yaptığımız image'ı içeren deployment.yaml file oluşturduk
5. service ve ingress.yaml file oluşturduk


infrastructure: 
1. jenkins server oluşturduk: java,maven,docker, kubectl, eksctl
2. eks cluster kuruldu
3. 


= Hello World with Docker and Kubernetes

This repo consists of a Spring Boot Hello World application. It shows:

. Run & test the application from CLI
. Create a Docker image, run the container and test it
. Create Kubernetes deployment and test it

Pre-requisities

- Jenkins server launched
- Docker, Kubectl, kubeadm, maven, java installed.



Let's get started!

Step 1: Creating simple dockerfile
In order to host and orchestrate our application on Kubernetes we have to build a docker image to build and run the application from the repo.
Create a dockerfile with the following contents and place it in the root location of the project repository.

Step 2: Creating Kubernetes yaml files
Create Kubernetes yaml files (deployment.yml,service.yml, ingress.yaml) with the following contents and place it in the root location of the project repository.

Step 3: For Automated Jenkins Configuration
Creating Jenkins job
Open the jenkins url and click on “New Item”
Provide a name for the job and select freestyle job
Provide the git repository url and credentials for cloning the project. Also please specify the branch.

Step-5: Create Token

- Add token to the github. So, go to your github Account Profile  on right of the top >>>> Settings>>>>Developer Settings>>>>>>>>Personal access tokens >>>>>> Generate new token

- Go to the >>>>>> project and open Git config file.

```bash
cd .git
vi config

Add "token" after "//" in the "url" part . And also paste "@" at the and of the token.
  "url = https://<yourtoken@>github.com/hello-world.git

```

Step-6: Create Webhook 

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
docker image build -t dolphinstar/spring-boot:latest .
```

=== Push Docker Image

```
docker image push dolphinstar/spring-boot:latest
```

=== Run Docker Container

```
docker container run -d --name hello-world -p 8080:8080 dolphinstar/spring-boot:latest
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

=== Create Deployment

```
kubectl apply -f deployment.yaml
```

=== Test Application

```
curl http://`kubectl get svc hello-world-service -o jsonpath={.status.loadBalancer.ingress[0].hostname}`
```

=== Delete Deployment

```
kubectl delete -f deployment.yaml
```
