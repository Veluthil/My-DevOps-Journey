# Project 1 - Multi Tier Web Application Setup Locally

Welcome to this project where I've orchestrated a 3-Tier Application Stack setup involving MySQL/MariaDB, Memcached, RabbitMQ, Tomcat, and Nginx. To achieve this, I've manually created VMs using a Vagrantfile and utilized a bash script for configuring Linux servers.

The project utilizes Maven to build the artifact, which is then deployed to the Tomcat server.

The Java project 'Vprofile' was created by Imran Teli, the instructor of 'DevOps Beginners to Advanced with Projects - 2023'.

### Prerequisites

Ensure you have the following tools and software installed:

1. Oracle VM VirtualBox.
2. Vagrant 2.3.4 or later.
3. Vagrant plugins:
   - Install the plugin: `vagrant plugin install vagrant-hostmanager`.
4. Git bash or an equivalent editor.
   
### VM Setup

Follow these steps to set up your virtual machines:

1. Clone the source code repository https://github.com/Veluthil/Vprofile-Project.git (or https://github.com/hkhcoder/vprofile-project.git).
2. Navigate into the repository folder.
3. Move to the `vagrant/Manual_provisioning` directory (This file is also accessible within this Project 1 at the following location: '\vagrant\Manual provisioning\Vagrantfile' directory).

Now, bring up the virtual machines using the command:

```sh
vagrant up
```

**Note:** The setup might take some time due to various factors. If the setup halts midway, run the `vagrant up` command again. The hostname and `/etc/hosts` entries for all VMs will be updated automatically.

## Database
In this project, MySQL serves as the chosen database management system. If you're working on a Linux Ubuntu 14.04 environment, you can follow these steps to set up MySQL:

- Update the package index: `$ sudo apt-get update`
- Install MySQL server: `$ sudo apt-get install mysql-server`

Next, navigate to the following file:
- `/src/main/resources/db_backup.sql`
- The `db_backup.sql` file is a MySQL dump containing the required database schema. To import it into your MySQL server, execute the following command:
  `> mysql -u <user_name> -p accounts < db_backup.sql`

### Service Provisioning

Services will be provisioned in the following order:

1. Nginx: Web Service
2. Tomcat: Application Server
3. RabbitMQ: Broker/Queuing Agent
4. Memcache: DB Caching
5. ElasticSearch: Indexing/Search Service
6. MySQL: SQL Database

Follow this order to ensure a successful setup.

### MySQL Setup

1. Log in to the database VM:

```sh
vagrant ssh db01
```

2. Verify the hosts entry:

```sh
cat /etc/hosts
```

3. Update the operating system with the latest patches:

```sh
sudo yum update -y
```

4. Set up the repository:

```sh
sudo yum install epel-release -y
```

5. Install MariaDB package:

```sh
sudo yum install git mariadb-server -y
```

6. Start and enable the MariaDB server:

```sh
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

7. Run the MySQL secure installation script:

```sh
sudo mysql_secure_installation
```

8. Set up the database:

```sh
mysql -u root -padmin123
```

```sql
CREATE DATABASE accounts;
GRANT ALL PRIVILEGES ON accounts.* TO 'admin'@'%' IDENTIFIED BY 'admin123';
FLUSH PRIVILEGES;
EXIT;
```

9. Download the source code and initialize the database:

```sh
git clone -b main https://github.com/Veluthil/Vprofile-Project.git
cd vprofile-project
mysql -u root -padmin123 accounts < src/main/resources/db_backup.sql
mysql -u root -padmin123 accounts
```

10. Restart the MariaDB server:

```sh
sudo systemctl restart mariadb
```

11. Start the firewall and allow port 3306:

```sh
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
sudo systemctl restart mariadb
```

### Memcache Setup

1. Install, start, and enable Memcache on port 11211:

```sh
sudo dnf install epel-release -y
sudo dnf install memcached -y
sudo systemctl start memcached
sudo systemctl enable memcached
sudo systemctl status memcached
sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
sudo systemctl restart memcached
```

2. Start the firewall and allow port 11211 and 11111:

```sh
firewall-cmd --add-port=11211/tcp
firewall-cmd --runtime-to-permanent
firewall-cmd --add-port=11111/udp
firewall-cmd --runtime-to-permanent
```

### RabbitMQ Setup

1. Log in to the RabbitMQ VM:

```sh
vagrant ssh rmq01
```

2. Verify the hosts entry:

```sh
cat /etc/hosts
```

3. Update the operating system with the latest patches:

```sh
sudo yum update -y
```

4. Set up the EPEL repository:

```sh
sudo yum install epel-release -y
```

5. Install dependencies and RabbitMQ:

```sh
sudo yum install wget -y
sudo yum install rabbitmq-server -y
sudo systemctl enable --now rabbitmq-server
sudo systemctl start rabbitmq-server
```

6. Configure RabbitMQ:

```sh
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
sudo rabbitmqctl add_user test test
sudo rabbitmqctl set_user_tags test administrator
sudo systemctl restart rabbitmq-server
```

### Tomcat Setup

1. Log in to the Tomcat VM:

```sh
vagrant ssh app01
```

2. Verify the hosts entry:

```sh
cat /etc/hosts
```

3. Update the operating system with the latest patches:

```sh
sudo yum update -y
```

4. Set up the repository:

```sh
sudo yum install epel-release -y
```

5. Install dependencies and Java:

```sh
sudo yum install java-11-openjdk java-11-openjdk-devel -y
sudo yum install git maven wget -y
```

6. Download and install Tomcat:

```sh
cd /tmp/
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
tar xzvf apache-tomcat-9.0.75.tar.gz
```

7. Configure Tomcat user and ownership:

```sh
sudo useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
sudo cp -r /tmp/apache-tomcat-9.0.75/* /usr/local/tomcat/
sudo chown -R tomcat.tomcat /usr/local/tomcat
```

8. Set up the Tomcat service:

```sh
sudo vi /etc/systemd/system/tomcat.service
```

Add the following content:

```ini
[Unit]
Description=Tomcat
After=network.target

[Service]
User=tomcat
WorkingDirectory=/usr/local/tomcat
Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINA_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
SyslogIdentifier=tomcat-%i

[Install]
WantedBy=multi-user.target
```

9. Reload systemd files and start the service:

```sh
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl enable tomcat
```

10. Set up the firewall and allow port 8080:

```sh
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --zone=public --add-port=8080/tcp --

permanent
sudo firewall-cmd --reload
```

### Code Build & Deploy (app01)

1. Download the source code:

```sh
git clone -b main https://github.com/Veluthil/Vprofile-Project.git
```

2. Update configuration:

```sh
cd vprofile-project
vim src/main/resources/application.properties
```

3. Build the code:

```sh
mvn install
```

4. Deploy the artifact:

```sh
sudo systemctl stop tomcat
sudo rm -rf /usr/local/tomcat/webapps/ROOT*
sudo cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
sudo systemctl start tomcat
sudo chown tomcat.tomcat /usr/local/tomcat/webapps -R
sudo systemctl restart tomcat
```

### Nginx Setup

1. Log in to the Nginx VM:

```sh
vagrant ssh web01
```

2. Verify the hosts entry:

```sh
cat /etc/hosts
```

3. Update the operating system:

```sh
sudo apt update
sudo apt upgrade
```

4. Install Nginx:

```sh
sudo apt install nginx -y
```

5. Create an Nginx conf file:

```sh
sudo vi /etc/nginx/sites-available/vproapp
```

Add the following content:

```nginx
upstream vproapp {
    server app01:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://vproapp;
    }
}
```

6. Remove the default Nginx conf:

```sh
sudo rm -rf /etc/nginx/sites-enabled/default
```

7. Create a link to activate the website:

```sh
sudo ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
```

8. Restart Nginx:

```sh
sudo systemctl restart nginx
```

By following these steps, you've automated the setup of your multi-tier application. Remember to follow the order of provisioning services for a successful setup.
