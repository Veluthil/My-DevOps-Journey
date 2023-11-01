# Project 7 - Continuous Integration and Continuous Delivery with Jenkins and GitHub (Build Triggers)

In Jenkins, build triggers are essential mechanisms that kickstart the execution of a Jenkins job, commonly referred to as a build or project. These triggers are designed to automatically commence the job in response to specific events or conditions. Within this project, we'll delve into some of these key build triggers.

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
- Congratulations, you've just used a Jenkinsfile from your GitHub in a Jenkins build.

### 4. Learn and use Popular Triggers.

1. Git Webhook
- Copy the URL with a port of Jenkins instance and go to your GitHub repository's settings -> `Webhooks` -> `Add webhook`:
```
Payload URL: <paste Jenkins' URL with port here>/github-webhook/
Content type: application/json
Add webhook
```
- Refresh the page, you should see the green check mark next to the just created webhook. Click on it and check `Recent Deliveries` - this means the json payload was delivered.
- Go to Jenkins' job and click on `Configure` -> `Build Triggers` -> select `GitHub hook trigger for GITScm polling` -> `Save`
- Now let's do a new commit in your repository and see a new build being automatically created. Go to Git Bash/Terminal, make sure you are in your repository's directory and create a new file:
```sh
touch testfile.txt
git add .
git commit -m "testtrigger1"
git push origin main
```
- You should see a new build being created in your Jenkins.

2. Poll SCM
- Go to Jenkins' job -> `Configure` -> uncheck `GitHub hook trigger for GITScm polling` -> select `Poll SCM` -> now, you have to give a schedule in a cron job format in order to make Jenkins check for commits on GitHub. This will be an opposite mechanism from the previous build trigger:
```
* * * * *
```
- Above cron job will be triggered every minute, hour, day, month and day of the week.
- `Git Polling Log` should have appeared in the left menu.
- Again, let's do a new commit in your repository and see a new build being automatically created. Go to Git Bash/Terminal, make sure you are in your repository's directory and create a new file:
```sh
touch testfile2.txt
git add .
git commit -m "testtrigger2"
git push origin main
```
- You should see a new build being created in your Jenkins.

3. Scheduled jobs
- Go to Jenkins' job -> `Configure` -> uncheck `Poll SCM` -> select `Build periodically` -> `Schedule`:
```
* * * * *
```
- Unlike the Poll SCM option, this job will trigger based solely on the schedule you select, without regard to whether new commits appear on your GitHub account. Feel free to choose any schedule that suits you.

4. Remote triggers
- First, Generate JOB URL: `Job Configure` -> `Build Triggers` -> check mark on `Trigger builds remotely` -> give a token name -> generate URL & save in a file
- Generate Token for User: click on your username drop down button (Top right corner of the page) -> `Configure` -> `API Token` -> `Generate` -> copy the token name and save username:tokenname in a file
- Generate CRUMB: wget command is required for this, so download wget binary for git bash. Extract content in your path to ../Git/mingw64/bin. Run below command in Git Bash/Terminal, (replace username, password, Jenkins URL):
```sh
wget -q --auth-no-challenge --user username --password password --output-document -
'http://JENNKINS_IP:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)'
```
Save the token in a file
- Build Job from URL: by now we should have below details
    JENKINS Job URL with token
        E:g http://52.15.216.180:8080/job/vprofile-Code-Analysis/build?token=testtoken
    API Token
        USERNAME:API_TOKEN
        E:g admin:116ce8f1ae914b477d0c74a68ffcc9777c
    Crumb
        E:g Jenkins-Crumb:8cb80f4f56d6d35c2121a1cf35b7b501
Fill all the above details in below URL and execute:
```sh
curl -I -X POST http://username:APItoken @Jenkins_IP:8080/job/JOB_NAME/build?token=TOKENNAME
-H "Jenkins-Crumb:CRUMB"
```
e:g curl -I -X POST http://admin:110305ffb46e298491ae082236301bde8e@52.15.216.180:8080/job/
vprofile-Code-Analysis/build?token=testtoken -H "Jenkins-Crumb:8cb80f4f56d6d35c2121a1cf35b7b501"
- You should see a job being build in your Jenkins.