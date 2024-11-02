pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'isasubhi/my-nginx-container'
        KUBE_DEPLOYMENT_NAME = 'nginx-deployment'
        NAMESPACE = 'my-ns'
        GIT_REPO = 'https://github.com/IsaSubhi/DevOps_Project.git'
        VERSION = '1.2'
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
                sh "docker build -t ${DOCKERHUB_REPO}:${VERSION} ."
            }
        }
        
        stage('3-Push Image to Dockerhub') {
            steps {
                sh "docker login"
                sh "docker push ${DOCKERHUB_REPO}:${VERSION}"
            }
        }

        stage('4-Deploy to Kubernetes') {
            steps {
                script {
                    def NamespaceExists = sh(script: "kubectl get namespace ${NAMESPACE} || echo 'not found'", returnStdout: true).trim()
                    if (NamespaceExists.contains("not found")) {
                        sh "kubectl create namespace ${NAMESPACE}"
                    } else {
                echo "Namespace ${NAMESPACE} already exists."
                    }
                    def DeploymentExists = sh(script: "kubectl get deployment ${KUBE_DEPLOYMENT_NAME} --namespace=${NAMESPACE} || echo 'not found'", returnStdout: true).trim()
                    if (DeploymentExists.contains("not found")) {
                        sh "kubectl create deployment ${KUBE_DEPLOYMENT_NAME} --namespace=${NAMESPACE} --image=${DOCKERHUB_REPO}:${VERSION}"
                    } else {
                        echo "Deployment ${KUBE_DEPLOYMENT_NAME} already exists in namespace ${NAMESPACE}."
                    }
                    def SvcExists = sh(script: "kubectl get service ${KUBE_DEPLOYMENT_NAME}-service --namespace=${NAMESPACE} || echo 'not found'", returnStdout: true).trim()
                    if (SvcExists.contains("not found")) {
                        sh "kubectl expose deployment ${KUBE_DEPLOYMENT_NAME} --name=${KUBE_DEPLOYMENT_NAME}-service --namespace=${NAMESPACE} --port=30006 --target-port=80"
                    } else {
                        echo "Service ${KUBE_DEPLOYMENT_NAME}-service already exists in namespace ${NAMESPACE}."
                    }
                }
            }
        }
        
        stage('Check All Pods Ready') {
            steps {
                script {
                    boolean allPodsReady = false
                    for (int i = 1; i <= 10; i++) {
                        echo "Checking readiness of all pods... Attempt: ${i}/10"
                        def podStatuses = sh(
                            script: """
                                kubectl get pods -n ${NAMESPACE} -o jsonpath='{range .items[*]}{.metadata.name}:{.status.conditions[?(@.type=="Ready")].status} {end}'
                            """,
                            returnStdout: true
                        ).trim()
                        def allReady = podStatuses.split(' ').every { status ->
                            status.split(':')[1] == 'True'
                        }
                        if (allReady) {
                            echo "All pods are ready!"
                            allPodsReady = true
                            break
                        } else {
                            echo "Not all pods are ready yet. Retrying in 3 seconds..."
                            sleep 3
                        }
                    }
                    if (!allPodsReady) {
                        error "Not all pods became ready within 10 retries."
                    }
                }
            }
        }
    }
}
