# Catalog App #

By Vinicius Conceição.

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

# IP Address and SSH PORT
- Public ip: 3.93.89.197
- SSH PORT : 2200

# Web Application URL
- http://3.93.89.197.xip.io/catalog

# Softwares Installed

- Apache 2
- PostgreSQL 
- finger — user information lookup program
- Git
- libapache2-mod-wsgi
- python-pip

# Summary of Configurations

Update all currently installed packages  
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo shutdown -r now
```
Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.

`sudo vi /etc/ssh/sshd_config`  
Search for the line:  
#Port 22  
Remove # and change 22 for 2200.  
Restart the service:  
`service sshd restart`  

Configure the Linux Firewall UFW to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).  
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/tcp
```
## Access and SSH keys
Create a new user account named grader.  
`sudo useradd grader`  
Create a file name grader in the etc/sudoers.d  
`sudo touch /etc/sudooers.d/grader`  
Edit the file created and insert the following info:  
`grader ALL=(ALL) NOPASSWD:ALL`  

Create a new ssh key-pair for the server in https://lightsail.aws.amazon.com/ls/webapp/account/keys named grader-keypair  
Download the key file to your machine 'grader_keypair.pem'  
Use Puttygen to generate the public and the private key.  
Save the private key in a safe place.  

Logged at the server   
- Create a new file and name it as authorized_keys.   
	`touch /home/grader/.ssh/authorized_keys`  
- Copy the content of the generated public key in your local machine  
- Paste the the content of the generated public key at authorized_keys file in the server  
`nano home/grader/.ssh/authorized_keys`  

At your machine, you can PUTTY or another tool to log in the with private key that was generated from the PUTTYgen  

## Web Server and DB Configuration
Configure localtimezone to UTC  
`sudo dpkg-reconfigure tzdata`  

Install and configure Apache to serve a Python mod_wsgi application.  
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```
 
Install and configure PostgreSQL:  
`sudo apt-get install postgresql`  
After installing PostgreSQL database server, remote access mode is disabled by default for security reasons.   
Double checked configurations to not allow remote connections in the database  
`sudo nano /etc/postgresql/9.1/main/pg_hba.conf`  
Create a new database user named catalog that has limited permissions to your catalog application database.  

postgres=# `CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';`  
postgres=# `ALTER ROLE catalog CREATEDB;`  

Database 'catalogdb' created  

  ```
 List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges
-----------+----------+----------+---------+---------+-----------------------
 catalogdb | catalog  | UTF8     | C.UTF-8 | C.UTF-8 |
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(4 rows)

 ```
 
 Clone and setup the Item Catalog project from the GitHub repository 
- Install Git
- Create `/var/www/catalog/` directory.
- Change to that directory and clone the catalog project:
`sudo git clone https://github.com/vconceicao/catalog_app.git.`
- From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
- Change to the `/var/www/catalog/catalog_app` directory.
- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`. 

Create a new  OAuth Client ID for the server in [Google Cloud Plateform](https://console.cloud.google.com/).

Install the project dependencies
```
sudo pip install httplib2
sudo pip install requests
sudo pip install --upgrade oauth2client
sudo pip install sqlalchemy
sudo pip install flask
sudo pip install psycopg2
```

Change the string connection to use postgres db in `/var/www/catalog/catalog_app/ database_setup.py`
```
engine = create_engine('postgresql://catalog:123@localhost/catalogdb')
```

Create tables for catalogdb
`python /var/www/catalog/catalog_app/database_setup.py`

- Create `/var/www/catalog/catalog.wsgi` file add the following lines:

```
#!/usr/bin/python
import sys
sys.path.insert(0, "/var/www/catalog/catalog_app/")
sys.path.insert(1, "/var/www/catalog/")
from catalog_app import app as application
 ```
 

Edit `/etc/apache2/sites-enabled/000-default.conf` and add the 
following lines to configure the virtual host:

  ```
  <VirtualHost *:80>
	 	  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	  <Directory /var/www/catalog/catalog_app/>
	  	Order allow,deny
		  Allow from all
	  </Directory>
	  Alias /static /var/www/catalog/catalog_app/static
	  <Directory /var/www/catalog/catalog_app/static/>
		  Order allow,deny
		  Allow from all
	  </Directory>
	   </VirtualHost>
  ```
Restart Apache
`sudo apache2ctl restart`

Access the link in your browser
http://3.93.89.197.xip.io/catalog

# Third Party Resources

- https://www.a2hosting.com/kb/developer-corner/postgresql/managing-postgresql-databases-and-users-from-the-command-line
- http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
- https://docs.sqlalchemy.org/en/latest/core/engines.html#postgresql
- https://stackoverflow.com/questions/5682809/django-static-file-hosting-an-apache
- https://github.com/bencam/linux-server-configuration
- https://www.digitalocean.com/community/tutorials/how-to-create-ssh-keys-with-putty-to-connect-to-a-vps
