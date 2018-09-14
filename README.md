# Linux-Server-Configuration

You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.


##Things to Do
Get your server.

1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.
2. Follow the instructions provided to SSH into your server.

Secure your server.

3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

Give grader access.
In order for your project to be reviewed, the grader needs to be able to log in to your server.

6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.

Prepare to deploy your project.

9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.

11. Install and configure PostgreSQL:

Do not allow remote connections
Create a new database user named catalog that has limited permissions to your catalog application database.

12. Install git.

Deploy the Item Catalog project.

13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!


### Details of project 

Server IP Address: 18.216.171.143
Server Alias: ec2-18-216-171-143.us-east-2.compute.amazonaws.com
SSH server access port: 2200
SSH login username: grader
Application URL: http://18.216.171.143

## Step by Step Walkthrough
We will do things step by step.will include the possible error associated with step and solution to it.

### 1  - Create Development Environment: Launch Virtual Machine and SSH into the server
 

1. Login to https://aws.amazon.com/.
2. Create instance and select ubuntu.
2. Download private keys and write down your public IP address.
3. Move the private key file into the folder
  `C:\Users\HOME\Desktop\key`
4. Set file rights (only owner can write and read.):  
  `$ chmod 600 C:\Users\HOME\Desktop\key\MyNewKey`
5. SSH into the instance:  
  `$ ssh -i C:\Users\HOME\Desktop\key\MyNewKey ubuntu@PUPLIC-IP-ADDRESS`

### 2 - User Management: Create a new user and give user the permission to sudo


1. Create a new user:  
  `$sudo adduser grader`
2. Give new user the permission to sudo
  1. Open the sudo configuration:  
    `$ sudo visudo`
  2. Add the following line below `root`:  
    `grader ALL=(ALL:ALL) ALL`


### 3 - Update and upgrade all currently installed packages

1. Update the list of available packages and their versions:  
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:  
  `$ sudo sudo apt-get upgrade`

### 4 Set ssh login using keys

1. generate key on local machine ssh-keygen.This Key can be used to login as grader@ip-address.
  `
  $ ssh-keygen
  $ cat /c/Users/HOME/.ssh/id_rsa.pub 
  `
Copy content of id_rsa.pub 

2. On grader virtual machine
  `
  $ su - grader
  $ mkdir .ssh
  $ touch .ssh/authorized_keys
  $ vim .ssh/authorized_keys
  `
  Copy the public key generated on your local machine to this file and save

3. reload SSH using `sudo service ssh restart`

4. login with 
  
  `ssh -i ~/.ssh/[privateKeyFilename] grader@ip-address`


### 5 - Change the SSH port from 22 to 2200 and configure SSH access
Source: [Ask Ubuntu][8]  

1. Change ssh config file:
  1. Open the config file:  
    `$ sudo /etc/ssh/sshd_config` 
  2. Change to Port 2200.
  
2. Add custom tcp port 2200 with amazon Lightsail.

3. Restart the ssh server  `sudo service ssh restart`

4. check by loggin in by adding port 2200 `ssh root@http://18.216.171.143/ -p 2200`


### 6 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  

`
sudo ufw allow ssh
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 
sudo ufw status
`

Remember to change inbound rules in your ec2 console.

### 7 - Configure the local timezone to UTC


1. Open the timezone selection dialog:  
  `$ sudo dpkg-reconfigure tzdata`
2.Select time zone and country

### 8 - Install and configure Apache to serve a Python mod_wsgi application

1. `sudo apt-get update`
2. Install Apache web server:  
  `$ sudo apt-get install apache2`
3. Open a browser and open your public ip address, e.g. http://http://18.222.216.108// - It should  display apache index.html page.
4.  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
5.  `$ sudo a2enmod wsgi`
6.  `$ sudo service apache2 start`
7.  `$ sudo apt-get install git`
8.  `$ sudo service apache2 restart`  


### 9 - Setting up repository
1. $ cd /var/www
2. $ sudo mkdir catalog
3. $ sudo chown -R grader:grader catalog
4. $ cd catalog
5. $ git clone https://github.com/rrai203/ItemCatalog.git 

### 10 - Setting up virtual Enviroment
1. .$ sudo apt-get install python-pip
2. $ sudo pip install virtualenv
3. $ sudo virtualenv venv
4. $ source venv/bin/activate
5. Keep in mind to give permission to the venv folder.If persmission not given packages will be not get installed
   `$ sudo chmod -R 777 venv`

### 11 - Installing dependencies for virtualenv

Be sure that virtual enviroment is activated while installing these dependencies.

1. pip install httplib2
2. pip install requests
3. sudo pip install --upgrade oauth2client
4. sudo pip install sqlalchemy
5. pip install Flask-SQLAlchemy
6. sudo pip install flask-seasurf
7. sudo apt-get install python-psycopg2

### 12 - configuring our virtual host

1. $ sudo nano /etc/apache2/sites-available/catalog.conf
2. Add the following content 

```
<VirtualHost *:80>
   ServerName 18.222.216.108
   ServerAlias ec2-18-222-216-108.us-east-2.compute.amazonaws.com
   ServerAdmin grader@18.222.216.108
   WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
   WSGIProcessGroup catalog
   WSGIScriptAlias / /var/www/catalog/catalog.wsgi
   <Directory /var/www/catalog/catalog/>
       Order allow,deny
       Allow from all
   </Directory>
   Alias /static /var/www/catalog/catalog/static
   <Directory /var/www/catalog/catalog/static/>
       Order allow,deny
       Allow from all
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

2. Enable it by `$ sudo a2ensite catalog`

## 13 - configuring wsgi file 

1. $ cd /var/www/catalog/
2. $ sudo nano catalog.wsgi

3.Add the following lines 

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'secret'
```

4. Rename to project.py to __init__.py
5. Change the path of client_secrets.json in project.py.


## 14 - Setting up database

$ sudo apt-get install libpq-dev python-dev.
$ sudo apt-get install postgresql postgresql-contrib
$ sudo -u postgres psql

create a new user catalog
1. `# CREATE USER catalog WITH PASSWORD 'catalog';`
2. `# ALTER USER catalog CREATEDB`
3. `# CREATE DATABASE catalog WITH OWNER catalog;`
4. `# \c catalog`
5. `# REVOKE ALL ON SCHEMA public FROM public;`
6. `# GRANT ALL ON SCHEMA public TO catalog;`
7. \q 
8. deactivate
9. Make changes to databse_setup.py 

```
create_engine('postgresql://catalog:catalog-password@localhost/catalog')
```
10. Restart Apache server `$ sudo service apache2 restart`

## 15 - Enabling google oauth login

1. go to google developer console.
2. edit the credentials.
3. authorize javascript origin add your ip address.
4. Download the JSON file and update it in client_secret.json. 

Restart apache server’s

`$ sudo service apache2 restart`


## References

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
