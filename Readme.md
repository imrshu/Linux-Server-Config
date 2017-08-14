# Linux Server Configurtaion

This project is a part of FSND (Full Stack Web Developer) by Udacity.

This project consists of a baseline installation of a Linux server and then prepare it to host my web applications. This server is secure from a number of attack vectors, server also configure a database server, and hosts one of my web application as well which is globally accessible now to the world.

## Useful information

Ip Address:- 34.211.175.198
SSH Port:- 2200

## Run the project (live View of project)

- In order to see this project directly in a browser just click this ip [http://34.211.175.198](http://34.211.175.198)


### Steps To setup or config the Server

# 1. SSH into linux server
- login into VM as root using:- `ssh -i ~/.ssh/udacity_key.rsa root@34.211.175.198`

# 2. Create a new user name it grader and grant it sudo permission.
- `sudo adduser grader`
- `sudo nano /etc/sudoers.d/grader`
- Add the below text in the newly created grader file.
- `grader ALL=(ALL:ALL) ALL`

# 3. Update All currently installed pacakages
- `sudo apt-get update`
- `sudo apt-get upgrade`

# 4. Configure local timezone to UTC
- `sudo dpkg-reconfigure tzdata`
- Also Intsall **NTP** for better synchronization of the server's time using below command:-
- `sudo apt-get install ntp`

Source:- [Ubuntu Help](https://help.ubuntu.com/community/UbuntuTime)

# 5. Configure Key based Authentication for user named grader
- Generate an **ssh-keypair** on local system Using `ssh-keygen` [I name it **graderkey**]
- Now login the remote Vm as root and create the following file : `touch /home/grader/.ssh/authorized_keys`
- Copy Content of the **graderkey.pub** file and paste it into **authorized_keys** file using below two commands-
- `sudo cat graderkey.pub` & copy all of the content.
- `sudo nano .ssh/authorized_keys` & paste the copied content here and save the file.
- Then Change File permissions to restrict access to the files using below commands-
- `sudo chmod 700 /home/grader/.ssh`
- `sudo chmod 644 /home/grader/.ssh/authorized_keys`
- Finally now change the Owner & Group of .ssh directory using `sudo chown -R grader:grader /home/grader/.ssh`
- Now you are able to log into the remote VM through ssh with the following command: $ `ssh -i ~/.ssh/graderkey grader@34.211.175.198`

# 6. Enforcing key-based ssh authentication
- Run `sudo nano /etc/ssh/sshd_config` & find **PasswordAuthentication line** and set it to **no**
- Save the file & restart the ssh service
- `sudo service ssh restart`

# 7. Change SSH port from 22 to 2200
- Run `sudo nano /etc/ssh/sshd_config` & find **Port line** and set the port no to **2200**
- Run `sudo service ssh restart`
-  Run `ssh -i ~/.ssh/graderkey -p 2200 grader@34.211.175.198` to log into the vm as grader user on non default ssh port no.

# 8. Disable ssh login for root user
- Run `sudo nano /etc/ssh/sshd_config` & find the **PermitRootLogin** line and set it to **no**.
- `sudo service ssh restart`

Source:- [AskUbuntu](https://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server)

# 9. Configure the UFW (Ubuntu firewall)
- Run `sudo ufw allow 2200/tcp`
- Run `sudo ufw allow 80/tcp`
- Run `sudo ufw allow 123/udp`
- Run `sudo ufw enable`

# 10. cron scripts configurationn to automatically manage package source list updates
- Run `sudo apt-get install unattended-upgrades` to install unattended-upgrades.
- Run `sudo dpkg-reconfigure --priority=low unattended-upgrades` to enable it.

# 11. Install Git (version control system)
- Run `sudo apt-get install git`.
- Configure your username using : `git config --global user.name <username>`.
- Configure your email using : `git config --global user.email <email>`.

# 12. Install and configure Apache to serve a Python mod_wsgi App
- Run `sudo apt-get install apache2` to install apache
- Run `sudo apt-get install python-setuptools libapache2-mod-wsgi` to Install mod_wsgi.
- Run `sudo a2enmod wsgi` to Enable mod_wsgi.
- Run `sudo service apache2 restart` to restart apache

# 13. Install and configure PostgreSQL (database server)
- Run `sudo apt-get install postgresql` to install postgresql.
- Run `sudo su - postgres` to login as postgres.
- Run `psql` to get into the psql shell.
- Create a new database named catalog using `CREATE DATABASE catalog;`
- Now create a new user named catalog in psql shell using `CREATE USER catalog;`
- Now Set a password for catalog user using `ALTER ROLE catalog WITH PASSWORD 'password';`
- Give "catalog" user permission to "catalog" app database using `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
- Now quit PostgreSQL using `\q`
- Now type **exit** to exit from postgres.

# 14. Clone and setup Catalog App project
- `cd /var/www` , then make directory named catalog using `sudo mkdir FlaskApp.
- Change owner for the FlaskApp folder using- `sudo chown -R grader:grader FlaskApp`.
- Now Move inside that newly created folder using `cd /FlaskApp`
- Now clone the github repo using `git clone https://github.com/imrshu/Toy-item-catalog.git FlaskApp`.
- Now move to the Cloned repo folder which we named as **FlaskApp* using- `cd FlaskApp`.
- Now rename `app.py` file to `__init__.py` using- `sudo mv app.py __init__.py`.
- Now Open up and Edit these three files `database_setup.py`, `__init__.py` and `functions_helper.py` first of all find for the line **engine = create_engine('sqlite:///toyshop.db')** and then change it to **engine = create_engine('postgresql://catalog:password@localhost/catalog')** in all of the above 3 files and save them.
- Run `Install pip sudo apt-get install python-pip` to install python pip installer.
- Run `sudo apt-get -qqy install postgresql python-psycopg2` to install psycopg2 python module.
- Run `sudo python database_setup.py` to Create database schema.

# 15. Configure and Enable a New Virtual Host
- Run `sudo nano /etc/apache2/sites-available/FlaskApp.conf` to Create FlaskApp.conf file.
- Now Add the following lines of code to the file to configure the virtual host
```
<VirtualHost *:80>
	ServerName 34.211.175.198
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
- Run `sudo a2ensite FlaskApp` to enable virtual host.

Source- [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

# 16. Create flaskapp.wsgi
1. `cd /var/www/FlaskApp`
2. `sudo nano flaskapp.wsgi`
3. Add the following lines of code to the flaskapp.wsgi file
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```

# 17. Finally Restarat the apache
- Run `sudo service apache2 restart`.

**Note 1 :-** Inside the Flask application, the database connection is now performed with
> engine = create_engine('postgresql://catalog:password@localhost/catalog')

**Note 2 :-** If you clone my github repo then you would not need to rename any file as i have already done that.
