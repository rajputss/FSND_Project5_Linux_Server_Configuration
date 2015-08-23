1. Launch your Virtual Machine with your Udacity account. This is provided in Project Details.

2. Follow the instructions provided to SSH into your server.

	1. Download Private Key.
	
	2. Move the private key file into the folder ~/.ssh (where ~ is your environment's home 
		directory). So if you downloaded the file to the Downloads folder, just execute the 
		following command in your terminal.
		
		mv ~/Downloads/udacity_key.rsa ~/.ssh/
		
	3. Open your terminal and type in:
	
		chmod 600 ~/.ssh/udacity_key.rsa
		
	4. In your terminal, type in:
	
		ssh -i ~/.ssh/udacity_key.rsa root@PUPLIC-IP-ADDRESS


3. Create a new user named grader.

	1. adduser grader
	
4. Give the grader the permission to sudo.

	1. Connect as grader from your local
	
		ssh grader@PUPLIC-IP-ADDRESS -p 22
	
	2. Type in: 
		
		visudo
	
		a. Under root ALL=(ALL:ALL) ALL, type in:
			
			grader ALL=(ALL:ALL) ALL
		
5. Update all currently installed packages, typing following commands:

	1. su - grader
	2. sudo apt-get update
	3. sudo apt-get upgrade
	
	
6.Change the SSH port from 22 to 2200.

	1. On local machine type the following command:
		
		a. ssh-keygen => You can pick whatever password you want. 
		b. Enter file name /Users/shawnrajput/.ssh/linuxCourse
	
	2. To place the key on remote server. Log into server as the user.
		
		a. mkdir .ssh
		b. touch .ssh/authorized_keys => This will create a file within your .ssh folder.
		c. From your local machine terminal type the following:
			
			I. cat .ssh/linuxCourse.pub => This will show the key id that we need for authentication
		
		c. Now on your terminal where you are signed in as grader type the following:
			I. nano .ssh/authorized_keys and paste the key id that you copied from .ssh/linuxCourse.pub
			
		d. Now type the following commands to set permissions for .ssh and .ssh/authorized_keys
			I. chmod 777 .ssh
			II. chmod 644 .ssh/authorized_keys
	
	3. Disable password based login. Type the following commands to complete this step
		
		1. sudo nano /etc/ssh/sshd_config => open the config file
		2. Replace the current Port with 2200 => change the port from 22 to 2200
		3. Replace no with yes for PasswordAuthentication 
		4. AllowUsers grader => Add it at the bottom of the file
		5. /etc/init.d/ssh restart or sudo service ssh restart => Restart service
		6. ssh -v grader@PUBLIC-IP-ADDRESS -p2200 => Log in using this command as grader 

7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
	
	1. sudo ufw default deny incoming 
	2. sudo ufw default allow outgoing
	3. sudo ufw allow ssh
	4. sudo ufw allow 2222/tcp
	5. sudo ufw allow www
	6. sudo ufw allow enable
	7. sudo ufw status => This will show you that ufw is enabled and activated.
	
8. Configure the local timezone to UTC
	
	1. sudo dpkg-reconfigure tzdata
	2. From the provided list, select OK for None of the above.
	3. From the provided list, select OK for UTC.
	
9. Install and configure Apache to serve a Python mod_wsgi application
	
	1. sudo apt-get install apache2
	2. Go to http://PUBLICIPADDRESSGOESHERE/ in your browser. You will see a message that states "It works!" on the page.
	3. sudo apt-get install python-setuptools libapache2-mod-wsgi
	4. sudo service apache2 restart
	

11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
	
	First we ned to install and configure git:
	
	1. sudo apt-get install git 
	2. git config --global user.name "Your name"
	3. git config --global user.email "youremail@email.com"
	
	Now we need to setup for deploying a Flask Application on Ubuntu VPS:
	
	4. sudo apt-get install libapache2-mod-wsgi python-dev
	5. sudo a2enmod wsgi
	6. cd /var/www
	7. sudo mkdir catalog
	8. cd catalog
	9. sudo mkdir catalog
	10. cd catalog
	11. sudo mkdir static templates
	12. sudo nano __init__.py
	13. Type the following: 
	
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Digital Ocean!"
if __name__ == "__main__":
    app.run()

	
	14. sudo apt-get install python-pip
	15. sudo apt-get install Flask
	16. sudo nano /etc/apache2/sites-available/catalog.conf
	17. Type the following:
	
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
	
	18. sudo a2ensite catalog
	19. cd /var/www/catalog
	20. sudo nano catalog.wsgi
	21. Type the following: 
	
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
 
	21. sudo service apache2 restart
	
	Now we need to clone github repo and make it web inaccessible:
	
	22. git clone https://github.com/rajputss/Catalog.git
	23. mv /var/www/catalog/Catalog/* /var/www/catalog/catalog/
	24. delete the leftover empty directory Catalog 
	25. Create and open .htaccess file:
	
		a. cd /var/www/catalog/
		b. sudo nano .htaccess
		c. Type the following:
			
			I. RedirectMatch 404 /\.git
	
	26. pip install httplib2
	27. pip install requests
	28. sudo pip install --upgrade oauth2client
	29. sudo pip install sqlalchemy
	30. sudo apt-get install python-psycopg2
	
10. Install and configure PostgreSQL:

	1. Create a new user named catalog that has limited permissions to your catalog application database.
	
		a. sudo apt-get install postgresql postgresql-contrib
		b. sudo nano /etc/postgresql/9.3/main/pg_hba.conf   => to check that there are no remote connections allowed.
		c. sudo nano database_setup.py and type
		
		engine = create_engine( 'postgresql://catalog:PASSWORDFORDBGOESHERE@localhost/catalog')
		
		d. sudo nano project.py and type 
		
		engine = create_engine( 'postgresql://catalog:PASSWORDFORDBGOESHERE@localhost/catalog')
		
		e. Anywhere you see fb_client_secrets.json or client_secrets.json, specify the absolute path in project.py.
		
		f. In my project, is like this CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']
		
		and I changed it to the following:
		 
		CLIENT_ID = json.loads(open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
			
		g. Also change this line oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='') was changed to
		
		oauth_flow = flow_from_clientsecrets(r'/var/www/catalog/catalog/client_secrets.json', scope='')
		
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

11. Back to completing final steps.

	1. Restart apache typing the following command:
	
	sudo service apache2 restart
	
	2. Now open a browser and put in your public ip-address as url, e.g. http://52.10.248.83/
	 
	   Hopefully everything is working fine and application comes up
	3. If getting an internal server error, check the Apache error files by typing the following command
		
		sudo tail -20 /var/log/apache2/error.log
		
	4. For OAuth logins to work. Do the following:
		
		a. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 52.10.248.83, its ec2-52-10-248-83.us-west-2.compute.amazonaws.com
		b. Open the Apache configuration files for the web app: $ sudo vim /etc/apache2/sites-available/catalog.conf
		c. Paste in the following line below ServerAdmin:
			
			ServerAlias HOSTNAME, e.g. ec2-52-10-248-83.us-west-2.compute.amazonaws.com
			
		d. Enable the virtual host:	
			
			sudo a2ensite catalog
		
		e. For Google OAuth to work:
			
			I. Go to the project on the Developer Console: https://console.developers.google.com/project
			II. Navigate to APIs & auth > Credentials > Click on Add Credentials drop down and select OAUTH 2.0 client ID
			III. Under JavaScript origins type your host name and your public ip-address like this:
			 
				http://ec2-52-10-248-83.us-west-2.compute.amazonaws.com
				http://52.10.248.83
			
			IV. Under Authorized redirect URIs type hostname with oauth2callback like this:
				
				http://ec2-52-10-248-83.us-west-2.compute.amazonaws.com/oauth2callback
		
		f. For Facebook authorization:
			
			I. Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
			II. Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
			III. To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'