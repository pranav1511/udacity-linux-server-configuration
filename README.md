# Linux Server Configuration
A simple python based web application deployed on a Linux Server along with the required security configuration and database setup.


## Details
* Public Static IP: `13.232.45.254`
* SSH Port: `2200`
* Website URI: http://udacityitemcatalog.tk/


## Configuration Steps

#### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail.

#### 2. Follow the instructions provided to SSH into the server.

#### 3. Update all currently installed packages.
```
sudo apt-get update
sudo apt-get upgrade
sudo apt full-upgrade
```
#### 4. Change the SSH port from 22 to 2200 and configure the Lightsail firewall to allow it.
* `sudo nano /etc/ssh/sshd_config`
* Change the port from 22 to 2200
* `sudo service ssh restart`
* Restart the server

#### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
sudo ufw enable
```
#### 6. Create a new user account named grader.
`sudo adduser grader`

#### 7. Give grader the permission to sudo.
* `sudo nano /etc/sudoers.d/grader`
* Add `grader ALL=(ALL:ALL) ALL` and save it

#### 8. Create an SSH key pair for grader using the ssh-keygen tool.
* On the local computer: `ssh-keygen`
* On the server:
```
mkdir .ssh
sudo nano .ssh/authorized_keys
```
* Copy the public key here
```
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
```
* Now we can login to grader user through: `ssh -i "KEY_PAIR" grader@13.232.45.254 -p 2200`

#### 9. Configure the local timezone to UTC.
* `sudo dpkg-reconfigure tzdata`
* Select `None of the above` and then `UTC`

#### 10. Install and configure Apache to serve a Python mod_wsgi application.
```
sudo apt-get install apache2 libapache2-mod-wsgi
sudo a2enmod wsgi
sudo service apache2 start
```

#### 11. Install and configure PostgreSQL
* Install PostgreSQL, login and connect to PostgreSQL
```
sudo apt-get install postgresql
sudo su - postgres
psql
```
* Create new user: catalog `CREATE USER catalog WITH PASSWORD '******';`
* Change catalog permission to create database `ALTER USER catalog CREATEDB;`
* Create a new database `CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to database `\c catalog`
* Revoke all permissions from other users and allow only catalog user
```
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
```
* Disconnect from PostgreSQL and logout
```
\q
exit
```

#### 12. Install git
`sudo apt-get install git`

#### 13. Clone and setup the Item Catalog project from the Github repository in the server
* Install virualenv and the other required dependencies into the virtual environment
```
sudo apt-get install virtualvenv
cd /var/www
sudo mkdir itemcatalog
cd itemcatalog
sudo git clone https://github.com/pranav1511/udacity-linux-server-configuration.git catalog
cd catalog
sudo git checkout deployment
sudo virtualenv venv
source venv/bin/activate 
sudo pip install -r requirements.txt
deactivate
```
* Add wsgi file
```
cd /var/www/itemcatalog
sudo nano catalog.wsgi
```
Add this to the file
```
#!/usr/bin/python 
import sys 
import logging 

logging.basicConfig(stream=sys.stderr) 
sys.path.insert(0,"/var/www/itemcatalog/")  

from catalog import app as application 
application.secret_key = 'super_secret_key'
```
* Configure Apache to add the site
```
cd /etc/apache2/sites-available/
sudo nano catalog.conf
```
Add this to the file
```
<VirtualHost *:80>
        ServerName mywebsite.com
        ServerAdmin admin@mywebsite.com
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
        <Directorymatch "^/.*/\.git/">
            Order deny,allow
            Deny from all
        </Directorymatch>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Disable default site and activate catalog
```
sudo a2dissite 000-default.conf
sudo a2dissite catalog.conf
sudo service apache2 restart
```
#### 14. Configure OAuth
* Open Google and Facebook developers page and update the links to this website

## Resources
* [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps/)
* [mod_wsgi](https://modwsgi.readthedocs.io/en/develop/)
* [virtualenv](https://virtualenv.pypa.io/en/stable/)