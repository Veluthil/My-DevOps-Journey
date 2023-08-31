# Project 6 - CI Project with Jenkins and Docker [Docker PAAC]

## Prerequisities:

 * AWS Account
 * GitHub account
 * EC2 with Jenkins from previous project 
 * EC2 with SonarQube from previous project
 * Docker Engine in Jenkins
 * AWS CLI
 * IAM User
 * ECR repo
 * Plugins in Jenkins: ECR, Docker pipeline, AWS SDK for credentials

 ### 1. Install Docker Engine on the Jenkins Server

 - SSH to your Jenkins instance and use the following commands:
 ```sh
 sudo -i
 sudo apt update && sudo apt install awscli -y
 # Update the apt package index and install packages to allow apt to use a repository over HTTPS
 sudo apt-get update
 sudo apt-get install ca-certificates curl gnupg
 # Add Docker's official GPG key
 sudo install -m 0755 -d /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 sudo chmod a+r /etc/apt/keyrings/docker.gpg
 # Use the following command to set up the repository
  echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 #Update the apt package index
 sudo apt-get update 
 # To install the latest version, run:
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
 # Verify that the Docker Engine installation is successful by running the hello-world image.
 sudo docker run hello-world
 # You can also check its status by running:
 systemctl status docker
```
- All of the Jenkins jobs run with a Jenkins user, therefore you need to add Jenkins user to the docker group to allow it run docker commands.
- Make sure you are now a root user and run:
```sh
 usermod -a -G docker jenkins
 reboot
```

### 2. Create IAM user and ECR repository.

- Go to IAM service in your AWS -> `Create user`:
```
User name: jenkins
Attach policies directly:
- AmazonEC2ContainerRegistryFullAccess
- AmazonECS_FullAccess
```
- Create user with above settings and next create an access key for this user. Go to `jenkins` user -> `Security Credentials` -> `Access keys` -> `Create access key`:
```
Use case: CLI
Check: I understand the above...
Create access key
```
- Download .csv file.
- Create ECR repository. Go to `Elastic Container Registry` service -> `Repositories` -> `Create Repository`:
```
Visibility settings: Private
Repository name: vprofileappimg [or whatever you like]
Create repository
```
- Copy the URI and store it for later.