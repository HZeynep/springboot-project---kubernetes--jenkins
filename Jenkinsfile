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
