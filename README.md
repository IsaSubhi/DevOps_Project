# Welcome to my DevOps Project
### Overview
As the employee pushes his code to GitHub, it triggers a Webhook in Jenkins that pulls the code, creates a Docker image the code and uploads it to DockerHub.
Jenkins then creates a deployment in Kubernetes with the Docker image for the employee to check the code.
![image](https://github.com/user-attachments/assets/f9086d18-1561-408b-baaa-fbb44e60018f)

## Requirements:
- GitHub
- Jenkins
- Jenkins Git Plugin
- Docker
- DockerHub
- Kubernetes
- ## Configuration:
[Jenkinsfile](Jenkinsfile)'s environments can be edited to use the code 
<br>
[index.html](index.html) is the code that is used to create the deployment
