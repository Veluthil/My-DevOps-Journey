# Project 2 - Automated Setup of Multi Tier App

In the first project I have set up the 3-Tier Application manually, but in this one I will create bash scripts to automate our VM creation/provisioning through Vagrantfile.

## Prerequisites

 * Oracle VM VirtualBox Manager
 * Vagrant 2.3.4 or later
 * Vagrant plugins
 * Git
 * IDE (SublimeText, VSCode, etc)

## 1. Prepare Bash Scripts for VMs
All of the scripts are available in this repository - `Project 2 - Automated Setup of Multi Tier App\vagrant\Automated_provisioning`.
### Bash Script for the database
- Create `mysql.sh` file for the database.
### Bash Script for Memcache
- Create `memcache.sh` for provisionining our memcached server.
### Bash Script for RabbitMQ
- `rabbitmq.sh` for RabbitMQ.
### Bash Script for Application
- Create a Bash Script for provisioning Tomcat server - `tomcat_ubuntu.sh` for Ubuntu OS and `tomcat.sh`.
### Bash Script for Nginx server
- Create a final Bash Script `nginx.sh` for provisioning Nginx server which will forward requests to the backend application.

## 2. Prepare Bash Scripts for VMs
- Clone the repository:
```sh
git clone https://github.com/Veluthil/Vprofile-Project.git
```

- Go to the directory where Vagrantfile exists and install the plugin below:
```sh
vagrant plugin install vagrant-hostmanager
```

- After the plugin got installed, run this command to setup all the VMs.
```sh
vagrant up
```
Be aware that this will take some time.

## 3. Validate the Application from the browser
- Validate frontend of the application using hostname given in Vagrantfile or IP address of `web01`.
- Check backend services. 
- Validate RabbitMq service.
- Check our memcache and database.
- To stop VMs use the command below:
```sh
vagrant halt
```
- To start bring up the VMs once more use:
```sh
vagrant up 
```
- Once you are done, destroy the VMs by using:
```sh
vagrant destroy
```

