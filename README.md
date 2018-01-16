IP: 18.217.12.109 User: grader port: 2200

###1.	installed packages
a.	sudo apt-get update 
b.	sudo apt-get upgrade
###2.	install finger
a.	sudo apt-get install finger
###3.	Change the SSH Port from 22 to 2200
a.	Sudo nano /etc/ssh/sshd_config
b.	Find the line with Port22, changed to Port2200
c.	Sudo service ssh restart
###4.	Create a new user, “grader”.
a.	Sudo adduser grader
###5.	Giving the “grader” sudo access
a.	Sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
b.	sudo nano /etc/sudoers.d/grader
c.	change the second line to grader to equal grader ALL=(ALL_ NOPASSWD:ALL
d.	Save and exit
###6.	Disable SSH for root user
a.	Sudo nano /etc/ssh/sshd_config
b.	Find ‘PermitRootLogin prohibit-password’
c.	Remove ‘prohibit-password’, place ‘no’
d.	Save and Exit
e.	Sudo service ssh restart
###7.	Login as “grader”
a.	Sudo login grader
###8.	Created authorized keys files under user “grader”
a.	Mkdir .ssh
b.	Touch .ssh/authorized_keys
c.	Sudo nano .ssh/authorized_keys
d.	Open new terminal (on client PC or Mac), run ssh-keygen
e.	Give name to the file
f.	Open .pub, using cat {filename.pub}, copy contents
g.	Back to lightsail terminal, paste in the contents from .pub.
i.	Find box to bottom right, copy contents in there, click in lightsail terminal, click Shift+Alt+Ctrl, double click or right click, should paste info
h.	Save and Exit
i.	Chmod 700 .ssh
j.	Chmod 644 .ssh/authorized_keys
k.	Back to opened terminal (on client PC or Mac)
i.	Ssh  -i {privatekeyfile} -p 2200 grader@{publicipaddres} 
###9.	Forcing Key Based Authentication 
a.	sudo nano /etc/ssh/sshd_config 
i.	(set the line start with PasswordAuthentication to no)
b.	Sudo service ssh restart
###10.	Configured Uncomplicated Firewall(UFW) to allow all incoming connections for SSH, HTP and N
a.	Sudo ufw default deny incoming
b.	Sudo ufw default allow outgoing
c.	Sudo ufw allow 2200/tcp
d.	sudo ufw allow 80/tcp
e.	Sudo ufw allow http
f.	Sudo ufw allow ntp
g.	Sudo ufw enable
###11.	 Local timezone to UTC
a.	Sudo timedatectl set-timezone UTC
###12.	 Install Apache Server Service
a.	Sudo apt-get install apache2
###13.	 Install mod_sgi
a.	Sudo apt-get install libapache2-mod-wsgi python-dev
b.	Sudo a2enmod wsgi
###14.	Install and configure POSTGRESQL
a.	Sudo apt-get install postgresql
b.	No remote connections are allowed: sudo vim /etc/postgresql/9.3/main/pg_hba.conf
c.	Sudo su – postgres
d.	Psql
e.	CREATE DATABASE catalog;
f.	CREATE USE catalog;
g.	ALTER ROLE catalog WITH PASSWORD ‘password’;
h.	GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
i.	\q
j.	exit
###15.	 Clone git repo
a.	Sudo mkdir /var/www/catalog
b.	Cd /var/www/catalog
c.	Sudo git clone https://github.com/ad988x/Item_Catalog.git
d.	Edit:
i.	Database_setup.py, project.py, soccer_team_populate_db.py 
ii.	create_engine('sqlite:///itemcatalogwithuser.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog')
e.	Sudo apt-get install python-pip
f.	Sudo pip install Flask
g.	Sudo pip install sqlalchemy
h.	Sudo pip install oauth2client
i.	Sudo pip install requests
j.	sudo apt-get -qqy install postgresql python-psycopg2
k.	sudo python database_setup.py
###16.	Create a catalog.wsgi file to serve the application over the mod_wsgi
a.	go to the html folder: `$ cd /var/www/html`
b.	Create the file: `$ sudo touch catalog.wsgi`
c.	Edit the file: `$ sudo nano catalog.wsgi`
d.	Insert these lines:
     	#!/usr/bin/python
		import sys
   	import logging
   	logging.basicConfig(stream=sys.stderr)
   	sys.path.insert(0, "/var/www/catalog/")
   	from catalog import app as application
		application.secret_key = 'super_secret_key'
###17.	Configure and enable new virtual host 
a.	Sudo nano /etc/apache2/sites-available/catalog.conf
<VirtualHost :80>
	    ServerName 18.217.12.109
	   ServerAdmin admin@18.217.12.109	    	    
	    WSGIScriptAlias / /var/www/catalog.wsgi
	    <Directory /var/www/catalog/>
	        Order allow,deny
	        Allow from all
	    </Directory>
	    Alias /static /var/www/catalog/catalog/static
	    <Directory /var/www/catalog/static/>
	        Order allow,deny
	        Allow from all
   </Directory>
	    ErrorLog ${APACHE_LOG_DIR}/error.log
	    LogLevel warn
	    CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
b.	Enable VH, sudo a2ensite catalog
c.	Sudo service apache2 reload
###18.	Restart Apache sudo service apache2 restart



