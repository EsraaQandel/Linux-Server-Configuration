## Overview 

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

# Server details

URL: [http://35.177.179.71/](http://35.177.179.71/)
SSH port: `2200`

# In this project we used [Amazon Lightsail](https://lightsail.aws.amazon.com/) 

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

- Make sure 

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

