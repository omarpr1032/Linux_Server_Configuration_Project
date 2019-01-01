Udacity Linux Server Configuration
==================================
This project consist on create a Linux virtual machine and configure it to host the item catalog web application. This task was made for the Udacity's Full Stack Web Developer Nanodegree. You can visit the website deployed at <http://34.228.216.167>.


#Tasks

- Create a Linux virtual machine instance
- Access via SSH to the instance
- Configure the local timezone and locale
- Create the user grader with the sudo permission and ssh access
- Disable ssh login for root user
- Enforce Key-based SSH authentication
- Update the installed packages
- Configure the SSH port to 2200
- Configure the UFW(Uncomplicated Firewall)
- Install Apache Server and the mod_wsgi module
- Install PostgreSQL
- Clone the Catalog app from Github
- Install Catalog App dependencies
- Update Catalog App Configuration
- Configure and enable a new virtual host

## Create a Linux virtual machine instance

- Enter to <https://signin.aws.amazon.com/> to create a new AWS account.
- Select the `Create instance` button.
- On Pick your instance image select  the `Linux/Unix` option
- Select a `OS Only` blueprint and pick `Ubuntu`
- In optional section SSH key pair manager `Create New +` ssh key and download. it to your machine and move the private key on your `~/.ssh` folder.
- Identify your instance.
- Press the `Create instance` button.
- Environment Information:
	><b>Instance Name</b>: Ubuntu
	
	><b>Public IP</b>: 34.228.216.167
	
	><b>Private Key</b>: Not provided here 


## Access via SSH to the instance

- Open the terminal and enter the command `chmod 400 ~/.ssh/sshkey.pem`.
- On the terminal type `ssh -i ~/.ssh/sshkey.pem ubuntu@34.228.216.167`.

## Configure the local timezone and locale

- `sudo dpkg-reconfigure tzdata` - To set the timezone run the command 
- `export LANGUAGE=en_US.UTF-8;export LC_ALL=en_US.UTF-8`

## Create a new user grader
Run on the terminal the following commands:

- `sudo adduser grader` - Add the new user.
- `sudo nano /etc/sudoers.d/grader` - Create the grader file and type:

		grader ALL=(ALL:ALL) ALL and save the file.
- `su - grader` - To login as grader user.
- `mkdir .ssh` - Makes the .ssh directory.
- `nano .ssh/authorized_keys`

	>On your local machine create a new key pair typing `ssh-keygen`, then copy and paste the key to this file and save it.
- `chmod 700 .ssh` - Change the directory permission
- `chmod 644 .ssh/authorized_keys` - Change the file premission
- `sudo service ssh restart` - Restarts the ssh service
- `exit`
- On the terminal type `ssh -i ~/.ssh/graderKey grader@34.228.216.167` to login as grader.

##Disable ssh login for root user
- `sudo nano /etc/ssh/sshd_config`
- Change PermitRootLogin without-password line to `PermitRootLogin no`
- `sudo service ssh restart` Restart the  ssh service 

##Enforce Key-based SSH authentication
- `sudo nano /etc/ssh/sshd_config`
- Change PasswordAuthentication yes line to `PasswordAuthentication no`
- `sudo service ssh restart` Restart the  ssh service 

## Update the installed packages
- Type `sudo apt-get update` to downloads the package lists from the repositories.
- Type `sudo apt-get upgrade` to fetch the new versions of packages.
- Also can type `sudo apt-get full-upgrade` to upgrades packages with auto-handling of dependencies.
- If the system requires a restart run the command `sudo reboot`.

##Configure the SSH port to 2200
To configure the SSH Port perform the followings instructions:

- Run the command `sudo nano /etc/ssh/sshd_config`.
- Change the port `22` to `2200` and save.
- Restart the ssh service typing `sudo service ssh restart`.
- Go to the instance management page and select the Networking tab.
- On the Firewall section and change add a new <b>custom TCP port 2200</b>.
- Type `exit`.
- Login again `ssh -i ~/.ssh/graderKey -p 2200 grader@34.228.216.167` with the new SHH port.

## Configure the UFW(Uncomplicated Firewall)

To configure the UFW run the followings commands:

- `sudo ufw allow 2200/tcp` - Allow SSH port 2200
- `sudo ufw allow 80/tcp` - Allow HTTP port 80
- `sudo ufw allow 123/udp` - Allow NTP port 123
- `sudo ufw enable` - Enable the Firewall

## Install Apache Server and the mod_wsgi module

- `sudo apt-get install apache2` - Install Apache serve
- `sudo apt-get install python-setuptools libapache2-mod-wsgi` - Installs the python wsgi_mod module
- `sudo service apache2 restart`

## Install PostgreSQL
- `sudo apt-get install postgresql` - Install the postgreSQL
- `sudo su - postgres` - login as postgres user
- `psql` - Entes in the postgreSQL shell
- On the postgreSql shell execute the commands:

		1. CREATE DATABASE catalog; - Create the database named catalog
		2. CREATE USER catalog; - Create the user catalog
		3. ALTER ROLE catalog WITH PASSWORD 'password'; - Change the catalog user password
		4. GRANT ALL PRIVILEGES ON DATABASE catalog To catalog; - Give to the user catalog permission to the catalog database.
		5. \q - exit from the postgreSQL shell
	
- `exit` -exit from the postgreSQL application

## Clone the Catalog app from Github

- `sudo apt-get install git` - Install git
- `cd /var/www` - Go to this directory
- `sudo mkdir catalog` - Create a catalog directory
- `sudo chown -R grader:grader catalog` - Change the directory owner
- `cd catalog` - Go to this directory
- `git clone https://github.com/omarpr1032/Item-Catalog-Project.git catalog` - Clone the catalog github project
- `mv application.py __init__.py` - Rename application.py to init.py
- `nano catalog.wsgi` - Create wsgi file and type:

		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0, "/var/www/catalog/")

		from catalog import app as application
		application.secret_key = 'supersecretkey'	
- `cd catalog` - Go to this directory
- `mv application.py __init__.py` - Rename application.py to init.py

## Install Catalog App dependencies
- `sudo pip install flask httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils psycopg2-binary requests` - Install other project dependencies

## Update Catalog App Configuration
- `nano __init__.py` - Open and change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
- Open the files `__init__.py, database_setup.py and add_data.py` and change the database engine to `postgresql://catalog:password@localhost/catalog`;
- `python database_setup.py` - Create the application tables
- `python add_data.py` - Add dummy data to the database tables

## Configure and enable a new virtual host
- `sudo nano /etc/apache2/sites-available/catalog.conf` - Create catalog.conf and paste:

		<VirtualHost *:80>
    		ServerName 34.228.216.167
    		ServerAlias ec2-34-228-216-167.us-west-2.compute.amazonaws.com
    		ServerAdmin admin@34.228.216.167
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
    		LogLevel warn
    		CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>

- `sudo service apache2 reload` - Reload the apache server
- `sudo a2ensite catalog` - Enable the virtual host
- `sudo systemctl reload apache2` - Activate the new configuration

# Contact
For any questions please feel free to contact me at omarpr1032@gmail.com
