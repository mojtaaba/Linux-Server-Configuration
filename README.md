# Linux Server Configuration

a baseline installation of a Linux server and to prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

- Public IP: 3.8.172.40

- SSH port: 2200

- Application: http://3.8.172.40.xip.io



## 1. Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## 2. Create a new user named grader
1. login into your server as ubuntu user then write the following
1. `$ sudo adduser grader`
4. `$ sudo nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## 3. Setup ssh login for grader
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on your local machine.

2. now create file to hold the public key on your server
	```
	$ su - grader
	$ mkdir .ssh
	$ sudo nano .ssh/authorized_keys
	```
3. Copy the public key generated on your local machine into this file and save it then Change the accessing permission:
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```

4. Restart SSH  `$ service ssh restart`
5. now you can  use ssh to login with the new user (grader)

	`ssh -i "Private-key-path" grader@3.8.172.40`

## 4. Change the SSH port from 22 to 2200 and accessing method
1. Use `sudo vim /etc/ssh/sshd_config`
2. change Port 22 to Port 2200
3. Change PermitRootLogin to no
4. Change PasswordAuthentication to no
5. save & quit. then Restart ssh with `sudo service ssh restart`

## 5. Configure the Uncomplicated Firewall
```
  $ sudo ufw default deny incoming.
  $ sudo ufw default allow outgoing.
  $ sudo ufw allow 2200/tcp.
  $ sudo ufw allow 80/tcp.
  $ sudo ufw allow 123/udp.
  $ sudo ufw enable
 ```
## 6. Configure the local timezone to UTC
1. Install unattended-upgrades: $ sudo apt-get install unattended-upgrades.
2. Enable it by: $ sudo dpkg-reconfigure --priority=low unattended-upgrades.

## 7. Install and configure Apache to serve a Python mod_wsgi application and Git
1. `$ sudo apt-get install apache2.`
2. `Install mod_wsgi with the following command: $ sudo apt-get install libapache2-mod-wsgi python-dev.`
3. `Enable mod_wsgi: $ sudo a2enmod wsgi.`
4. `$ sudo service apache2 start.`
5. `$ sudo apt-get install git.`

## 8. Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
5. Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
6. Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres"

	```
	exit
	```

## 9. Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/mojtaaba/Item_Catalog_linux.git`
6. Rename the project's name `sudo mv ./Item_Catalog_linux ./FlaskApp`
7. Move to the inner FlaskApp directory using `cd FlaskApp`
8. Rename `application.py` to `__init__.py` using `sudo mv application.py __init__.py`
9. Edit `database_setup.py`, `__init__.py` and `functions_helper.py` and change `engine = create_engine('sqlite:///Catalog.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
10. Edit the client_Secret.json path and add the whole path
11. Install pip `sudo apt-get install python-pip`
12. Use pip to install dependencies `sudo pip install -r requirements.txt`
13. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
14. Create database schema `sudo python database_setup.py`

## 10. Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host.

	```
	<VirtualHost *:80>
		ServerName 3.8.172.40
		ServerAdmin youemail@gmail.com
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
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## 11. Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp:

	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi
	```
2. Add the following lines of code to the flaskapp.wsgi file:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```
## 12. Authenticate login through Google

1. Go to Google Cloud Plateform.
2. Click APIs & services on left menu.
3. Click Credentials.
4. Create an OAuth Client ID , and add following as authorized JavaScript origins.
    - http://localhost:5000
    - http://3.8.172.40.xip.io
5. Add 	the following as authorized redirect URI.
    - http://3.8.172.40.xip.io/login
    - http://3.8.172.40.xip.io/gconnect
    - http://localhost:5000/login
    - http://localhost:5000/gconnect
6. Download the corresponding JSON file, open it and copy the contents.
7. Open /var/www/catalog/catalog/client_secret.json and paste the previous contents into the this file.
8. Replace the client ID in templates/login.html file in the project directory.


## congratulations
1. now just Restart Apache `sudo service apache2 restart ` and you can use the Server via http://3.8.172.40.xip.io
