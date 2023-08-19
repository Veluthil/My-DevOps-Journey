## Comprehensive DevOps Infrastructure Setup Guide

Welcome to the comprehensive step-by-step guide for setting up a powerful DevOps infrastructure using various AWS services. This guide will walk you through each critical step to ensure you create a robust and efficient architecture for your applications.

### Prerequisites

Before diving into the setup process, make sure you have the following prerequisites ready:

- An AWS Account
- Default Virtual Private Cloud (VPC)
- A registered public domain in Route53
- Maven installed on your system
- JDK 8 or higher

### Architecture Overview

This guide will help you establish a multi-faceted architecture using AWS services. By following the instructions below, you'll set up a seamless DevOps environment that incorporates various technologies and tools:

1. **Key Pair Creation for Beanstalk EC2 Login:**
   - Go to the AWS EC2 console and generate a key pair to facilitate secure login to EC2 instances. Remember to save the private key for future use.

2. **Backend Service Security Groups:**
   - Create dedicated security groups for backend services such as ElastiCache, RDS, and ActiveMQ.
   - Remove the default inbound rule and replace it with an "Allow All Traffic" rule for better control.

3. **Setting Up RDS Database:**
   - Create a Subnet Group by selecting all available Availability Zones and Subnets.
   - Configure a DB Parameter Group with the `mysql5.7` family.
   - Create a MySQL database with engine version `5.7.33` using the Free-Tier template.
   - Configure instance details, master username, and an auto-generated password.
   - Choose an instance type of `db.t2.micro` for cost efficiency.
   - Associate a pre-made subnet group and security group.
   - Ensure no public access and use password authentication.
   - Configure additional settings, including the initial DB name.
   - After creating the DB, note the auto-generated password and credentials.

4. **ElastiCache Configuration:**
   - Set up a Memcached Parameter Group with your desired name and the `memcached1.4` family.
   - Create a Memcached Subnet Group across all Availability Zones and Subnets.
   - Establish a Memcached Cluster, specifying the engine version and node type as `cache.t2.micro`.
   - Choose a single node configuration for simplicity.
   - Associate the backend security group created earlier.

5. **Creating Amazon MQ:**
   - Create a RabbitMQ broker instance using the single-instance-broker configuration.
   - Assign a unique name to the broker and select the desired instance type.
   - Record the generated username and password for future reference.
   - Configure the broker for private access within the default VPC.
   - Associate it with the backend security group created in step 2.

6. **Database Initialization:**
   - Copy the RDS endpoint for future use.
   - Launch a new EC2 instance using the Ubuntu 18.04 image, and provide appropriate security group settings.
   - Access the instance using your key pair, and perform necessary updates and installations.
   - Adjust the backend security group's inbound rule to allow connections on port 3306.
   - Connect to the RDS database using the provided credentials and perform verification tasks.
   - Clone the source code repository, navigate to the correct directory, and initialize the MySQL database using provided backup data.
   - Use the following commands:
     ```bash
     git clone https://github.com/Veluthil/Vprofile-Project.git
     cd Vprofile-Project
     cd src/main/resources
     mysql -h <your-rds-endpoint> -u admin -p<db-password> accounts < db_backup.sql
     mysql -h <your-rds-endpoint> -u admin -p<db-password> accounts
     show tables;
     ```

7. **Elastic Beanstalk Environment Setup:**
   - Collect endpoint details for all backend services from the AWS console; these will be used in the `application.properties` file.
   - Create an Elastic Beanstalk environment using the Tomcat platform, tailored for your application's needs.
   - Customize capacity, load balancing settings, and security configurations.

8. **Updating Backend Security Group and ELB:**
   - Allow connections from the Elastic Beanstalk application security group to the backend services security group.
   - Update the backend security group to enable communication on ports 3306, 11211, and 5671.
   - Configure the Elastic Beanstalk environment's configuration tab, including adding a listener for HTTPS and defining health check paths.

9. **Building and Deploying Artifact:**
   - Update the `application.properties` file within your cloned project, ensuring accurate endpoint URLs and credentials.
   - Navigate to the project's root directory containing the `pom.xml` file.
   - Execute the following Maven commands to build the artifact:
     ```bash
     mvn install
     ```
   - Upload the generated artifact (such as `target/vprofile-v2.war`) to your Elastic Beanstalk environment.
   - This action will also automatically upload the artifact to the S3 bucket associated with Elastic Beanstalk.
   - Within the AWS console, select the uploaded application and initiate the deployment process.

10. **Creating DNS Record in Route53:**
    - Set up an A record in the Route53 service, pointing to the Elastic Beanstalk endpoint.
    - This step will provide a user-friendly domain name to access your application securely.

11. **Cloudfront Distribution for Content Delivery:**
    - Create a Cloudfront distribution to enhance content delivery through caching.
    - Configure the distribution to redirect HTTP requests to HTTPS.
    - Specify alternate domain names and SSL certificate settings for added security.
    - Select a suitable security policy, such as TLSv1.

12. **Functionality Verification and Clean-up:**
    - Access your application using the provided domain names and verify its functionality.
    - Check both HTTP and HTTPS versions of the application.

13. **Project Clean-up:**
    - When you're satisfied with your exploration and learning, ensure to delete or terminate all created resources to avoid unnecessary charges.

Thank you for taking on this comprehensive DevOps infrastructure setup journey. By following these steps, you're not only gaining hands-on experience but also establishing a robust foundation for your future DevOps endeavors.