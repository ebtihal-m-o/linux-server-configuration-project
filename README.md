#  udacity-linux-server-configuration
In this project I use Microsoft Azure and ubuntu server.
**Ubuntu Server** 14.04 LTS
##### Reference --> https://github.com/Hisham-Developer/linux-server-configuration#
##### Reference --> https://github.com/FahadAlsubaie/linux_server_configuration

## steps
 1. Create PRIVATE KEY and PUBLIC by run `KEY ssh-keygen` command and copy pub key   
 2. Create virtual machine at Microsoft Azure
 3. Login to the server
 4. Create a new user named **grader** by run `sudo adduser grader` and give a password for the user ***Password= 11223344***
 5. Give the grader sudo prevelagie by run `sudo vi /etc/sudoers.d/grader` 
 press **i** to insert and write: *grader ALL=(ALL:ALL) NOPASSWD:ALL*
press **esc** and then write :wq to save
6. Run `sudo vi /etc/ssh/sshd_config`  to change ssh port from 22 to 2200 and edit `PermitRootLogin without-password` **to** `PermitRootLogin no` and save 
7. Run `sudo service ssh restart` 
8. ssh to grader by run `su - grader` then `mkdir .ssh`
9.  Run `touch .ssh/authorized_keys` then `vi .ssh/authorized_keys` and paste your public key and save the file.
10. **Update and upgrade your system then you should download all the dependecnies:** 
    sudo apt-get install apache2 
    sudo apt-get install libapache2-mod-wsgi python-dev 
    sudo a2enmod wsgi 
    sudo apt-get install git 
    sudo apt-get install python-pip 
    sudo pip install virtualenv 
    sudo service apache2 start 
11. **Allow incoming connection by run** 
    sudo ufw allow 2200/tcp 
    sudo ufw allow 80/tcp 
    sudo ufw allow 123/udp 
    sudo ufw deny 22
    sudo ufw enable 
12. Change timezone to **UTC** by run `sudo dpkg-reconfigure tzdata` and press enter twice.
13.  Now you can enter the site using 2200 port  `ssh -p 2200 grader@104.45.6.0`
14. Run `cd /var/www` to clone your app 
15. Create catalog folder by run `sudo mkdir catalog` 
16. Change owner of the newly created catalog folder to **grader** `sudo chown -R grader:grader catalog`
17. `cd catalog/`
18. Clone your project from github `sudo git clone https://github.com/ebtihal-m-o/ItemCatalog.git catalog`
19. Create a **catalog.wsgi**  file, by run `sudo touch catalog.wsgi`  then `sudo vi catalog.wsgi` and insert  this content:
```
import sys 

import logging 

logging.basicConfig(stream=sys.stderr) 

sys.path.insert(0, "/var/www/catalog/") 

from catalog import app as application 

application.secret_key = 'super_secret_key' 
```
20. **Install virtual environment**
Create a new virtual environment with `sudo virtualenv venv` 
##### Activate it with

    source venv/bin/activate

Change permissions `sudo chmod -R 777 venv`

    pip install Flask
    
    pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
21. Create a new file by run this  `sudo vi /etc/apache2/sites-available/000-default.conf`  and edit the file:
```
<VirtualHost *:80>  

ServerName 104.45.6.0

ServerAlias item-catalog.eastus.cloudapp.azure.com 

ServerAdmin webmaster@104.45.6.0
DocumentRoot /var/www/html 

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

CustomLog ${APACHE_LOG_DIR}/access.log combined 

WSGIScriptAlias / /var/www/catalog/catalog.wsgi 
</VirtualHost> 
```
```
sudo a2ensite 000-default.conf 
```
22. Set up postgress run..
```
sudo apt-get install libpq-dev python-dev  

sudo apt-get install postgresql postgresql-contrib 

sudo su - postgres 
```
psql

CREATE USER catalog WITH PASSWORD 'password';

ALTER USER catalog CREATEDB;

CREATE DATABASE catalog WITH OWNER catalog;

\c catalog

REVOKE ALL ON SCHEMA public FROM public;

GRANT ALL ON SCHEMA public TO catalog;

\q

exit

23.  **install** pip install psycopg2-binary ***To run:***
python /var/www/catalog/catalog/DB.py
24. restart apache2 and then visit the link: `sudo service apache2 restart`
25.  Visit site at http://www.104.45.6.0.xip.io/
