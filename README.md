# Linux Server Configuration


The aim of this project was to set up a Linux server to host web applications.The previously created [Items-Catalog](https://github.com/Casneil/Items-Catalog) web application runs live on Amazon Lightsail secure web server. A baseline installation of a Linux distribution on a virtual machine is taken and prepared to host the web application, installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The [Items-Catalog](https://github.com/Casneil/Items-Catalog)  is a RESTful web application that displays items, shops and also allow users to login, add, delete and edit items or shops they add. The CRUD (create, read, update and delete) operations and web page templates are handled using Python Flask framework. OAuth 2.0 framework allows users to securely login to the application using Google Sign-in so users can create items that are viewable by everyone but only modifiable by the original creator.

### 1-Setting up the Amazon's Lightsail Instance 

* In your browser go to [Amazon's Lightsail Site](https://lightsail.aws.amazon.com) and register if you dont have an existing account already.

* Once you're able to sign in  to your account and navigate to the button that says create an instance. Click to create a instance and choose Ubuntu asthe instance Image and **OS** only as **Blueprint** Choose **Ubuntu 18.04 LTS**.  

### 2-Getting the SSH Key

* Navigate to the [Account section](https://lightsail.aws.amazon.com/ls/webapp/account/profile) and download the SSH Key to your local maschine then save it under your .ssh directory on Linux ("maybe different for other operating systmes"). 

* Change the private key file permission not to be accessible by others **sudo chmod 600 LightsailDefaultPrivateKey-us-west-2.pem**.

### 3-Connecting to the Server via SSH

* Now log in from your terminal ssh ubuntu@3.121.84.110 -p 22 -i ~/.ssh/LightsailDefaultPrivateKey-us-west-2.pem.

### 4-Install updates

          
*  Updates the list of available packages and versions for upgrade.`sudo apt-get update`   
            
*  Installs newer versions of the packages.`sudo apt-get upgrade` 

### 5-Create user grader
           
           
*  Create new user grader`sudo adduser grader`

*  Give grader the permission to sudo`sudo touch /etc/sudoers.d/grader`                   
           
*  Check if grader file was added in sudoers directory`sudo ls /etc/sudoers.d`
           
*  Edit the file grader to add the sudo access`sudo nano /etc/sudoers.d/grader` 

      grader ALL=(ALL) NOPASSWD:ALL
      Save changes(Ctrl+X, Ctrl+Y, Enter)


### 6-Generate SSH key for grader

*  On the local machine terminal, generate key pair using `ssh-keygen`, providing passphrase
*  Name it grader_keys.pub
*  Save the private key in `~/.ssh` on local machine
*  On the server instance:

          `su - grader
           mkdir .ssh
           touch .ssh/authorized_keys
           nano .ssh/authorized_keys

           `Copy the public key content from local machine to this file and save, change the access level
           chmod 700 .ssh
           chmod 644 .ssh/authorized_keys


### 7-Disabling SSH port 22

*  Configure the Lightsail firewall in AWS console to allow port 2200 ssh connections
*  Make changes in the config file in the server from port 22 to 2200 `sudo nano /etc/ssh/sshd_config
*  Restart SSH `sudo service ssh restart
*  Configure the Lightsail firewall in AWS console to allow incoming conections into ports HTTP (port 80), and NTP
   (port 123)
*  Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80),
   and NTP (port 123)

           `sudo ufw allow 2200/tcp
            sudo ufw allow www
            sudo ufw allow 123/udp
            sudo ufw deny 22
            sudo ufw default deny incoming
            sudo ufw default allow outgoing
            sudo ufw enable
            sudo ufw status

*  SSH into grader using the key on port 2200 `ssh grader@52.26.30.44 -p 2200 -i ~/.ssh/grader_keys`
