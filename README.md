## Overview 

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

# Server details

- URL: [http://35.177.179.71/](http://35.177.179.71/)
SSH port: `2200`

- In this project we used [Amazon Lightsail](https://lightsail.aws.amazon.com/) 

## On the server side 

- Add user
using `sudo adduser grader`

- Add user to the sudo group 
using `sudo usermod -a -G sudo grader`

- Update Packages 
using `apt-get update` then `apt-get upgrade`


## On your local machine 

- generate key pairs using `ssh-keyhen 

- place the public key on your remote server 
get key by `cat .ssh/linuxp.pub` 


## Back on the remote server 

- allow SSH keys for user grader
`mkdir .ssh` then `touch .ssh/authorized_keys`

- paste the pub key `nano .ssh/authorized_keys`  

-`chmod 700 .shh` then `chmod 644 .shh/authorized_keys`

- Forcing Key Based Authentication 
by `sudo nano /etc/ssh/sshd_config` to change 
`PasswordAuthentication` to no 

- for the changes to take effect type `sudo service ssh restart`

- You can login as the `grader` user using 
`ssh grader@35.177.179.71 -p 22 -i ~/.ssh/linuxp6`

- Change timezone to UTC by `sudo timedatectl set-timezone UTC`

- Add SSH port 2200 by `sudo nano /etc/ssh/sshd_config` and add `port 2200`

- for the changes to take effect type `sudo service ssh force-reload`

- You can only connect via port 22200 by :

`ssh grader@35.177.179.71 -p 2200 -i ~/.ssh/linuxp6`


## Configure UFW 

- Make sure to Change the open port for SSH from 22 to 2200 on the instance side. [read more](https://stackoverflow.com/questions/47342988/aws-ssh-port-timeout-after-changing-port-number)

- block all incoming connections on all ports by `sudo ufw default deny incoming`

- Allow outgoing connection on all ports by `sudo ufw default allow outgoing`

- Allow incoming connection for SSH on port 2200 by `sudo ufw allow 2200/tcp`

- Allow incoming connections for HTTP on port by `sudo ufw allow www`

- Allow incoming connection for NTP on port 123 by `sudo ufw allow ntp`

- Enable the firewall by `sudo ufw enable`

## serving the Item Catalog APP 

- Installing Apache by `sudo apt-get install apache2`

- Installing mod_wsgi `sudo apt-get install libapache2-mod-wsgi`

- Installing PostgreSQL `Installing PostgreSQL` 

- Create a PostgreSQL role named catalog and a database named catalogdb by 

```
sudo -u postgres -i
postgres:~$ createuser catalog
postgres:~$ createdb catalogdb
postgres:~$ psql catalog
postgres=# ALTER DATABASE catalog OWNER TO catalog;
postgres=# ALTER USER catalog WITH PASSWORD 'catalog';
postgres=# \q
postgres:~$ exit

```

- Install requirements by 

```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
```

- Install Git by 'sudo apt-get install git

- Clone the repository by 
```
cd /var/www
sudo mkdir ItemCatalogProject
sudo chown -R grader:grader ItemCatalogProject
cd ItemCatalogProject
sudo git clone https://github.com/EsraaQandel/Item-Catalog-Project.git ItemCatalogProject
```

- edits to the cloned app 

1- change `postgresql://postgres:postgres@localhost/sportsapp` to `postgresql://catalog:catalog@localhost/catalogdb` in project.py
by `sudo nano /var/www/ItemCatalogProject/ItemCatalogProject/project.py` then add the full path to client_secrets.json file  to `/var/www/ItemCatalogProject/ItemCatalogProject/client_secrets.json`

2- change `postgresql://postgres:postgres@localhost/sportsapp` to `postgresql://catalog:catalog@localhost/catalogdb` in database_setup.py
by `sudo nano /var/www/ItemCatalogProject/ItemCatalogProject/database_setup.py`

3- change `postgresql://postgres:postgres@localhost/sportsapp` to `postgresql://catalog:catalog@localhost/catalogdb` in filldb.py
by `sudo nano /var/www/ItemCatalogProject/ItemCatalogProject/filldb.py`

4- Configure Google OAuth by adding the public IP address of your server to the “Authorized javascript origins”

5- In the `project.py` file, there was an extra space the drove me crazy for two days and ruined the home page make sure to remove it from  the project.py here

 ```  
  return render_template(
        'categories.html ',
        categories=categories,
        LatestItems=LatestItems,
        loggedIn=loggedIn)
```

- Create the web application WSGI file by `sudo nano /var/www/ItemCatalogProject/app.wsgi`

- Add the following lines to the file, and save 

```
#!/usr/bin/python
import sys 
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ItemCatalogProject/")
from project import app as application
application.secret_key = 'super_secret_key'

```

- Update the Apache configuration file by `sudo nano /etc/apache2/sites-enabled/000-default.conf`
and then add the following line right after the `<VirtualHost *:80>` element and save:

```
    ServerName 35.177.179.71
	ServerAdmin esraamedhatesamir@gmail.com
	WSGIScriptAlias / /var/www/ItemCatalogProject/app.wsgi
	<Directory /var/www/ItemCatalogProject/ItemCatalogProject>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/ItemCatalogProject/ItemCatalogProject/static
	<Directory /var/www/ItemCatalogProject/ItemCatalogProject/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
```

- Restart Apache by `sudo apache2ctl restart` 


# Refrences:

-[Hosting your Python Application using Flask and Amazon Web Services (AWS)](http://amunategui.github.io/idea-to-pitch/)
- **Special Thanks to my frined *[Basma](https://github.com/basmaashouur)* for a very helpful README**
