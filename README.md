# Linux-Server-Configuration-Udacity

This project is deploying the Item Catalog Application to be hosted on a Ubuntu Linux server on an Amazon Lightsail instance. To set this up, please go through the following instructions. The application is deployed live on http://18.138.251.117 and SSH port 2200.

---

### Create a Ubuntu Linux Server instance on Amazon Lightsail

* Log in to Lightsail or create an AWS account

* Click Create an instance

* Select Linux/Unix platform

* Select OS Only and Ubuntu for blueprint

* Select an instance plan

* Name your instance

* Click Create

---

### SSH into your Server instance

Within the instance you just created, you can click Connect using SSH to SSH into your server instance. Alternative, you
can follow these steps to connect using your own SSH client:

```
1. In your Lightsail account, click Account on the top navigation bar and click Account again

2. Click SSH keys tab

3. Click download to download the private key. It should be a .pem file

4. Create a new .rsa file e.g. lightsail_key.rsa within your local ~/.ssh folder

5. Copy and paste content from downloaded private key file into .rsa file

6. Set file permission as owner only using $ chmod 600 ~/.ssh/lightsail_key.rsa

7. SSH into the instance using $ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.138.251.117
```

Next, we'll update all currently installed packages. In either the browser-based ssh client launched from Lightsail or your own ssh client:

* Run ```sudo apt-get update``` to update packages.

* Run ```sudo apt-get upgrade``` to install the newest versions of the packages.

* Set this for any future updates: ```sudo apt-get dist-upgrade```

* To change the SSH port. Run ```sudo nano /etc/ssh/sshd_config``` to open up the configuration file.

* Change the port number from 22 to 2200 in this file.

* Save and exit the file.

* Restart SSH using ```sudo service ssh restart```

---
### Configure the firewall
* Check the firewall status using $ sudo ufw status

* Set the default firewall to deny all incomings using ```sudo ufw default deny incoming```

* Set the default firewall to allow all outgoings using ```sudo ufw default allow outgoing```

* Allow incoming TCP packets on port 2200 to allow SSH using ```sudo ufw allow 2200/tcp```

* Allow incoming TCP packets on port 80 to allow www using ```sudo ufw allow www```

* Allow incoming UDP packets on port 123 to allow NTP using ```sudo ufw allow 123/udp```

* Close port 22 using ```sudo ufw deny 22```

* Enable firewall using ```sudo ufw enable```

* Check the current firewall status using ```sudo ufw status```

* Update the firewall configuration in your Amazon Lightsail instance by going to the Networking tab.

* Delete default SSH port 22 and add ports with these settings:
      HTTP 80/TCP
      Custom 123/UDP
      Custom 2200/TCP 
      
* If you are using your own ssh client, open up a new terminal and try ssh in via the new port 2200 using ```ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.138.251.117 -p 2200```

* If you ssh in using Lightsail's browser-based ssh client, you can no longer use that. Open up a terminal and try ssh in via the new port 2200 using ```ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.138.251.117 -p 2200```

---

### Create user account grader

* Create a new user account grader using ```sudo adduser grader```

* Create a file using ```sudo touch /etc/sudoers.d/grader```

* Edit the file using ```sudo nano /etc/sudoers.d/grader```

* Add this ```grader ALL=(ALL:ALL) ALL``` to the file and save it

* Set SSH login for user grader

* Open a terminal in your local machine, use ssh-keygen to create an SSH key pair for grader and save it in the ~/.ssh path

* Insert the generated public key into the ubuntu server

* In your local machine's ~/.ssh folder, open and read the generated public key using ```cat ~/.ssh/FILE-NAME.pub```

* In your ubuntu server, create the .ssh folder inside the grader folder i.e. /home/grader, and within it create the authorized_keys file:

   ```
   $ mkdir .ssh
   $ touch .ssh/authorized_keys
   $ nano .ssh/authorized_keys
   ```
* Copy the generated public key and paste it into this authorized_keys file and save it

* In your terminal logged in to the ubuntu server, use ```chown -R grader.grader /home/grader/.ssh``` to change ownership and permissions of the .ssh folder to grader

* Run ```chmod 700 .ssh`` and ```chmod 644 .ssh/authorized_keys``` to change the permissions too

* Restart SSH using $ sudo service ssh restart

* Try to login in as user grader using ```ssh -i ~/.ssh/grader_key -p 2200 grader@18.138.251.117```

* You will be asked for grader's password. To disable it, open the configuration file using ```sudo nano /etc/ssh/sshd_config```

* Change PasswordAuthentication yes to no

* Restart SSH using ```sudo service ssh restart```

---

## Configure the local timezone to UTC

* Run ```sudo dpkg-reconfigure tzdata```

* Choose None of the above to set timezone to UTC

---

### Install and configure Apache

* Install Apache using ```sudo apt-get install apache2```

* In your browser go to http://18.138.251.117, you should see a Apache2 Ubuntu Default Page if Apache is configured correctly

* Install and configure the mod_wsgi file

* Install the mod_wsgi package using ```sudo apt-get install libapache2-mod-wsgi python-dev```

* Enable mod_wsgi using ```sudo a2enmod wsgi```

* Restart Apache using ```sudo service apache2 restart```

* To check if Python is installed, use ```python```. Exit with ```exit()```

---

## Install PostgreSQL

* Run this ```sudo apt-get install postgresql```

* Open this file using ```sudo nano /etc/postgresql/9.5/main/pg_hba.conf``

* Check to make sure the file content is as follow:

```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```
---

## Create a new PostgreSQL user catalog

* Change to PostgreSQL default user postgres using ```sudo su - postgres```

* Connect to PostgreSQL using ```psql``

* Create user catalog with the password as 'password' using ```# CREATE ROLE catalog WITH PASSWORD 'password';```

* Change user role to allow creation of database tables using ```# ALTER USER catalog CREATEDB;```

* Create database with owner using ```# CREATE DATABASE catalog WITH OWNER catalog;```

* Connect to the database catalog using ```# \c catalog```

* Revoke all the rights using ```# REVOKE ALL ON SCHEMA public FROM public;```

* Grant access to user catalog using ```# GRANT ALL ON SCHEMA public TO catalog;```

* Check that catalog has login access with ```\du```

* If not, run ```ALTER USER catalog LOGIN;```

* Exit from psql using ```\q```

* Exit from user postgres using ```exit```

---

## Create new Linux user catalog

* Create a new Linux user using ```sudo adduser catalog```

* Give user catalog sudo access using ```sudo visudo```

* Add ```catalog ALL=(ALL:ALL) ALL``` under line ```root ALL=(ALL:ALL) ALL```

* Save and exit the file

* Log in as user catalog using ```sudo su - catalog```

* Exit from user catalog using ```exit```

---

## Install git and clone the Item Catalog project

* Install git using ```sudo apt-get install git```

* Create directory using ```mkdir /var/www/catalog```

* Navigate to this directory using ```cd /var/www/catalog```

* Git clone the catalog project using ```sudo git clone RELEVENT-URL catalog```

* Change the ownership of the catalog folder using ```sudo chown -R ubuntu:ubuntu catalog/```

* Navigate to this directory using: ```/var/www/catalog/catalog```

* Change the file name from applicationsports_users.py to init.py using ```mv applicationsports_users.py __init__.py```

* Change this line app.run(host='0.0.0.0', port=8000) to app.run() in the init.py file

---

## Setup for deploying a Flask App on Ubuntu instance

* Install pip using $ sudo apt-get install python-pip

* Install the packages using
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
```
---

## Setup the .conf file

* Create this file using ```sudo touch /etc/apache2/sites-available/catalog.conf```

* Add the following to the file:
```
   <VirtualHost *:80>
		ServerName 13.250.107.163
		ServerAdmin test@admin.com
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
* Run ```sudo a2ensite catalog``` to enable the virtual host

* Restart Apache using ```sudo service apache2 reload```

---

## Setup the .wsgi file

* Create this file using ```sudo touch /var/www/catalog/catalog.wsgi```

* Add the following to the file:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/catalog/")

   from catalog import app as application
   application.secret_key = 'super_secret_key'
```   

* Restart Apache using ```sudo service apache2 reload```

---

## Update the database path in the app files

* Update the database line in __init__.py, database_setup_users.py, and lotsofsportsitems_users.py with 
```engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')```
---
## Last Steps

* Disable the default Apache page. Run ```sudo a2dissite 000-default.conf```

* Restart Apache using ```sudo service apache2 reload```

* Create the database file
```
sudo python database_setup_users.py
sudo python lotsofsportsitems_users.py
```
* Restart Apache using ```sudo service apache2 reload```

---

Accessing http://18.138.251.117 the application should be live. If there are internal errors, check the Apache error file e.g. run ```sudo tail -100 /var/log/apache2/error.log``` and resolve the traceback call errors it displays.
