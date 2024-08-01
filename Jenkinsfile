pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'isasubhi/my-nginx-container'
        KUBE_DEPLOYMENT_NAME = 'nginx-deployment'
        GIT_REPO = 'git@github.com:IsaSubhi/DevOps_Project.git'
    }

    stages {
        stage('1-Clone Git Repository') {
            steps {
                sh "git init"
                sh "git pull ${GIT_REPO}"
            }
        }
        
        stage('2-Build Docker Image') {
            steps {
                sh """
                echo 'FROM nginx:latest' > Dockerfile
                echo 'COPY index.html /usr/share/nginx/html/index.html' >> Dockerfile
                """
                sh "docker build -t ${DOCKERHUB_REPO}:1.0 ."
            }
        }
        
        stage('3-Push Image to Dockerhub') {
            steps {
                sh "docker login"
                sh "docker push ${DOCKERHUB_REPO}:1.0"
            }
        }

        stage('3-Deploy to Kubernetes') {
            steps {
                sh "kubectl create deployment ${KUBE_DEPLOYMENT_NAME} --image=${DOCKERHUB_REPO}:1.0"
                sh "kubectl expose deployment ${KUBE_DEPLOYMENT_NAME} --port=80 --target-port=80 --type=NodePort --name=${KUBE_DEPLOYMENT_NAME}-service"
            }
        }
    }
}
