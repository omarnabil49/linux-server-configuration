# linux-server-configuration
## Project info

Public IP address: 18.195.60.34  or http://www.18.195.60.34.xip.io/ for using google oauth  
URL : http://ec2-18-195-60-34.eu-central-1.compute.amazonaws.com/  
SSH port: 2200

# Linux Server Configuration Steps

## Launching AWS Lightsail Instance and connecting to it

1. Create new aws lightsail instance 
2. My public IP is: `18.195.60.34`
3. Download private key from aws lightsail account, its default name is : `LightsailDefaultKey-eu-central-1.pem`

## Launching VM and configuring ssh

1. Open terminal from path where private key is downloaded and enter command  
     `mv LightsailDefaultKey-eu-central-1.pem ~/.ssh/` 
2. Type `chmod 600 ~/.ssh/LightsailDefaultKey-eu-central-1.pem`
3. Connect to the instance using `ssh -i ~/.ssh/LightsailDefaultKey-eu-central-1.pem ubuntu@18.195.60.34`

## Update package source list and update installed packages

1. `sudo apt-get update`  
2. `sudo apt-get upgrade`
3. Run command `sudo apt install unattended-upgrades` to install `unattended-upgrades` package
4. edit `/etc/apt/apt.conf.d/50unattended-upgrades` and adjust the following 
```
Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        "${distro_id}:${distro_codename}-updates";
//      "${distro_id}:${distro_codename}-proposed";
//      "${distro_id}:${distro_codename}-backports";
};
```
5.  Edit `/etc/apt/apt.conf.d/20auto-upgrades` to To enable automatic updates and add the following
```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

6. Run commande `sudo dpkg-reconfigure --priority=low unattended-upgrades` to enable it

I still got the problem so I used this solution

7. Run command `sudo apt-get install aptitude` to install aptitude
8. Run commands 
```
sudo aptitude update
sudo aptitude safe-upgrade
```
9. Restart apache `sudo service apache2 restart`


## Create user grader and give it sudo access

1. Enter command `sudo adduser grader`
2. Choose any password for now (I have choosed grader as a password)
3. Create new file named grader using `sudo touch /etc/sudoers.d/grader`
4. Open this file using `sudo nano /etc/sudoers.d/grader` and add this line   
     `grader ALL=(ALL:ALL) ALL`

## Generating key pair for user grader to use key-based authentication

1. Run command `ssh-keygen` on local machine to generate key pair
2. Save the file in `~/.ssh/` and I named it `grader_auth_key.pub`
3. Run `cat ~/.ssh/grader_auth_key.pub` to print the public key
4. Copy the content of `grader_auth_key.pub` file and then go back to grader vm and create new folder .ssh  
     using command `mkdir .ssh`
5. Run command `sudo nano ~/.ssh/authorized_keys` and paste the copied content in the generated file the save and close it.
6. Change permissions using `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
7. Run `sudo nano /etc/ssh/sshd_config` command and change port from 22 to 2200 and change `PasswordAuthentication` and   `PermitRootLogin` to no
8. Restart ssh using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Enter these commands to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)  

 `sudo ufw status` it should be inactive  
 ```
 sudo ufw default deny incoming    
 sudo ufw default allow outgoing    
 sudo ufw allow 2200/tcp    
 sudo ufw allow 80/tcp    
 sudo ufw allow 123/udp    
 sudo ufw deny 22    
 sudo ufw enable  `
```

Go to Amazon Lightsail Instance and go to `networking` tab and change firewall configurations to allow ports `2200/tcp`, `80/tcp` and `123/udp` and deny `22/ssh`


Login with grader on local machine using `ssh -i ~/.ssh/grader_auth_key -p 2200 grader@18.195.60.34`

## Configure the local timezone to UTC

Run command `sudo dpkg-reconfigure tzdata` and change time to UTC

## Install apache2, mod-wsgi

Run commands

`sudo apt-get install apache2`  
`sudo apt-get install libapache2-mod-wsgi`  

Enable `mod_wsgi` using command sudo `a2enmod wsgi`  

## Install and configure PostgreSQL

1. Run command `sudo apt-get install postgresql`  
2. Run `sudo apt-get install libpq-dev python-dev` to install PostgreSQL Python packages
3. Connect to the database with user postgres using `sudo su - postgres` and `psql` commands
4. Create user `catalog` using `create user catalog with password catalog;` command
5. Create database `itemCatalog` with owner catalog using `create database itemcatalog with owner catalog;`
6. Give all permissions to catalog user  
   Enter `revoke all on schema public from public;` then `grant all on schema public to catalog;` commands
7. Log out and exit 

## Install git and setup project

1. Use `sudo apt-get install git` to install git
2. Navigate to path `/var/www/` folder and create new folder `itemCatalog`
3. Change the owner of the directory to grader using `sudo chown -R grader:grader itemCatalog`
4. Navigate to the created folder and clone the item catalog project from github using  
   `git clone https://github.com/omarnabil49/item-catalog itemCatalog`

## Setup new virtual host

1. Create `itemcatalog.conf` file in path `/etc/apache2/sites-enabled` and edit it using `sudo nano /etc/apache2/sites-enabled/itemcatalog.conf` and add following lines
```
<VirtualHost *:80>
        ServerName 18.195.60.34
        ServerAdmin omar.nabil.49@gmail.com
        WSGIDaemonProcess itemCatalog python-home=/var/www/itemCatalog/itemCata$
        WSGIProcessGroup itemCatalog
        WSGIScriptAlias / /var/www/itemCatalog/myapp.wsgi
        <Directory /var/www/itemCatalog/itemCatalog>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/itemCatalog/itemCatalog/static
        <Directory /var/www/itemCatalog/itemCatalog/static/>
            Order allow,deny
            Allow from all
        </Directory>`
```

then save and exit

2. Enable the virtual host using `sudo a2ensite itemcatalog`
3. Navigate to `itemCatalog` folder using `cd /var/www/itemCatalog/` command
4. Create `.wsgi` file with `sudo nano myapp.wsgi` command and add this content 
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/itemCatalog")

from itemCatalog import app as application
application.secret_key = 'super_secret_key'
```
5. Rename `application.py` file and change it to `__init__.py`  
6. Edit `database_setup`, `lotsofseries.py` and `__init__.py` and change `create_engine('sqlite:///Series.db')` to `create_engine('postgresql://catalog:catalog@localhost/itemcatalog')`  
7. Edit `__init__.py` and change `client_secrets.json` file path form `'client_secrets.json'` to `'/var/www/itemCatalog/itemCatalog/client_secrets.json'`  

## Install virtual environment

1. Navigate to `/var/www/itemCatalog/itemCatalog`
2. Use command `sudo pip install virtualenv` to install the virtual environment
3. Run `sudo virtualenv venv` command to create a new virtual environment
4. Use `venv/bin/activate` command to activate the virtual environment

## Install all dependencies

1. Install pip `sudo apt-get install pip`
2. Install Flask `sudo apt-get install flask`
3. Install oauth2client `sudo apt-get install oauth2client`
4. Install httplib2 `sudo apt-get install httplib2`
5. Install psycopg2 `sudo apt-get install psycopg2`
6. Install sqlalchemy `sudo apt-get install sqlalchemy`
7. Install requests `sudo apt-get install requests`  

Deactivate the virtual environment by using `deactivate` command

## Launching the application

1. Run `database_setup.py` and then `lotsofseries.py` files.
2. Run `sudo service apache2 restart` command to start the application
3. Go to `http://18.195.60.34` to run the application

## Resources

1. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
2. http://flask.pocoo.org/docs/0.12/installation/
3. http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
4. https://help.ubuntu.com/community/UbuntuTime 
5. https://help.ubuntu.com/lts/serverguide/automatic-updates.html
6. https://help.ubuntu.com/community/AutomaticSecurityUpdates
7. https://serverfault.com/questions/262751/update-ubuntu-10-04/262773#262773
