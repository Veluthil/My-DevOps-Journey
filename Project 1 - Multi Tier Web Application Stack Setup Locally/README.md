Welcome to this project where I've orchestrated a 3-Tier Application Stack setup involving MySQL/MariaDB, Memcached, RabbitMQ, Tomcat, and Nginx. To achieve this, I've manually created VMs using a Vagrantfile and utilized a bash script for configuring Linux servers.

The project utilizes Maven to build the artifact, which is then deployed to the Tomcat server.

## Setup Flow:

1. VMs were provisioned using Vagrant within Oracle VirtualBox.

2. After logging into each machine, I manually executed shell commands via Bash to configure services.

3. I meticulously set up the required services.

4. Once the complete stack was up and running, I verified its functionality through the browser.

5. The workflow involved Nginx forwarding requests to the Tomcat server, which then directed them to the RabbitMQ message broker, further to Memcached, and finally to the MySQL server. Notably, queries executed on the Tomcat server were cached in Memcache for optimization.

## Prerequisites
- JDK 1.8 or later
- Maven 3 or later
- MySQL 5.6 or later

## Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- MySQL

## Database
In this project, MySQL serves as the chosen database management system. If you're working on a Linux Ubuntu 14.04 environment, you can follow these steps to set up MySQL:

- Update the package index: `$ sudo apt-get update`
- Install MySQL server: `$ sudo apt-get install mysql-server`

Next, navigate to the following file:
- `/src/main/resources/db_backup.sql`
- The `db_backup.sql` file is a MySQL dump containing the required database schema. To import it into your MySQL server, execute the following command:
  `> mysql -u <user_name> -p accounts < db_backup.sql`

Feel free to explore this project and delve into its intricacies. Happy coding!
