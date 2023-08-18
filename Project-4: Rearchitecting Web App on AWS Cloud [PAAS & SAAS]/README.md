## Pre-requisites:

- AWS Account
- Default VPC
- Route53 Public Registered Name
- Maven
- JDK8

## Architecture

1. **Create a Key pair for Beanstalk EC2 Login:**
    - Go to the EC2 console and create a key pair; save it for later.

2. **Create Security Groups for backend services (ElastiCache, RDS, and ActiveMQ):**
    - Create a Security Group. Delete the default inbound rule and allow All Traffic from this SG.

3. **Create RDS Database:**
    - Create Subnet Group:
      Name: (Choose your name)
      AZ: Select All
      Subnet: Select All
    - Create Parameter Group:
      Parameter group family: mysql5.7
      Type: DB Parameter Group
      Group Name: (choose your name)
    - Create Database:
      Method: Standard Create
      Engine Options: MySQL
      Engine version: 5.7.33
      Templates: Free-Tier
      DB Instance Identifier: choose your name
      Master username: admin
      Password: Auto generate psw
      Instance Type: db.t2.micro (to keep it free tier)
      Subnet grp: choose your pre-made subnet group
      SecGrp:  choose your pre-made security group 
      No public access
      DB Authentication: Password authentication
      Additional Configuration
      Initial DB Name: accounts
    - After creating the DB, click View credential details and make a note of an auto-generated password.

4. **Create ElastiCache:**
    - Create Parameter Group:
      Name: choose your name
      Family: memcached1.4
    - Create Subnet Group:
      Name: your name
      AZ: Select All
      Subnet: Select All
    - Create Memcached Cluster:
      Go to Get Started -> Create Clusters -> Memcached Clusters
      Name: your name
      Engine version: 1.4.5
      Parameter Grp: select your name
      NodeType: cache.t2.micro
      SecGrp: choose your backend security group created in point 2

5. **Create Amazon MQ:**
    - Engine type: RabbitMQ
    - Single-instance-broker
    - Broker name: choose your name
    - Instance type: mq.t3.micro
    - Username: your username
    - Password: your password

6. **Database Initialization:**
    - Copy RDS endpoint.
    - Create a new EC2 instance to initialize the DB.
    - SSH to your Ubuntu EC2 instance using your key-pair and public IP.
    - Run the following commands:
        ```sh
        apt update -y
        apt upgrade -y
        apt install mysql-client -y
        ```
    - Update your backend SG Inbound rule to allow connection on port 3306.

7. **Create Elastic Beanstalk Environment:**
    - Collect endpoints for all the backend services.
    - Create Application:
        - Name: create your name 
        - Platform: Tomcat
        - Configuration: Custom configuration
        - Capacity: LoadBalanced, Min:2, Max:4, InstanceType: t2.micro
        - Rolling updates and deployments: Deployment policy: Rolling, Percentage: 50%
        - Security: EC2 key pair: your key-pair

8. **Update Backend SG & ELB:**
    - Update your backend-SG to allow connection from the application SG.
    - Add Listener on port 443 (HTTPS).

9. **Build and Deploy Artifact:**
    - Update application.properties file with correct endpoints and credentials.
    - Go to root directory and run:
        ```sh
        mvn install
        ```
    - Upload the artifact (target/vprofile-v2.war) to your Elastic Beanstalk.

10. **Create DNS Record in Route53 for Application:**
    - Create an A record aliasing Elastic Beanstalk endpoint.

11. **Create Cloudfront Distribution for CDN:**
    - Origin Domain: DNS record name.
    - Viewer protocol: Redirect HTTP to HTTPS.
    - Alternate domain name: DNS record name.
    - SSL Certificate: 
    - Security policy: TLSv1

12. **Clean-up:**
    - Delete/terminate everything you created in reverse order.
    
Thank you!
