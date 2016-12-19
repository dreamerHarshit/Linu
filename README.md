# Linux Server Configuration

Udacity Full Stack Nanodegree project 5

---------------------------------------

## Objective

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host a web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.


### Server Details

Server IP address: 52.27.62.61

SSH port: 2200

Application URL: http://52.27.62.61


### Software Installed

* Apache2
* PostgreSQL
* bleach
* flask-seasurf
* git
* github-flask
* httplib2
* libapache2-mod-wsgi
* oauth2client
* python-flask
* python-pip
* python-psycopg2
* python-sqlalchemy
* requests

## Configuration

#### Update all currently installed packages

```sh
sudo apt-get update
sudo apt-get upgrade -y
```


#### Configure [Automatic Security Updates][AutomaticSecurityUpdates]

```sh
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

#### Create a new user named `grader`

```sh
adduser grader
```

#### Give the user `grader` permission to sudo

```sh
echo "grader ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/grader
```

#### Set up SSH Authentication

Generate SSH key pairs, then copy the contents of the generated .pub file to the clipboard
```sh
# RUN ON LOCAL MACHINE
ssh-keygen -t rsa -b 2048 -C "Just some comment"
```

Configure public key on server.
   As the `grader` user paste `.pub` file contents in to `.ssh/authorized_key` file
```sh
# RUN ON SERVER
su grader
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
```

Set correct permissions
```sh
chmod 700 ~/.ssh
chmod 644 ~/.ssh/authorized_keys
```

#### Change the SSH port from 22 to 2200

Open SSH config file
```sh
vim /etc/ssh/sshd_config
```

Change `Port 22` to `Port 2200`


#### Remote login of the root user has been disabled

Open SSH config file

```sh
vim /etc/ssh/sshd_config
```

Ensure `PermitRootLogin` has a value  no`


#### Enforce SSH Authentication (i.e prevent password login)

Open SSH config file
```sh
vim /etc/ssh/sshd_config
```

Ensure `PasswordAuthentication` has a value  no`


#### Restart SSH service

```sh
sudo service ssh restart
```

#### Configure the Uncomplicated Firewall (UFW)

Block all incoming requests
```sh
sudo ufw default deny incoming
```

Allow all outgoing requests
```sh
sudo ufw default allow outgoing
```

Allowing incoming connections for SSH (port 2200)
```sh
sudo ufw allow 2200/tcp
```

Allowing incoming connections for HTTP (port 80)
```sh
sudo ufw allow www
```

Allowing incoming connections for NTP (port 123)
```sh
sudo ufw allow ntp
```

Enable `ufw`
```sh
sudo ufw enable
```

#### Configure the local timezone to UTC

Reconfiguring the tzdata package
```sh
sudo dpkg-reconfigure tzdata
# select `None of the above` then `UTC`
```

#### Install and configure Apache to serve a Python mod_wsgi application

Install required packages
```sh
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```

#### Install and configure PostgreSQL

```sh
sudo apt-get install postgresql
```

#### Create a new user named catalog that has limited permissions to your catalog application database

Change to postgres user
```sh
sudo -i -u postgres
```

Create new dastbase user `catalog`
```sh
postgres@server:~$ createuser --interactive -P
Enter name of role to add: catalog
Enter password for new role:
Enter it again:
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) n
Shall the new role be allowed to create more new roles? (y/n) n
```

Create `catalog` Database
```sh
postgres:~$ psql
CREATE DATABASE catalog;
\q
```

logout of postgres user
```sh
exit
```

#### Install git, clone and setup Catalog App project

Install git
```sh
sudo apt-get install git
```

clone repo

Protect `.git` directory
```sh
sudo chmod 700 /var/www/catalog/catalog/.git
```

Install application dependences
```sh
sudo apt-get -qqy install python-psycopg2
sudo apt-get -qqy install python-flask
sudo apt-get -qqy install python-sqlalchemy
sudo apt-get -qqy install python-pip
sudo pip install bleach
sudo pip install flask-seasurf
sudo pip install github-flask
sudo pip install httplib2
sudo pip install oauth2client
sudo pip install requests
```

Create a wsgi file entry point to work with mod_wsgi
```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/catalog")

from catalog import app as application
```

Update last line of `/etc/apache2/sites-enabled/000-default.conf` to handle requests using the WSGI module, add the following line right before the closing </VirtualHost> line:
```sh
WSGIScriptAlias / /var/www/catalog/catalog/myapp.wsgi
```

Update Database connection string in `database_setup` to the following:
```sh
postgresql://catalog:password@localhost/catalog
```

Ensure oauth tokens are correct

Restart Apache
```sh
sudo service apache2 restart
```

[AutomaticSecurityUpdates]: https://help.ubuntu.com/community/AutomaticSecurityUpdates
