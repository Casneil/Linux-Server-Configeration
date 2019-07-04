# Linux Server Configuration


The aim of this project was to set up a Linux server to host web applications.The previously created [Items-Catalog](https://github.com/Casneil/Items-Catalog) web application runs live on Amazon Lightsail secure web server. A baseline installation of a Linux distribution on a virtual machine is taken and prepared to host the web application, installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The [Items-Catalog](https://github.com/Casneil/Items-Catalog)  is a RESTful web application that displays items, shops and also allow users to login, add, delete and edit items or shops they add. The CRUD (create, read, update and delete) operations and web page templates are handled using Python Flask framework. OAuth 2.0 framework allows users to securely login to the application using Google Sign-in so users can create items that are viewable by everyone but only modifiable by the original creator.

## 1-Setting up the Amazon's Lightsail Instance 

* In your browser go to [Amazon's Lightsail Site](https://lightsail.aws.amazon.com) and register if you dont have an existing account already.

* Once you're able to sign in  to your account and navigate to the button that says create an instance. Click to create a instance and choose Ubuntu asthe instance Image and **OS** only as **Blueprint** Choose **Ubuntu 18.04 LTS**.  

## 2-Getting the SSH Key

* Navigate to the [Account section](https://lightsail.aws.amazon.com/ls/webapp/account/profile) and download the SSH Key to your local maschine then save it under your .ssh directory on Linux ("maybe different for other operating systmes"). 

* Change the private key file permission not to be accessible by others **sudo chmod 600 LightsailDefaultPrivateKey-us-west-2.pem**.

## 3-Connecting to the Server via SSH

* Now log in from your terminal ssh ubuntu@3.121.84.110 -p 22 -i ~/.ssh/LightsailDefaultPrivateKey-us-west-2.pem.

## 4-Install updates

          
*  Updates the list of available packages and versions for upgrade.`sudo apt-get update`   
            
*  Installs newer versions of the packages.`sudo apt-get upgrade` 

## 5-Create user grader
           
           
*  Create new user grader`sudo adduser grader`

*  Give grader the permission to sudo`sudo touch /etc/sudoers.d/grader`                   
           
*  Check if grader file was added in sudoers directory`sudo ls /etc/sudoers.d`
           
*  Edit the file grader to add the sudo access`sudo nano /etc/sudoers.d/grader` 

      grader ALL=(ALL) NOPASSWD:ALL
      Save changes(Ctrl+X, Ctrl+Y, Enter)


## 6-Generate SSH key for grader

*  On the local machine terminal, generate key pair using `ssh-keygen`, providing passphrase
*  Name it grader_keys.pub
*  Save the private key in `~/.ssh` on local machine
*  On the server instance:

           su - grader
           mkdir .ssh
           touch .ssh/authorized_keys
           nano .ssh/authorized_keys

           Copy the public key content from local machine to this file and save, change the access level
           chmod 700 .ssh
           chmod 644 .ssh/authorized_keys


## 7-Disabling SSH port 22

*  Configure the Lightsail firewall in AWS console to allow port 2200 ssh connections
*  Make changes in the config file in the server from port 22 to 2200 `sudo nano /etc/ssh/sshd_config
*  Restart SSH `sudo service ssh restart
*  Configure the Lightsail firewall in AWS console to allow incoming conections into ports HTTP (port 80), and NTP
   (port 123)
*  Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80),
   and NTP (port 123)

           sudo ufw allow 2200/tcp
           sudo ufw allow www
           sudo ufw allow 123/udp
           sudo ufw deny 22
           sudo ufw default deny incoming
           sudo ufw default allow outgoing
           sudo ufw enable
           sudo ufw status

*  SSH into grader using the key on port 2200 `ssh grader@52.26.30.44 -p 2200 -i ~/.ssh/grader_keys`


## 8-Deploy the project

#### Change the time zone 
*  Check current timezone setting `sudo cat /etc/timezone
*  Configure the local timezone to UTC `sudo dpkg-reconfigure tzdata

#### Install and configure Apache to serve a Python mod_wsgi application
*  Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi
*  Install Apache `sudo apt-get install apache2
*  Your **Instance ip address** should now show the Apache2 default page

#### Install and configure Postgresql

          sudo apt-get install libpq-dev python3-dev
          sudo apt-get install postgresql postgresql-contrib
          sudo su - postgres
          psql
          
*  Then: 

          CREATE USER catalog WITH PASSWORD 'password';
          CREATE DATABASE catalog WITH OWNER catalog;
          \c catalog
          REVOKE ALL ON SCHEMA public FROM public;
          GRANT ALL ON SCHEMA public TO catalog;
          \q
          exit

## 9-Cloning the Items-Catalog from Github [this:](https://github.com/Casneil/Items-Catalog) 

*  Create a directory for the Items-Catalog like so:

          cd /var/www/
          sudo mkdir catalog
          sudo chown grader:grader catalog
          git clone (your_repo_url) catalog
          cd catalog
          sudo nano catalog.wsgi
          
*  Add this to the `catalog.wsgi` file:          

          import sys
          sys.stdout = sys.stderr
          activate_this = '/var/www/catalog/env/bin/activate_this.py'
          with open(activate_this) as file_:
              exec(file_.read(), dict(__file__=activate_this))
          sys.path.insert(0,"/var/www/catalog")

          from app import app as application
          
 ## 10-Installing the virtual environment and application imports 
 
          sudo apt-get install python3-pip
          sudo -H pip3 install virtualenv
          virtualenv env
          source env/bin/activate
          
  #### Imports 
  
          pip3 install flask packaging oauth2client redis passlib flask-httpauth
          pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests  
          
          
#### Enable the catalog app       

          sudo nano /etc/apache2/sites-available/catalog.conf

*  then add this to the file:

          <VirtualHost *:80>
              ServerName 52.26.30.44
              ServerAdmin sarithakamath24@gmail.com
              WSGIScriptAlias / /var/www/catalog.wsgi
              <Directory /var/www/catalog/>
                  Order allow,deny
                  Allow from all
              </Directory>
              Alias /static /var/www/catalog/static
              <Directory /var/www/catalog/static/>
                  Order allow,deny
                  Allow from all
              </Directory>
              ErrorLog ${APACHE_LOG_DIR}/error.log
              LogLevel warn
              CustomLog ${APACHE_LOG_DIR}/access.log combined
           </VirtualHost>

*  Save file then run this:

           sudo a2dessite 000-default.conf
           sudo a2ensite catalog.conf
           
           
  ## 11- Reload & Restart Apache Server
  
          sudo service apache2 reload
          sudo service apache2 restart
          
          
## Resources

[Flask msgi_mod](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/) ./
[Apache Server Configuration](https://httpd.apache.org/docs/current/configuring.html)
[Ask Ubuntu](https://askubuntu.com/)
[Udacity Knowledge](https://knowledge.udacity.com/)
[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
[Postgres](https://www.postgresql.org/docs/current/sql-createuser.html)


