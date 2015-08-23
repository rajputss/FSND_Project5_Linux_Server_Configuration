#LINUX SERVER CONFIGURATION

<img width="1210" alt="screen shot 2015-08-23 at 12 59 13 pm" src="https://cloud.githubusercontent.com/assets/4694107/9429970/9fd08bbc-49a8-11e5-83d7-27d9c51dc07f.png">

In this project I have taken a baseline installation of a Linux distribution on a virtual machine and prepare it to host my catalog web application that I built going through Udacity's Project 3 course, that comprises of Full Stack Foundation and Authentication and Authorization: OAUTH. This web application can be accessed at http://52.10.248.83/
as well as ec2-52-10-248-83.us-west-2.compute.amazonaws.com
Following the course for Project5, I have ensured that restaurant menu application is secured and prepared to withstand number of attack vectors. This includes any updates, upgrades, and configuration of web and database servers as well installation of different packages required to get this app running. 

* Step 1 Launch your Virtual Machine with your [Udacity's account](https://www.udacity.com/account#!/development_environment). This is provided in Project Details.

* Step 2 Follow the instructions provided to SSH into your server.

	- Download Private Key.
	
	- Move the private key file into the folder ~/.ssh (where ~ is your environment's home 
		directory). So if you downloaded the file to the Downloads folder, just execute the 
		following command in your terminal.
		
		mv ~/Downloads/udacity_key.rsa ~/.ssh/
		
	- Open your terminal and type in:
	
		chmod 600 ~/.ssh/udacity_key.rsa
		
	- In your terminal, type in:
	
		ssh -i ~/.ssh/udacity_key.rsa root@PUPLIC-IP-ADDRESS


* Step 3 Create a new user named grader.

	- adduser grader
	
* Step 4 Give the grader the permission to sudo.

	- Connect as grader from your local
	
		ssh grader@PUPLIC-IP-ADDRESS -p 22
	
	- Type in: 
		
		visudo
	
		a. Under root ALL=(ALL:ALL) ALL, type in:
			
			grader ALL=(ALL:ALL) ALL
		
* Step 5 Update all currently installed packages, typing following commands:

	- su - grader
	
	- sudo apt-get update
	
	- sudo apt-get upgrade
	
	
* Step 6 Change the SSH port from 22 to 2200.
	
	- On local machine type the following command:
		a. ssh-keygen => You can pick whatever password you want. 
		
		b. Enter file name /Users/shawnrajput/.ssh/linuxCourse
	
	- To place the key on remote server. Log into server as the user.
		
		a. mkdir .ssh
		
		b. touch .ssh/authorized_keys => This will create a file within your .ssh folder.
		
		c. From your local machine terminal type the following:
			
			I. cat .ssh/linuxCourse.pub => This will show the key id that we need for authentication
		
		c. Now on your terminal where you are signed in as grader type the following:
			
			I. nano .ssh/authorized_keys and paste the key id that you copied from .ssh/linuxCourse.pub
			
		d. Now type the following commands to set permissions for .ssh and .ssh/authorized_keys
			
			I. chmod 777 .ssh
			
			II. chmod 644 .ssh/authorized_keys
	
	- Disable password based login. Type the following commands to complete this step
		
		a. sudo nano /etc/ssh/sshd_config => open the config file
		
		b. Replace the current Port with 2200 => change the port from 22 to 2200
		
		c. Replace no with yes for PasswordAuthentication 
		
		d. AllowUsers grader => Add it at the bottom of the file
		
		e. /etc/init.d/ssh restart or sudo service ssh restart => Restart service
		
		f. ssh -v grader@PUBLIC-IP-ADDRESS -p2200 => Log in using this command as grader 

* Step 7 Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
	
	1. sudo ufw default deny incoming 
	
	2. sudo ufw default allow outgoing
	
	3. sudo ufw allow ssh
	
	4. sudo ufw allow 2222/tcp
	
	5. sudo ufw allow www
	
	6. sudo ufw allow enable
	
	7. sudo ufw status => This will show you that ufw is enabled and activated.
	
* Step 8 Configure the local timezone to UTC
	
	1. sudo dpkg-reconfigure tzdata
	2. From the provided list, select OK for None of the above.
	3. From the provided list, select OK for UTC.
	
* Step 9 Install and configure Apache to serve a Python mod_wsgi application
	
	1. sudo apt-get install apache2
	
	2. Go to http://PUBLICIPADDRESSGOESHERE/ in your browser. You will see a message that states "It works!" on the page.
	
	3. sudo apt-get install python-setuptools libapache2-mod-wsgi
	
	4. sudo service apache2 restart
	

* Step 11 Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
	
	First we ned to install and configure git:
	
	- sudo apt-get install git 
	
	- git config --global user.name "Your name"
	
	- git config --global user.email "youremail@email.com"
	
	Now we need to setup for deploying a Flask Application on Ubuntu VPS:
	
	- sudo apt-get install libapache2-mod-wsgi python-dev
	
	- sudo a2enmod wsgi
	
	- cd /var/www
	
	- sudo mkdir catalog
	
	- cd catalog
	
	- sudo mkdir catalog
	
	- cd catalog
	
	- sudo mkdir static templates
	
	- sudo nano __init__.py
	
	- Type the following: 
```	
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Digital Ocean!"
if __name__ == "__main__":
    app.run()
```
	
	- sudo apt-get install python-pip
	- sudo apt-get install Flask
	- sudo nano /etc/apache2/sites-available/catalog.conf
	- Type the following:
```	
<VirtualHost *:80>
  ServerName PUBLIC-IP-ADDRESS
  ServerAdmin admin@PUBLIC-IP-ADDRESS
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
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
	
```
	- sudo a2ensite catalog
	- cd /var/www/catalog
	- sudo nano catalog.wsgi
	- Type the following: 
```	
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
 
```
	- sudo service apache2 restart
	
	Now we need to clone github repo and make it web inaccessible:
	
	- git clone https://github.com/rajputss/Catalog.git
	- mv /var/www/catalog/Catalog/* /var/www/catalog/catalog/
	- delete the leftover empty directory Catalog 
	- Create and open .htaccess file:
	
		a. cd /var/www/catalog/
		b. sudo nano .htaccess
		c. Type the following:
		
```	
RedirectMatch 404 /\.git
```
		
	- pip install httplib2
	- pip install requests
	- sudo pip install --upgrade oauth2client
	- sudo pip install sqlalchemy
	- sudo apt-get install python-psycopg2
	
* Step 10 Install and configure PostgreSQL:

	- Create a new user named catalog that has limited permissions to your catalog application database.
	
		a. sudo apt-get install postgresql postgresql-contrib

		b. sudo nano /etc/postgresql/9.3/main/pg_hba.conf   => to check that there are no remote connections allowed.
		
		c. sudo nano database_setup.py and type
		
		engine = create_engine( 'postgresql://catalog:PASSWORDFORDBGOESHERE@localhost/catalog')
		
		d. sudo nano project.py and type 
		
		engine = create_engine( 'postgresql://catalog:PASSWORDFORDBGOESHERE@localhost/catalog')
		
		e. Anywhere you see fb_client_secrets.json or client_secrets.json, specify the absolute path in project.py.
		
		f. In my project, it is like this 
```
CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']
```
		
		and I changed it to the following:
``` 
CLIENT_ID = json.loads(open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
```			
		g. Also this line 
```
oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='') 
```

was changed to
```		
		oauth_flow = flow_from_clientsecrets(r'/var/www/catalog/catalog/client_secrets.json', scope='')
```

		h. sudo nano lotsofmenus.py
		
		engine = create_engine( 'postgresql://catalog:PASSWORDFORDBGOESHERE@localhost/catalog')
		
		i. mv project.py __init__.py
		j. sudo adduser catalog
		k sudo su - postgres

			1. Type psql
			2. CREATE USER catalog WITH PASSWORD 'PASSWORDFORDBGOESHERE';
			3. ALTER USER catalog CREATEDB;
			4. CREATE DATABASE catalog WITH OWNER catalog;
			5. \c catalog
			6. REVOKE ALL ON SCHEMA public FROM public;
			7. GRANT ALL ON SCHEMA public TO catalog;
			8. \q
			9. exit
	
		l. python database_setup.py

		m. python lotsofmenus.py
		

Back to completing the final steps for 11.

	1. Restart apache typing the following command:
	
	sudo service apache2 restart
	
	2. Now open a browser and put in your public ip-address as url, e.g. http://52.10.248.83/
	 
	Hopefully everything is working fine and application comes up

	3. If getting an internal server error, check the Apache error files by typing the following command
		
	sudo tail -20 /var/log/apache2/error.log
		
	4. For OAuth logins to work. Do the following:
		
		a. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 

		52.10.248.83, its ec2-52-10-248-83.us-west-2.compute.amazonaws.com

		b. Open the Apache configuration files for the web app: $ sudo vim 

		/etc/apache2/sites-available/catalog.conf

		c. Paste in the following line below ServerAdmin:
			
		ServerAlias HOSTNAME, e.g. ec2-52-10-248-83.us-west-2.compute.amazonaws.com
			
		d. Enable the virtual host:	
			
		sudo a2ensite catalog
		
		e. For Google OAuth to work, do the following.
			
			I. Go to the project on the [Developer Console](https://www.console.developers.google.com/project)

			II. Navigate to APIs & auth > Credentials > Click on Add Credentials drop down and select OAUTH 2.0 client ID
		
			III. Under JavaScript origins type your host name and your public ip-address like this:
			 
			http://ec2-52-10-248-83.us-west-2.compute.amazonaws.com
			http://52.10.248.83
	
			IV. Under Authorized redirect URIs type hostname with oauth2callback like this:
		
			http://ec2-52-10-248-83.us-west-2.compute.amazonaws.com/oauth2callback

		f. For Facebook authorization:
	
			I. Go on the Facebook Developers Site to [My Apps](https://www.developers.facebook.com/apps/)

			II. Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field

			III. To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'

### Various sources used to complete this project and for error resolution

First and foremost, I would like to credit Udacity and it's team; specifically Mike Wales. His awesome course have really helped me complete this. There are many sources that were used to actually understand this project rubric and complete it. This includes digital ocean, github project of fellow udacians and how they have completed this project, stack overflow etc. 

1. [Udacity Discussion Forums](https://discussions.udacity.com/c/nd004-p5-linux-based-server-configuration)
2. [Udacity's Linux Server Configuration Course](https://www.udacity.com/course/viewer#!/c-ud299-nd) For steps 1,2, 3, 4, 5, 6, 7, 8, 9, 
3. Github profiles with similar projects
	* (https://github.com/WebDesigner32)
	* (https://github.com/stueken)
4. [Update and upgrade all currently installed packages](http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade)
5. [Changing the port from 22 to 2200) (http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server)
	* I have learned from this site a bit but followed Mike Wales course for implementation
6. [Get rid of the warning message sudo: unable to resolve host ... when sudo is executed:](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none)
	* I learned of this source from fellow udacians [github profile](https://github.com/stueken)
7. [Handle the message System restart required after login:](http://superuser.com/questions/815433/how-urgent-is-a-system-restart-required-for-security)
	* I learned of this source from fellow udacians [github profile](https://github.com/stueken)
8. [Get rid of the message "Could not reliably determine the servers's fully qualified domain name" after restart](http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name)
	* I learned of this source from fellow udacians [github profile](https://github.com/stueken)
9. [Securing postgresql on an ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
	* I learned of this source from fellow udacians [github profile](https://github.com/WebDesigner32)
10. Step 10 that requires database setup. [github profile](https://github.com/WebDesigner32)
