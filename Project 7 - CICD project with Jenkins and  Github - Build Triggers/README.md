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
- Copy the content of the public key.

### 2. Create a new GitHub repository.

- You can give it any name, for example "Jenkins-Triggers", make it public or private.