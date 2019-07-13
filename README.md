# Linux web server deployment

This is one of Udacity's Full Stack Web Developer Nanodegree Program 's projects using Linux - Ubuntu 16.04 LTS, [AWS Lightsail](https://lightsail.aws.amazon.com/), PostgreSQL and Apache. 
The project involves deploying an already [created application](https://github.com/cicomsa/Catalog-App).

URLs: [http://35.178.211.88](http://35.178.211.88) and [http://35.178.211.88.xip.io](http://35.178.211.88.xip.io)

### IP address and SSH port:

* Public IP address: 35.178.211.88
* Port: 2200

### Start an Ubuntu Linux server instance on Amazon Lightsail

* Login into [AWS Lightsail](https://lightsail.aws.amazon.com/)
* Click ```Create instance```
* Choose ```Linux/Unix platform```, ```OS Only``` and ```Ubuntu 16.04 LTS```
* Choose a plan instance
* Provide the instance a name(optional)
* Click ```Create``` button

### SSH into your server

* On AWS Lightsail, click on ```Account``` button in the header and select ```Account```
* Click on ```SSH keys``` and then on ```Download```
* Rename the downloaded file ```lightsail_key.rsa``` and move it to ```~/.ssh``` folder in your local machine
* In your local machine server run ```chmod 600 ~/.ssh/lightsail_key.rsa```
* Connect to your instance with ```ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.178.211.88``` where ```35.178.211.88``` is your instance's Public IP address

### Update and upgrade installed packages

* In your instance server run ```sudo apt-get update``` and ```sudo apt-get upgrade```

### Change the SSH port from 22 to 2200

* In your instance server run ```sudo nano /etc/ssh/sshd_config```
* Change ```Port 22``` line to ```Port 2200```
* Save and exit file
* Restart SSH with ```sudo service ssh restart```

### Configure the Uncomplicated Firewall (UFW) - allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

In your instance server run:
* ```sudo ufw status ``` - The UFW should be inactive. 
* ```sudo ufw default deny incoming``` - To deny incoming traffic
* ```sudo ufw default allow outgoing``` - To enable outgoing traffic
* ```sudo ufw allow 2200/tcp``` - To allow incoming tcp packets on port 2200
* ```sudo ufw allow www``` - To allow HTTP traffic in.
* ```sudo ufw allow 123/udp``` - To allow incoming udp packets on port 123
* ```sudo ufw deny 22``` - To deny tcp and udp packets on port 22.
* ```sudo ufw enable``` - Turn UFW on
* Proceed with the ssh connections disruptions
* ```sudo ufw status``` - To check UWF rules status
* Exit the SSH connection: ```exit```

On [AWS Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances):
* Click on the three vertical dots on your instance and then on ```Manage```
* Click on ```Networking``` and in the Firewall section click on ```Edit rules```
* Edit to have the following rules ONLY: ```HTTP TPC 80```, ```Custom UDP 123``` and ```Custom TPC 2200```
* Save

In your local machine server:
* Connect to your instance ```ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@35.178.211.88``` where ```35.178.211.88``` is your instance's Public IP address

### Updated packages to most recent versions

* In your instance server run ```sudo apt-get update```, ```sudo apt-get dist-upgrade``` 
* Shut down the connection: ```sudo shutdown -r now``` and connect back to your instance: ```ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@35.178.211.88``` where ```35.178.211.88``` is your instance's Public IP address

### Create a new user account named ```grader```

* In your instance server run: ```sudo adduser grader```
* Enter a password and fill in the rest of the details for the new user
* Then run ```sudo visudo```
* Give ```grader``` sudo privilages: add line ```grader  ALL=(ALL:ALL) ALL``` under ```root    ALL=(ALL:ALL) ALL```
* Save and exit file

### Create an SSH key pair for ```grader``` using the ```ssh-keygen``` tool

In your local machine server:
* Run ```ssh-keygen```
* Enter file where to save key: ```grade_key```
* Enter passphrase 
* Run ```cat ~/.ssh/grade_key.pub``` and copy the contents of the file
* * Connect to your instance with ```ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.178.211.88``` where ```35.178.211.88``` is your instance's Public IP address
* Log in to the ```grader```'s virtual machine: ```su - grader```

On ```grader```'s virtual machine server:
* Create directory ```.ssh``` with ```mkdir .ssh```
* Run ```sudo nano ~/.ssh/authorized_keys``` and paste the copied content into this file
* Save and exit file
* Run ```chmod 700 .ssh``` and ```sudo chmod 644 .ssh/authorized_keys``` to give permissions
* Run ```sudo nano /etc/ssh/sshd_config``` ; ```PasswordAuthentication``` and ```PermitRootLogin``` must be set to ```no```
* Restart SSH: ```sudo service ssh restart```
* Run ```exit``` twice until back on your local machine server

On your local machine server connect back to the instance as ```grader``` with ```ssh -i ~/.ssh/grader_key -p 2200 grader@35.178.211.88``` where ```35.178.211.88``` is your instance's Public IP address and ```grader_key``` is the file in which the ssh-key was saved in

### Configure the local timezone to UTC

* While connected as ```grader```, run ```sudo dpkg-reconfigure tzdata```
* If locale errors, run ```export LC_ALL="en_US.UTF-8"```, ```export LC_CTYPE="en_US.UTF-8"``` and ```sudo dpkg-reconfigure locales```

### Install and configure Apache to serve a Python mod_wsgi application

* While connected as ```grader```, run ```sudo apt-get install apache2``` to install Apache2
* Run ```sudo apt-get install libapache2-mod-wsgi-py3``` to install Python 3 mod_wsgi package
* Enable ```mod_wsgi``` running in the server ```sudo a2enmod wsgi```

### Install and configure PostgreSQL

Step 1:
* While connected as ```grader```, run ```sudo apt-get install postgresql``` to install postgreSQL
* Run ```sudo su - postgres``` to switch to ```postgres``` user
* Run ```psql``` to open PostgreSQL server
* Create user ```catalog``` with ```postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';```
* Give ```catalog``` the ability to create databases: ```ALTER ROLE catalog CREATEDB;```
* Run ```\q``` to exit PostgreSQL server and ```exit``` to go back to ```grader```

Step2:
* While connected as ```grader```, create user ```catalog``` with ```sudo adduser catalog```
* Run ```sudo visudo``` and add line ```catalog  ALL=(ALL:ALL) ALL``` under ```grader  ALL=(ALL:ALL) ALL```
* Save and exit file

Step3:
* Connect as ```catalog``` user with ```su - catalog```
* Run ```createdb catalog``` to create ```catalog``` database
* Run ```\q``` to exit PostgreSQL server and ```exit``` to go back to ```grader```

### Clone Application 

* While connected as ```grader```, install ```git``` with ```sudo apt-get install git```
* Run ```mkdir /var/www/catalog/``` and ```cd /var/www/catalog/```
* Run ```sudo git clone https://github.com/cicomsa/Catalog-App catalog```
* Run ```sudo chown -R grader:grader /var/www/catalog/``` fot ```grader``` to own the directory

### Application files changes

* While connected as ```grader```, go to project directory ``` cd /var/www/catalog/catalog``` and rename ```aplication.py``` file to ```__init__.py``` with ```mv application.py __init__.py```
* Run ```sudo nano __init__.py``` 
* Replace the last 2 lines with ```app.run()```
* Replace ```engine = create_engine('sqlite:///categoriesitems.db', connect_args={'check_same_thread': False})``` with ```engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')```
* Replace ```app.secret_key = b'_8#y2L"F4Q8z\n\xec]/'``` with ```app.secret_key = secretkey'```
* Save and exit file
* Run ```sudo nano catalog.py``` and replace ```engine = create_engine('sqlite:///categoriesitems.db')``` with ```engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')```
* Run ```sudo nano database.py``` and replace ```engine = create_engine('sqlite:///categoriesitems.db')``` with ```engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')```

### Authenticate login through Google

* In [Google Cloud Platform](https://console.cloud.google.com/) click ```APIs & services``` on left menu and then on ```Credentials```.
* Click on ```Create Credentials``` and then on ```OAuth client ID``` 
* Add ```http://<PUBLIC ID ADDRESS>``` and ```http://<PUBLIC ID ADDRESS>.xip.io``` as authorized JavaScript origins.
* ```http://<PUBLIC ID ADDRESS>.xip.io``` will also be added as an authorized JavaScript origins in the main app configurations - if error, click on the error link and add the URL as an authorized URL
* Back to previous configuration page, add ```http://<PUBLIC ID ADDRESS>.xip.io/oauth2callback``` as authorized redirect URI.
* Download the corresponding JSON file, open it and copy the contents.

In ```grader```'s virtual machine:
* While connected as ```grader```run ```sudo nano /var/www/catalog/catalog/client_secret.json``` and paste the copied contents into the file.
* Save and exit file
* Run ```sudo nano templates/login.html``` and replace the ```client_ID```  value with the one from the ```client_secrets.json```

### Install the virtual environment 

* Connected as ```grader```, run ```sudo apt-get install python3-pip``` and ```sudo apt-get install python-virtualenv```
* Run ```/var/www/catalog/catalog/```
* Create a virtual environment with ```sudo virtualenv -p python3 venv3``` and run ```sudo chown -R grader:grader venv3/```
*  Run ```. venv3/bin/activate```

### Install dependencies

Inside venv3 run:
* ```pip install httplib2```
* ```pip install requests```
* ```pip install --upgrade oauth2client```
* ```pip install sqlalchemy```
* ```pip install flask```
* ```sudo apt-get install libpq-dev```
* ```pip install psycopg2```
* ```python3 __init__.py```
* ```CTRL+C```
* ```deactivate```

### Set up virtual host

* Connected as ```grader``` run ```/etc/apache2/mods-enabled/wsgi.conf```
* Under line ```#WSGIPythonPath directory|directory-1:directory-2:...``` add ```WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages```
* Save and exit
* Run ```sudo nano /etc/apache2/sites-available/catalog.conf```
* Add lines:

```
<VirtualHost *:80>
    ServerName <PUBLIC IP address>
    ServerAlias www.<<PUBLIC IP address>.xip.io
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

* Save and exit file

### Enable virtual host

* Connected as ```grader``` run ```sudo a2ensite catalog``` and ```sudo service apache2 reload```

### Set up the Flask application

* Connected as ```grader``` run ```sudo nano /var/www/catalog/catalog.wsgi```.
* Add lines:

activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "secretkey"
```

* Save and exit file
* Run ```sudo service apache2 restart```

### Set up the database

* Connected as ```grader``` run ```sudo nano /var/www/catalog/catalog/database.py```
* Insert at the beginning of the file:

import sys
sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.5/site-packages")

* Save and exit file
* Run ```cd /var/www/catalog/catalog/```, ```. venv3/bin/activate```, ```python3 database.py``` to populate database
* Run ```deactivate```

### Disable the default Apache site

* Connected as ```grader``` run ```sudo a2dissite 000-default.conf``` and ```sudo service apache2 reload```

### Launch the Web Application

* Connected as ```grader``` run ```sudo chown -R www-data:www-data /var/www/catalog/catalog/```
* ```sudo service apache2 restart```
* The application is live on ```http://<PUBLIC IP ADDRESS>```
* To be able to use Google OAuth use ```http://<PUBLIC IP ADDRESS>.xip.io```

### Log application messages

* Connected as ```grader``` run ```sudo tail /var/log/apache2/error.log```

### Helpful resources and thankful to:
* [Stackoverflow.com](https://stackoverflow.com) forums
* [Github.com](https://github.com/) forums
* [Askubuntu.com](https://askubuntu.com/) forums
* [Udacity's Students Hub](https://knowledge.udacity.com/)
* [Digital Ocean](https://www.digitalocean.com/community/tutorials) tutorials
* [ashutosh-sharma/Linux-Server-Configuration-Project-6-FSND-Udacity](https://github.com/ashutosh-sharma/Linux-Server-Configuration-Project-6-FSND-Udacity)
* [boisalai/udacity-linux-server-configuration](https://github.com/boisalai/udacity-linux-server-configuration/tree/8880a6f69ef47580bec84e55df496e4b3f247550)
* [golgtwins/Udacity-P7-Linux-Server-Configuration](https://libraries.io/github/golgtwins/Udacity-P7-Linux-Server-Configuration)
