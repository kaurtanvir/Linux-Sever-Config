# Linux Server Configuration
This project takes a baseline installation of a Linux server and then prepare it to host web applications. 
## Features
1. IP address:- 34.223.230.88
2. SSH port:- 2200
3. Click on the link http://ec2-34-223-230-88.us-west-2.compute.amazonaws.com/ to view project in the browser.

## Steps to configure the Server
 1. SSH into linux server.
 2. Update all currently installed packages:
       - sudo apt-get update
 3. Change SSH port from 22 to 2200
    - sudo nano /etc/ssh/sshd_config & find the line for port  and set the port no to 2200
    - Restart the service:  sudo service ssh restart
 4. Configure UFW 
       - sudo ufw allow 2200/tcp
       - sudo ufw allow 80/tcp
       - sudo ufw allow 123/udp
       - sudo ufw enable
 5. Create a new user grader and grant it sudo permission.
       - sudo adduser grader
 6. Create Key based Authentication for user grader:
     - Generate ssh-keypair on local machine using ssh-keygen. I saved it under default .ssh directory as /.ssh/linuxCourse
     - Make a directory .ssh on the server by logging in as grader : mkdir .ssh in the /home/grader directory.
     - Create the following file : touch /home/grader/.ssh/authorized_keys
     - Copy content of the linuxCourse.pub file from the local machine and paste it into authorized_keys file on the server 
     - Change file permissions to restrict access to the files using:
          sudo chmod 700 /home/grader/.ssh
         sudo chmod 644 /home/grader/.ssh/authorized_keys
      - Login through ssh with the following command: $ ssh -v grader@34.223.230.88 -p 2200 -i /c/Users/tanvir/.ssh/linuxCourse
7. Enforcing key-based ssh authentication
    - Edit the sshd_config file using: sudo nano /etc/ssh/sshd_config 
    - Set PasswordAuthentication line to no
    - Save the file & restart the ssh service: sudo service ssh restart
8. Configure local timezone to UTC
   - sudo dpkg-reconfigure tzdata
   - sudo apt-get install ntp
9. Disable ssh login for root user
   - sudo nano /etc/ssh/sshd_config & find the PermitRootLogin line and set it to no.
   - sudo service ssh restart
10. Install Git (version control system)
    - sudo apt-get install git.
11. Install and configure Apache to serve a Python mod_wsgi application
    - sudo apt-get install apache2 to install apache
    - sudo apt-get install libapache2-mod-wsgi to install mod_wsgi.
    - sudo service apache2 restart to restart apache
12. Install and configure PostgreSQL (database server)
      - sudo apt-get install postgresql to install postgresql.
      - sudo -i -u postgres to login as postgres.
      - psql to get into the psql shell.
13. Create a new database named catalog using CREATE DATABASE catalog.
14. Create a new user named catalog using CREATE USER catalog.
15. Clone and setup Item Catalog project
     - cd /var/www , then make directory using: sudo mkdir FlaskApp.
     - Clone the github repo using git clone https://github.com/tkaur157/Item-Catalog.git FlaskApp inside the created folder FlaskApp
     - Rename app.py file to ```__init__.py``` using: sudo mv app.py ```__init__.py```.
     - Edit the database information in the files database_setup.py, ```__init__.py``` and database_populator.py to change the db configuration url from sqllite to postgres using engine=create_engine('postgresql://catalog:password@localhost/catalog')
     - sudo apt-get -qqy install postgresql python-psycopg2 to install psycopg2 python module.
16. Configure and enable a new virtual host. 
    - sudo nano /etc/apache2/sites-available/FlaskApp.conf to create FlaskApp.conf file.
    - Add the following lines of code to the file to configure the virtual host
```<VirtualHost *:80>
        ServerName 34.223.230.88
        ServerAdmin admin@gmail.com
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
                Order allow,deny
                Allow from all
         </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
18. Create flaskapp.wsgi
    - cd /var/www/FlaskApp
    - sudo nano flaskapp.wsgi
    - add the following lines of code to the flaskapp.wsgi file
```#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")
from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```
19. Restart apache
     - sudo service apache2 restart.
