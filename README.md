# IP Address - 52.66.88.151
# URL: https://52.66.88.151

# Software
1. Python2.7
2. Apache2
3. libapache2-mod-wsgi
4. Postgresql
5. Flask
6. Git
7. Sqlalchemy
8. psycopg2
9. httplib2


User Management:

#### User grader created.

- sudo adduser graderc

#### User grader granted sudo access.

- Create a grader file in ``/etc/sudoers.d`` with the following contents

	``grader ALL=(ALL) NOPASSWD:ALL``

#### SSH Configuration - denied root login

- Navigate to /etc/ssh/sshd_config and set ``PermitRootLogin`` to ``no``

Security

#### SSH Configuration - default port changed to 2200
- Navigate to /etc/ssh/sshd_config and change the Port to 2200

#### SSH Configuration - enabled key-based authentication
- Navigate to /etc/ssh/sshd_config and set ``PubkeyAuthentication`` to ``yes``

- Restart the SSH server sudo service ssh restart

#### SSH Public key set up for User grader. using ssh-rsa

1. Create a ssh key for grader using ``ssh-keygen -t rsa`` on your local system.
2. Provide the directory where the public and private key can be stored. ``~/.ssh/<<>>``
3. Naviage to the lightsail instance and create the .ssh directory in /home/grader.

	``mkdir .ssh``
4. Create a file called ``authorized_keys`` in the .ssh directory.

5. Copy the public key generate on the local machine in step 1 into ``authorized_keys`` in the lightsail machine under /home/grader/.ssh.

- Restart the SSH server sudo service ssh restart

#### UFW firewall setup to allow http, https, ssh and ntp
Login into the Lightsail box using the public key for grader and allow the following ports using UFW.

``ufw allow Apache Full``
``ufw allow 2200/tcp``
``ufw allow ntp``

Additional Info: Item_catalog app uses FB for 3rd party authorization. FB does not allow plain text authentication(http) and only allows https connections for authentication. Hence had to allow port 443 to allow proper functioning of the application.

#### All applications are up-to date.
- Run ``sudo apt-get update`` to update all the packgaes before additional packages can be installed.


#### Application functionality:
Web server redirects all port 80 traffic to port 443. This was done to support FB authentication. 

Note to reviewer: FB does not allow authentication over HTTP (80) and HTTPs(443) is mandatory 

#### Database has been configured using postgresql.


## Software installed: 

#### Apache2 - ``sudo apt-get install apache2``

#### Apache2-mod-wsgi - ``sudo apt-get install libapache2-mod-wsgi ``

#### Postgresql - ``sudo apt-get install postgresql``

#### Python2.7 - ``sudo apt-get install python``

#### Python-pip - ``sudo apt-get install python-pip``

#### Git - ``sudo apt-get install git``

##### Python package requirements for application

Note: Before running pip run ``EXPORT LC_ALL=C``

- Psycopg2 - ``sudo pip install psycopg2``
- sqlachemy - ``sudo pip install sqlachemy``
- flask - ``sudo pip install flask``
- httplib2 - ``sudo pip install httplib2``

## Apache Configuration and setup: 

1. Create the following file udacity_item_catalog.conf  in /etc/apache2/sites-available. This file enables the wsgi and redirect to https.

	``<VirtualHost *:80>
		ServerName 52.66.88.151
		ServerAdmin admin@52.66.88.151.com
		WSGIScriptAlias / /var/www/html/udacity_item_catalog/flaskapp.wsgi
		#WSGIDaemonProcess udacity_item_catalog python-home=/usr/local python-path=/var/www/html/udacity_item_catalog
		#WSGIProcessGroup udacity_item_catalog
		<Directory /var/www/html/udacity_item_catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/html/udacity_item_catalog/static
		<Directory /var/www/html/udacity_item_catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
		Redirect "/" "https://52.66.88.151/
	</VirtualHost>``

	Note to reviewers: Visting port 80 redirects to port 443(https)

#### Enabling SSL to support FB Authentication.

1. Create the following SSL config file:

``<IfModule mod_ssl.c>
	<VirtualHost _default_:443>
		ServerAdmin webmaster@localhost
		ServerName 52.66.88.151
		DocumentRoot /var/www/html/udacity_item_catalog
		WSGIScriptAlias / /var/www/html/udacity_item_catalog/flaskapp.wsgi
		<Directory /var/www/html/udacity_item_catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/html/udacity_item_catalog/static
		<Directory /var/www/html/udacity_item_catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined
		SSLEngine on
		SSLCertificateFile	/etc/ssl/certs/apache-selfsigned.crt
		SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
		<FilesMatch "\.(cgi|shtml|phtml|php)$">
				SSLOptions +StdEnvVars
		</FilesMatch>
		<Directory /usr/lib/cgi-bin>
				SSLOptions +StdEnvVars
		</Directory>
	</VirtualHost>
	</IfModule>``

##### Apache Config test

* Run the following ``sudo apache2ctl configtest`` to check if there are any syntax errors in the apache config.

##### Loading the new config and restarting apache2.
1. Enable the Apache conf file using

	``sudo a2ensite udacity_item_catalog.conf ``

2. Restart the apache server using:
	``sudo service apache2 restart``



#### PostgreSQL Configuration
Post installation of Postgresql launch into postgresql terminal and execute the following commands.

1. Creating a user to be used by the application. 
	create user vagrant with encrypted password 'XXXXXXXXX';

2. Creating a database catalog to be used by the item_catalog application.
	postgres=# create database catalog;

3. Granting user vagrant access to catalog database.
	postgres=# grant all privileges on database catalog to vagrant;


#### Setting up the application.
1. Download the application from the private repo git clone https://github.com/satish28/udacity_item_catalog.git to download the application. 


Note to reviewer: Currently the repo is set up as private and it will not be available for public access. This is the same application which was submitted for project 4 (Item catalog) review. Minor changes to the configuration as described in the Application Code changes below.

2. Run ``python models.py`` to setup the database.

##### Application code changes

1. Change the database from sqlite to postgresql. The following change was made to the create_engine function. 

	``engine = create_engine('postgresql+psycopg2://vagrant:XXXXX@localhost/catalog')``
	Notes: A database catalog was created in postgresql. User(Vagrant) with password XXXX was created and granted privileges to the catalog database.

2. Changes to App. 
	``if __name__ == "__main__":
    app.run()	``

3. Creating a flaskapp.wsgi file inorder for apache to render the application.

	``#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/html/udacity_item_catalog")
	from views import app as application
	application.secret_key = 'xxxxxxxxxxxxxxxxx'``

4. Update the path to fb_secrets.json in the app. to /var/www/html/udacity_item_catalog/fb_secrets.json

##### Deploying the database.

1. run ``python models.py`` to set up the database for the application.

###References: 
1. Deploying a flask app in apache using mod_wsgi: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

2. Creating a user, database and granting access in postgresql : https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e

3. deploy a self-signed ceritifcate for apache.
https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04