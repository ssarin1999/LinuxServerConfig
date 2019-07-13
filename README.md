# Linux Server Configuration Project 

This project revolved around taking a baseline installation of a Linux server and preparing it to host a web application. The server had to be secured from a number of attack vectors, installed and configured with a database server, and deploy one of my existing web applications onto it.
IP Address: 35.182.250.231
Port: 2200
## Software Installed
 Apache2<br>
 Python mod_wsgi<br>
 PostgreSQL<br>
 Git<br>
 Python-PIP<br>


## Instructions/Configurations

### Start a new Ubuntu Linux Server instance on Amazon Lightsail
1. Create an AWS account
2. Click **Create instance** button on the home page
3. Select **Linux/Unix** platform
4. Select **OS Only** and **Ubuntu** as blueprint
5. Select an instance plan
6. Name your instance
7. Click **Create** button

### SSH into your Server
1. Download private key from the **SSH keys** section in the **Account** section on Amazon Lightsail.
2. Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your **local machine**
3. Copy and paste content from downloaded file to **lightsail_key.rsa**
4. Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`
5. SSH into the instance using the key: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.218.99.181`

### Update All Currently Installed Packages
1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install newest versions of packages

###  Change The SSH Port From 22 To 2200
1. Run `$ sudo nano /etc/ssh/sshd_config` to open up the configuration file
2. Change the port number from **22** to **2200** in this file
3. Save and exit the file
4. Restart SSH: `$ sudo service ssh restart`

### Configure The Firewall
1. Check firewall status: `$ sudo ufw status`
2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`
3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow 123/udp`
7. Close port 22: `$ sudo ufw deny 22`
8. Enable firewall: `$ sudo ufw enable`
9. Check current firewall status: `$ sudo ufw status`
10. Update the firewall configuration on Amazon Lightsail website under **Networking**. Delete default SSH port 22 and add **port 80, 123, 2200**
11. Open up a new terminal and you can now ssh in via the new port 2200: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.218.99.181 -p 2200`

### Create A New User Account **grader** And Give New User Account Sudo Access
1. Create a new user account **grader**:`$ sudo adduser grader`
2. Edit this file: `$ sudo nano /etc/sudoers`, add code `grader ALL=(ALL:ALL) ALL`. Save and exit

### Set SSH Login Using Keys
1. Create an SSH key pair for **grader** using the `ssh-keygen` tool on your **local machine**. Save it in `~/.ssh` path
2. Deploy public key on development environment
    * On your local machine, read the generated public key
     `cat ~/.ssh/FILE-NAME.pub`
    * On your virtual machine
   ```$ su -grader
      $ mkdir .ssh
      $ touch .ssh/authorized_keys
      $ nano .ssh/authorized_keys
      ```
    * Copy the public key to this _authorized_keys_ file on the virtual machine and save
3. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` on your virtual machine to change file permission
4. Restart SSH: `$ sudo service ssh restart`
5. Now can now login in as grader: `$ ssh -i ~/.ssh/grader_key -p 2200 grader@18.218.99.181`
6. You will be asked for grader's password. To unable it, open configuration file again: `$ sudo nano /etc/ssh/sshd_config`
7. Change `PasswordAuthentication yes` to **no**
8. Restart SSH: `$ sudo service ssh restart`

### Change The Local Timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Choose **None of the above** to set timezone to UTC

### Install and Configure Apache
1. Install **Apache**: `$ sudo apt-get install apache2`
2. Go to http://35.182.250.231/, if Apache is working correctly, the **Apache2 Ubuntu Default Page** will show up

### Install and Configure Python mod_wsgi
1. Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
3. Restart **Apache**: `$ sudo service apache2 restart`
4. Check if Python is installed: `$ python`

### Install PostgreSQL
1. Run `$ sudo apt-get install postgresql`
2. Make sure PostgreSQL does not allow remote connections
3. Open file: `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
4. Check to make sure it looks like this:
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
### Create New PostgreSQL User **catalog**
1. Switch to PostgreSQL default user **postgres**: `$ sudo su - postgres`
2. Connect to PostgreSQL: `$ psql`
3. Create user **catalog** with LOGIN role: `# CREATE ROLE catalog WITH PASSWORD 'password';`
4. Allow user to create database tables: `# ALTER USER catalog CREATEDB;`
5. Create database: `# CREATE DATABASE catalog WITH OWNER catalog;`
6. Connect to database **catalog**: `# \c catalog`
7. Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`
8. Grant access to **catalog**: `# GRANT ALL ON SCHEMA public TO catalog;`
9. Exit psql: `\q`
10.Exit user **postgres**: `exit`

### Create New Linux User **catalog** and New Database
1. Create a new Linux user: `$ sudo adduser catalog`
2. Give **catalog** user sudo access:
   * `$ sudo visudo`
   * Add `$ catalog ALL=(ALL:ALL) ALL` under line `$ root ALL=(ALL:ALL) ALL`
   * Save and exit the file
3. Log in as **catalog**: `$ sudo su - catalog`
4. Create database **catalog**: `createdb catalog`
5. Exit user **catalog**: `exit`

### Install Git and Clone Restaurant Application from GitHub
1. Run `$ sudo apt-get install git`
2. Create dictionary: `$ mkdir /var/www/catalog`
3. CD to this directory: `$ cd /var/www/catalog`
4. Clone the catalog app: `$ sudo git clone RELEVENT-URL catalog`
5. Change the ownership: `$ sudo chown -R ubuntu:ubuntu catalog/`
6. CD to `/var/www/catalog/catalog`
7. Change file **application.py** to **__init__.py**: `$ mv application.py __init__.py`
8. Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in **__init__.py** file

### Edit client_secrets.json file
1. Create a new project on Google API Console and download `client_scretes.json` file
2. Copy and paste contents of downloaded `client_scretes.json` to the file with same name under directory `/var/www/catalog/catalog/client_secrets.json`

### Setup To Deploy Flask App
1. Install pip: `$ sudo apt-get install python-pip`
2. Install packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
   ```

### Setup and Enable a Virtual Host
1. Create file: `$ sudo touch /etc/apache2/sites-available/catalog.conf`
2. Add the following to the file:
```
   <VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin admin@xx.xx.xx.xx
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
3. Run `$ sudo a2ensite catalog` to enable the virtual host
4. Restart **Apache**: `$ sudo service apache2 reload`

### Configure .wsgi File
1. Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`
2. Add content below to this file and save:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/catalog")

   from catalog import app as application
   application.secret_key = 'super_secret_key'
```
3. Restart **Apache**: `$ sudo service apache2 reload`

### Edit the Database Path in Project Files
1. Replace lines in `__init__.py`, `database_setup.py`, and `lotsofitems.py` with `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')`

### Disable Defualt Apache Page
1. `$ sudo a2dissite 000-defualt.conf`
2. Restart **Apache**: `$ sudo service apache2 reload`

### Set Up Database Schema
1. Run `$ sudo python database_setup.py`
2. Run `$ sudo python lotsofitems.py`
3. Restart **Apache**: `$ sudo service apache2 reload`
4. Now follow the link to http://35.182.250.231, the application should be runing online
5. If internal errors occur: check the [Apache error file](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)

##Third Party Resources
1. Amazon Lightsail
2. Google API Console
3. Apache

