# Project 7 - Continuous Integration and Continuous Delivery with Jenkins and GitHub (Build Triggers)

## Pre-requisities:

* AWS Account
* GitHub account
* Jenkins
* Nexus
* SonarQube
* Slack 

### 1. Create Keypair.

- You will need a key-pair while launching the instances. If you do not have one, create it - open Git Bash on Windows or Terminal on MacOS and type:
```sh
ssh-keygen.exe
``` 
- The ssh keys will be stored in your GitHub account, you should be able to find them in your home directory. 
```sh
ls ~/.ssh/
cat ~/.ssh/id_rsa.pub
```
- Copy the content of the public key and go to your GitHub account -> `Settings` -> `SSH and GPG keys` -> `New SSH key` -> Give a title and paste the content of your generated public key -> `Add SSH key`.

### 2. Create a new GitHub repository.

- You can give it any name, for example "Jenkins-Triggers", make it public or private (I have made this one: https://github.com/Veluthil/Jenkins-Triggers.git)
- Click on `SSH` and copy the link that appears. Go back to Git Bash/Terminal, create a new directory, where you will clone this repository and cd to it. 
```sh
git clone HERE INSERT YOUR COPIED LINK 
```
- cd into this cloned repository and add a simple Jenkinsfile, for example:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps{
                sh 'echo "Build completed."'
            }
        }
    }
}
```
- Now run:
```sh
git add .
git commit -m "Initial commit"
git push origin main
```
- You can check whether Jenkinsfile appeared in your GitHub repository.

### 3. Create new Jenkins job.

- Go to Jenkins (you have to start your Jenkins instance from the previous projects and go to 'http://JENKINS-PUBLIC-IP:8080'). In order to prevent a possible error "Host key verification failed.", got to `Manage Jenkins` -> `Configure Global Security` -> `Git Host Key Verification Configuration` -> select `Accept first connection` -> `Apply`/`Save`.

- Create new job and choose `Pipeline` -> in `Pipeline Definition` choose `Pipeline script from SCM`:
```
SCM: Git
Repository URL: get the SSH URL for your previous step repo
Credentials: Add Jenkins
    Kind: SSH Username with private key
    ID: gitsshkey [or aything you like]
    Description: gitsshkey [or aything you like]
    Username: your GitHub account
    Private Key: Enter directly -> Add -> Copy your PRIVATE key (in bash/terminal use 'cat ~/.ssh/id_rsa`)
    Add
    Choose your credentials
Branch: change to 'main' from 'master'
Script Path: Jenkinsfile
Save
```
- Build Now.