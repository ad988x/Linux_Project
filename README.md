IP: 18.218.70.76 User: grader port: 2200

### 1.	installed packages
	sudo apt-get update 
	sudo apt-get upgrade
	sudo dpkg-reconfigure --priority=low unattended-upgrades
	sudo apt-get dist-upgrade(if all packages get updated)
### 2.	install finger
	sudo apt-get install finger
### 3.	Change the SSH Port from 22 to 2200
	Sudo nano /etc/ssh/sshd_config
	Find the line with Port22, changed to Port2200
	Sudo service ssh restart
### 4.	Create a new user, “grader”.
	Sudo adduser grader
### 5.	Giving the “grader” sudo access
	Sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
	sudo nano /etc/sudoers.d/grader
	change the second line to grader to equal grader ALL=(ALL_ NOPASSWD:ALL
	Save and exit
### 6.	Disable SSH for root user
	Sudo nano /etc/ssh/sshd_config
	Find ‘PermitRootLogin prohibit-password’
	Remove ‘prohibit-password’, place ‘no’
	Save and Exit
	Sudo service ssh restart
### 7.	Login as “grader”
	Sudo login grader
### 8.	Created authorized keys files under user “grader”
	Mkdir .ssh
	Touch .ssh/authorized_keys
	Sudo nano .ssh/authorized_keys
	Open new terminal (on client PC or Mac), run ssh-keygen
	Give name to the file
	Open .pub, using cat {filename.pub}, copy contents
	Back to lightsail terminal, paste in the contents from .pub.
	Find box to bottom right, copy contents in there, click in lightsail terminal, click Shift+Alt+Ctrl, double click or 		right click, should paste info
	Save and Exit
	Chmod 700 .ssh
	Chmod 644 .ssh/authorized_keys
	Back to opened terminal (on client PC or Mac)
	Ssh  -i {privatekeyfile} -p 2200 grader@{publicipaddres} 
### 9.	Forcing Key Based Authentication 
	sudo nano /etc/ssh/sshd_config 
		(set the line start with PasswordAuthentication to no)
	Sudo service ssh restart
### 10.	Configured Uncomplicated Firewall(UFW) to allow all incoming connections for SSH, HTP and N
	Sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp	
	Sudo ufw allow 123/udp
	Sudo ufw enable
### 11.	 Local timezone to UTC
	Sudo timedatectl set-timezone UTC
### 12.	 Install Apache Server Service
	Sudo apt-get install apache2
	Sudo service apache2 restart
### 13.	 Install mod_sgi
	Sudo apt-get install libapache2-mod-wsgi python-dev
	bSudo service apache2 restart
### 14.	Install and configure POSTGRESQL
	Sudo apt-get install postgresql
	No remote connections are allowed: sudo vim /etc/postgresql/9.3/main/pg_hba.conf
	Sudo su – postgres
	Psql
	CREATE DATABASE catalog;
	CREATE USE catalog;
	ALTER ROLE catalog WITH PASSWORD ‘password’;
	GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	\q
	exit
### 15.	 Clone git repo
	Cd /var/www
	Sudo mkdir FlaskApp
	Cd FlaskApp
	Sudo git clone https://github.com/ad988x/Item_Catalog.git
	Sudo mv ./Item_Catalog ./FlaskApp
	cd FlaskApp
	Sudo mv project.py __init__.py
	Edit:
		Database_setup.py, project.py, soccer_team_populate_db.py 
		create_engine('sqlite:///itemcatalogwithuser.db') to engine =  		create_engine('postgresql://catalog:password@localhost/catalog')
	Sudo apt-get install python-pip
	Sudo pip install Flask
	Sudo pip install sqlalchemy
	Sudo pip install oauth2client
	Sudo pip install requests
	sudo apt-get -qqy install postgresql python-psycopg2
	sudo python database_setup.py
	
### 16. Log into __init__.py and hard code the paths to .json files
	Sudo nano __init__.py
	Find all .json files and input ‘/var/www/FlaskApp/FlaskApp/’
	Save and Exit
	
### 17. Go to google api console, download JSON and update, client_secrets.json
	Cd /var/www/FlaskApp/FlaskApp
	Sudo nano client_secrets.json
	Delete posting in there and past new from google.
	Save and Exit.
	
### 18.	Configure and enable new virtual host 
	Sudo nano /etc/apache2/sites-available/FlaskApp.conf
	<VirtualHost *:80>
	    ServerName 18.218.70.76
	   ServerAdmin duzy2172@gmail.com	    	    
	    WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	    <Directory /var/www/FlaskApp/FlaskApp>
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

	Enable VH, sudo a2ensite catalog
	
### 19. Create a catalog.wsgi file to serve the application over the mod_wsgi
	go to the html folder: `$ cd /var/www/FlaskApp`
	Edit the file: `$ sudo nano flaskapp.wsgi`
	Insert these lines:
     	#!/usr/bin/python
	import sys
   	import logging
	
   	logging.basicConfig(stream=sys.stderr)
   	sys.path.insert(0, "/var/www/FlaskApp/FlaskApp")
	
   	from __init__ import app as application
	application.secret_key = 'super_secret_key'
		
### 20.	Restart Apache sudo service apache2 restart

## REFERENCES
	https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 	http://flask.pocoo.org/docs/0.12/patterns/packages/



