# Project 5 - Continuous Integration with Jenkins, Nexus, Sonarqube and Slack


## Pre-requisities:

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
- Go to `http://<PUBLIC_IP_FOR_NexusServer>:8081`, then sign-in. Initial password will be located `/opt/nexus/sonatype-work/nexus3/admin.password`
```sh
cat /opt/nexus/sonatype-work/nexus3/admin.password
```
- Username is `admin`, password is taken from the previous step. After logging in setup new password and select `Disable Anonymous Access`.

#### For SonarQube Server:

- Go to `http://<PUBLIC_IP_FOR_SonarServer>`.

- Login with username `admin` and password `admin`.