# Project 5 - Continuous Integration with Jenkins, Nexus, SonarQube and Slack


## Prerequisities:

 * AWS Account
 * GitHub account
 * Jenkins
 * Nexus
 * SonarQube
 * Slack 

### 1. Create Security Groups for Jenkins, Nexus and SonarQube

* Jenkins SecGrp
```sh
Name: JenkinsSG
Allow: SSH from MyIP
Allow: 8080 from Anywhere IPv4 and IPv6
```

* Nexus SecGrp
```sh
Name: NexusSG
Allow: SSH from MyIP
Allow: 8081 from MyIP and JenkinsSG
```

* SonarQube SecGrp
```sh
Name: SonarSG
Allow: SSH from MyIP
Allow: 80 from MyIP and JenkinsSG
```

After creating `SonarSG` add another Inbound rule to JenkinsSG - Allow access on 8080 from SonarSG. SonarQube will send the reports back to Jenkins.

### 2. Create EC2 instances for Jenkins, Nexus and SonarQube

#### Jenkins Server Setup

- Create `JenkinsServer` with the following settings and the userdata script:
```sh
Name: JenkinsServer
AMI: Ubuntu 20.04
SecGrp: JenkinsSG
InstanceType: t2.small
KeyPair: CREATE NEW KEY PAIR, FOR EXAMPLE JenkinsKey.pem
Additional Details: copy from userdata/jenkins_setup.sh or live blank and after launching this instance ssh to it and paste all the commands manually
```

#### Nexus Server Setup

- Create `NexusServer` with the following settings and the userdata script.
```sh
Name: NexusServer
AMI: CentOS 7
InstanceType: t2.medium
SecGrp: NexusSG
KeyPair: CREATE NEW KEY PAIR, FOR EXAMPLE NexusKey.pem
Additional Details: copy from userdata/nexus_setup.sh
```

#### SonarQube Server Setup

- Create `SonarServer` with the following settings and the userdata script.
```sh
Name: SonarServer
AMI: Ubuntu 20.04
InstanceType: t2.medium
SecGrp: SonarSG
KeyPair: CREATE NEW KEYPAIR, FOR EXAMPLE SonarKey.pem
Additional Details: copy it from userdata/sonar_setup.sh
```

### 3. Post Installation Steps

#### For Jenkins Server:

- SSH to Jenkins server and check system status for Jenkins. 
- To unlock Jenkins get the admin password from this directory `/var/lib/jenkins/secrets/initialAdminPassword` 
```sh
sudo -i
systemctl status jenkins
cat /var/lib/jenkins/secrets/initialAdminPassword 
```
- Go to `http://<PUBLIC_IP_of_JenkinsServer>:8080`, enter copied initialAdminPasswrd.
- Install the following plugins:
```sh
Github Integration
Maven Integration
Nexus Artifact Uploader
SonarQube Scanner
Slack Notification
Build Timestamp
```
- Create a user, take a note of a password and username you will select.

#### For Nexus Server:

- SSH to Nexus server and check system status for Nexus.
```sh
sudo -i
systemctl status nexus
```
- Go to `http://<PUBLIC_IP_FOR_NexusServer>:8081`, then sign in. Initial password will be located `/opt/nexus/sonatype-work/nexus3/admin.password`
```sh
cat /opt/nexus/sonatype-work/nexus3/admin.password
```
- Username is `admin`, password is taken from the previous step. After logging in setup new password and select `Disable Anonymous Access`.

#### For SonarQube Server:

- Go to `http://<PUBLIC_IP_FOR_SonarServer>`.

- Login with username `admin` and password `admin`.

### 4. Install JDK8 and Maven.

- This application uses both JDK11 and JDK8
- Go to `Manage Jenkins` -> `Tools`
- ssh to Jenkins EC2 and find JAVA_HOME path in /usr/lib/jvm/
- Find JDK section and click Add JDK:
```sh
Name: OracleJDK11
untick Install Automatically
JAVA_HOME: /usr/lib/jvm/java-1.11.0-openjdk-amd64
```

- Add JDK8 - it is not installed yet, so look at the next step.
```sh
Under JDK -> Add JDK
Name: OracleJDK8
untick Install Automatically
JAVA_HOME: LOOK AT THE NEXT STEP
```
- Currently Jenkins has JDK-11 (this app can also work with it), so ssh into Jenkins server and install JDK-8. Get the PATH to JDK-8 and paste to JAVA_HOME in the previous step. 
- After installation `JAVA_HOME` for JDK-8 is `/usr/lib/jvm/java-1.8.0-openjdk-amd64`
```sh
sudo apt update -y
sudo apt install openjdk-8-jdk -y
sudo -i
ls /usr/lib/jvm
### both jdk-11 and jdk-8 should be in this path ###
java-1.11.0-openjdk-amd64  java-11-openjdk-amd64  openjdk-11
java-1.8.0-openjdk-amd64   java-8-openjdk-amd64
``` 

- Next set up Maven - click Add Maven in Maven section.
```sh
Name: MAVEN3
Version: 3.9.3
``` 

### 5. Install Plugins for CI

- Go to `Manage Jenkins` -> `Manage Plugins` -> `Available`
- Install without restart:
```sh 
Nexus Artifact Uploader
SonarQube Scanner
Build Timestamp
Pipeline Maven Integration
Pipeline Utility Steps
```

### 6. Create the first Pipeline

- Go to `Dashboard` -> `New Item` in Jenkins, choose Pipeline and name it as you like
- In Pipeline section paste the following script:
```groovy
pipeline {
	agent any
	tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

	stages {
	    stage('Fetch code') {
            steps {
               git branch: 'vp-rem', url: 'https://github.com/devopshydclub/vprofile-repo.git'
            }

	    }

	    stage('Build'){
	        steps{
	           sh 'mvn install -DskipTests'
	        }

	        post {
	           success {
	              echo 'Now Archiving it...'
	              archiveArtifacts artifacts: '**/target/*.war'
	           }
	        }
	    }

	    stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }
	}
}
```
- Save and click `Build now` - you can check logs. The pipeline should be built successfully.
- In Workspace go to /target/ - you should see the artifact.

### 7. Code Analysis

- First integrate SonarQube with Jenkins and install SonarQube Scanner tool.
- Go to `Manage Jenkins` -> `Tools` -> `SonarQube Scanner` -> `Add SonarQube Scanner`
```sh
Name: sonar4.7
Check: Install automatically
Version: SonarQube Scanner 4.7.0.*
```
- Go to `Configure system` -> `SonarQube servers`
```sh
Check: Environmental variables
Name: sonar
Server URL: http://FETCH SONARQUBE PUBLIC OR PRIVATE IP - REMEMBER THAT PUBLIC IP WILL CHANGE EACH TIME YOU WILL POWER OFF AND ON AN INSTANCE
Server authentication token: save and LOOK FOR THE NEXT STEP
```
- Go to SonarQube -> `My Account` -> `Security` -> `Generate Token` and copy its value in a next step
```sh
Generate Tokens: jenkins
```
- In Jenkins click add `Jenkins` in `Server authentication token`:
```sh
Kind: Secret text
Secret: COPY YOUR SECRET HERE
ID: MySonarToken
Description: MySonarToken
```

- Create a new Jenkinsfile:
```groovy
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}
```

- You can either update a previous item with this pipeline or create a new one.
- Run this pipeline.

### 8. Sonar Analysis

- Now let the SonarQube scan the code and upload results to its server.
- In the previous pipeline add a new stage:
```groovy
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

    }
}
```

- Run the pipeline and after its success go to SonarQube and `Projects` - you can see that the project has passed the Quality Gate.
- You can create your own Quality Gates - this will be explained in the next step.

### 9. Quality Gates

- Add another step in the pipeline:
```groovy
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
```
- In SonarQube go to `Quality Gates` -> `Create`:
```sh
Name: vprofile-QG
Add condition: 
Check: On overall code
Quality Gate fails when: Bugs
Value: 60 to make it fail, 100 to make it pass
```
- Go to your project -> `Project Settings` -> `Quality Gate` -> Choose the newly created one (vprofile-QG).
- Now you need to create a Webhook to allow SonarQube send a message to Jenkins.
- Go to `Project Settings` -> `Webhooks` -> `Create`
```sh
Name: jenkinrepos-ci-webhook
URL: http://JENKINS PRIVATE IP:8080/sonarqube-webhook
```
- Make sure that you set Jenkins Security Group to allow traffic on 8080 from anywhere or from SonarQube Security Group.
- Now test your pipeline: depending on a Value set in a new Quality Gate your pipeline will fail or pass. You can check the details in SonarQube Project section.
- Change the Quality Gate to the default one or increase the value to 100 in a currently set one.

### 10. Upload the artifact to Nexus Repository

- Go to `http://NEXUS PUBLIC IP:8081` and sign in with login `admin` and password that you set previously.
- Go to `Settings` -> `Repositories` -> `Create Repository` -> `maven2 (hosted)`
```sh
Name: vprofile-repo
```
- Create repository.
- Go to Jenkins -> `Manage Jenkins` -> `Manage Credentials` -> `Stores scoped to Jenkins` -> `Jenkins` -> `Global credentials` -> `Add Credentials` 
```sh
Username: admin
Password: YOUR NEXUS PASSWORD
ID: nexuslogin
Description: nexuslogin
```
- Add new stage in a pipeline or create a new pipeline in Jenkins:
```groovy
pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: 'NEXUS PRIVATE IP:8081',
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'vprofile-repo',
                  credentialsId: 'nexuslogin',
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
    				]
 				)
            }
        }
    }
}
```
- Go to `Manage Jenkins` -> `Configure System` -> `Build Timestamp` -> `Pattern` 
```
yy-MM-dd_HH-mm [or set it to any other pattern you like]
```
- And save.
- You can also change the SonarQube servers URL to private, if you previously used the public one and make sure your token is selected.
In SonarQube Webhook you can also update the Jenkins IP to the private one, if you used its public one before.
- Run the pipeline, after its successfull run go to Nexus -> `Browse` -> `vprofile-repo` -> you should see the artifact there with attached timestamp pattern. You can run the pipeline a few more times and check the artifacts names. From there you can download your artifacts (Summary -> Path -> your artifact).

### 11. Slack integration - adding notifications

- If you do not have an account on Slack, create one. 
- Create a new workspace on Slack and name it, for example `vprofilecicd`.
- Create a new channel, for example `jenkinscicd`.
- Now create a token to allow Jenkins authenticate into this workspace - search for 'add apps to slack' on Google and look for `Jenkins CI` app. Click on it and choose your previously created channel (jenkinscicd). Click on `Add Jenkins CI integration`.
Copy and store the token that got created. Click on `Save settings`.
- Go to Jenkins -> `Manage Jenkins` -> `Plugins` -> search for `Slack Notification` and install without a restart.
- Go to `Configure System` -> `Slack`
```
Workspace: vprofilecicd-[HERE INSERT UNIQUE CHARACTERS FOR YOUR WORKSPACE - CHECK IT ON YOUR SLACK ACCOUNT]
Credentials: Jenkins
Kind: Secret text
Secret: YOUR TOKEN FROM THE PREVIOUS STEP
ID: slacktoken
Description: slacktoken
Save and select your token
Default channel: #jenkinscicd
```
- Test connection, if you get a success save.
- Add the post installation stage in your pipeline and a key map for colors: 
```groovy
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}

    stages{
        stage('Print error'){
            steps{
                sh 'fake comment'
            }
        }
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: 'INSERT YOUR NEXUS PRIVATE IP:8081',
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'vprofile-repo',
                  credentialsId: 'nexuslogin',
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    } 
}
```
- Run your pipeline -  check for a success notification on your Slack channel. If everything goes right - congratulations on finishing this project! 
You can also make some mistakes in the Jenkinsfile and run the pipeline to see a Failure notification on your Slack channel.