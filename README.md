# Linux Server Configuration
Taking a baseline Ubuntu Linux virtual machine that is hosted on Amazon EC2
 and preparing it to host a web application.<br/>
 Includes installing updates,
 securing the server from a number of attack vectors, and installing/configuring web and database servers.

## Server Information
 - Public IP Address: 35.166.133.241 
 - SSH port: 2200
 - SSH connection command: 
 ```
 ssh -i ~/.ssh/udacity_key.rsa root@35.166.133.241 -p 2200
 ```
 <br/>where [key_file_path] is the path to the file containing the supplied private key of the grader user.

## Configuration Steps

### Create a new user named *grader* and grant the user sudo permissions
 ```
 # create linux user
 sudo adduser grader

 # give user sudo permissions
 # user will be asked for their password when using sudo
 echo "grader ALL=(ALL) ALL" > /etc/sudoers.d/grader
```

### Enable key based login for the user *grader*
On the local machine, use the command ssh-keygen to create public and private keys for the user *grader*.
```
ssh-keygen -f ~/.ssh/grader
```
Put the public key in the *grader* user's .ssh folder on the server.
```
su grader
mkdir /home/grader/.ssh
nano /home/grader/.ssh/authorized_keys
# copy public key from the local machine and paste into the authorized_keys file on the server, and save
```

### Disable remote login of the *root* user
```
sudo nano /etc/ssh/sshd_config
# set PermitRootLogin to no, and save the file

# restart ssh service
sudo service ssh restart
```

### Update installed packages
 ```
sudo apt-get update
sudo apt-get upgrade
```

### Change SSH port from 22 To 2200
 ```
 sudo nano /etc/ssh/sshd_config
 # change the line 'Port 22' to 'Port 2200', and save the file
 ```

### Configure Uncomplicated Firewall
 ```
# close all incoming ports
sudo ufw default deny incoming
# open all outgoing ports
sudo ufw default allow outgoing
# open ssh port
sudo ufw allow 2200/tcp
# open http port
sudo ufw allow 80/tcp
# open ntp port
sudo ufw allow 123/udp
# turn on firewall
sudo ufw enable
```

### Configure Local timezone to UTC
The server was already set to UTC timezone, but the following command could be used.
 ```
 sudo dpkg-reconfigure tzdata
 # choose 'None of the above' and then select 'UTC'
 ```

### Install the Apache web server and WSGI module
 ```
 sudo apt-get install apache2 libapache2-mod-wsgi
 ```

### Install and configure the PostgreSQL database system
 ```
 sudo apt-get install PostgreSQL
 # check that remote connections are not allowed in PostgreSQL config file
 sudo nano /etc/postgresql/9.3/main/pg_hba.config
 ```

### Create a user named catalog that has limited permissions to the catalog application database
 
 ```
# create a linux user named catalog
sudo adduser catalog

# create a PostgreSQL role named catalog and a database named catalog
sudo -u postgres -i
postgres:~$ creatuser catalog
postgres:~$ createdb catalog
postgres:~$ psql
postgres=# ALTER DATABASE catalog OWNER TO catalog;
postgres=# ALTER USER catalog WITH PASSWORD 'catalog'
postgres=# \q
postgres:~$ exit
```

### Install git and clone the web application project
Clone the web application to the web server's root folder, and ensure that the git
folder is not accessible from the web server.
 ```
sudo apt-get install git
cd /var/www
git clone https://github.com/dreamerHarshit/P3--Catalog-project.git

# ensure git folder is not accessible via web server
echo "RedirectMatch 404 /\.git" > /var/www/.htaccess
 ```

### Install the Python libraries required by the web application
 ```
sudo apt-get install python-pip python-dev python-psycopg2
sudo pip install -r /var/www/P3--Catalog-project/requirements.txt
```
### Configure the web application to connect to the PostgreSQL database instead of a SQLite database
 ```
 sudo nano /var/www/P3--Catalog-project/config.py
 # change the DATABASE_URI setting in the file from 'sqlite:///catalog.db' to 'postgresql://catalog:db_password@localhost/catalog', and save
 ```

### Create schema and populate the catalog database with sample data
 ```
 python /var/www/database_setup.py
 ```

### Configure Apache to serve the web application using WSGI
Create the web application WSGI file.
 ```
sudo nano /var/www/P3--Catalog-project/app.wsgi
```
Add the following lines to the file, and save the file.
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/P3--Catalog-project/")
from catalog import app as application
application.secret_key = 'asecretkey'
```

Update the Apache configuration file to serve the web application with WSGI.
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
Add the following line inside the `<VirtualHost *:80>` element, and save the file.
```
WSGIScriptAlias / /var/www/P3--Catalog-project/app.wsgi
```
Restart Apache.
```
sudo apache2ctl restart
```

### Test the web application
Browse to the public ip address of the server, [http://52.42.38.177](http://52.42.38.177), and 
the web application should start up.
If not, then the command ```sudo tail /var/log/apache2/error.log``` will be useful.


### Fix sudo warning message
The sudo command was displaying a warning message *unable to resolve host [ip-10-20-2-241]*.
So I added the hostname to the loopback address in the /etc/hosts file, so that the line reads: 
```127.0.0.1 localhost ip-10-20-2-241```

### Fix apache2ctl warning message
The apache2ctl command was displaying a warning message: 
*Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message*. 
So I set the ServerName with the following command.
```
sudo echo -e "\nServerName localhost" >> /etc/apache2/apache2.conf
```
