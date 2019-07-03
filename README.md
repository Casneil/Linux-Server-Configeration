# Linux Server Configuration


The aim of this project was to set up a Linux server to host web applications.The previously created [Items-Catalog](https://github.com/Casneil/Items-Catalog) web application runs live on Amazon Lightsail secure web server. A baseline installation of a Linux distribution on a virtual machine is taken and prepared to host the web application, installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

The [Items-Catalog](https://github.com/Casneil/Items-Catalog)  is a RESTful web application that displays items, shops and also allow users to login, add, delete and edit items or shops they add. The CRUD (create, read, update and delete) operations and web page templates are handled using Python Flask framework. OAuth 2.0 framework allows users to securely login to the application using Google Sign-in so users can create items that are viewable by everyone but only modifiable by the original creator.

# Setting up the Amazon's Lightsail Instance 

* In your browser go to [Amazon's Lightsail Site](https://lightsail.aws.amazon.com) and register if you dont have an existing account already.

* Once you're able to sign in  to your account and navigate to the button that says create an instance. Click to create a instance and choose Ubuntu asthe instance Image and **OS** only as **Blueprint** Choose **Ubuntu 18.04 LTS**.  
