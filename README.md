# linux-server-configuration
## Project info

Public IP address: 18.195.60.34 or http://www.18.195.60.34.xip.io/ for using google oauth.
SSH port: 2200.

# Linux Server Configuration Steps

## Launching AWS Lightsail Instance and connecting to it

1. Create new aws lightsail instance 
2. My public IP is: `18.195.60.34`
3. Download private key from account its name is : LightsailDefaultKey-eu-central-1.pem

## Launching VM and configuring ssh

1. Open terminal from path where private key is downloaded and enter command
     `mv LightsailDefaultKey-eu-central-1.pem ~/.ssh/` 
2. Type `chmod 600 ~/.ssh/LightsailDefaultKey-eu-central-1.pem`
3. Connect to the instance using `ssh -i ~/.ssh/LightsailDefaultKey-eu-central-1.pem ubuntu@18.195.60.34`

## Update package source list and update installed packages

1. `sudo apt-get update`
2. `sudo apt-get upgrade`

## Create user grader and give it sudo access

1. Enter command `sudo adduser grader`
2. Choose any password for now (I have choosed grader as a password)
3. Create new file named grader using `sudo touch /etc/sudoers.d/grader
4. Open this file using `sudo nano /etc/sudoers.d/grader` and add this line 
     'grader ALL=(ALL:ALL) ALL`

## Generating key pair for user grader to use key-based authentication

1. Run command `ssh-keygen` on local machine to generate key pair
2. Save the file in `~/.ssh/` and I named it `grader_auth_key.pub`
3. Run `cat ~/.ssh/grader_auth_key.pub` to print the public key
4. Copy the content of `grader_auth_key.pub` file and then go back to grader vm and create new folder .ssh
     using command `mkdir .ssh`
5. Run command `sudo nano ~/.ssh/authorized_keys` and paste the copied content in the generated file the save and close it.
6. Change permissions using `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
7. Run `sudo nano /etc/ssh/sshd_config` command and change port from 22 to 2200 and change `PasswordAuthentication` and `PermitRootLogin` to no
8. Restart ssh using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Enter these commands to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

`sudo ufw status` it should be inactive
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow 2200/tcp`
`sudo ufw allow 80/tcp`
`sudo ufw allow 123/udp`
`sudo ufw deny 22`
`sudo ufw enable`

Go to Amazon Lightsail Instance and go to `networking` tab and change firewall configurations to allow ports `2200/tcp`, `80/tcp` and `123/udp` and deny `22/ssh`


Login with grader on local machine using `ssh -i ~/.ssh/grader_auth_key -p 2200 grader@18.195.60.34`

## Configure the local timezone to UTC

Run command `sudo dpkg-reconfigure tzdata` and change time to UTC

## Install apache2, mod-wsgi

Run commands

`sudo apt-get install apache2`
`sudo apt-get install libapache2-mod-wsgi`
Enable mod_wsgi using command sudo `a2enmod wsgi`


``
