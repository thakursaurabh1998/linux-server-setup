# Linux Server Configuration with Google Cloud Compute Engine

This is the third project of the Full Stack Nano Degree II by Udacity.

Access the live WebApp [Science Digest](http://35.185.137.139)

---

#### IP-Address: 35.185.137.139
#### URL: [http://35.185.137.139](http://35.185.137.139)

---


### 1. Creating an instance in google compute engine
  When creating an instance in google engine tick on HTTP in firewall settings. Rest select all the settings accordingly.
### 2. Creating firewall rule in compute engine
  After creating the instance, open VPC Network settings in google cloud. Then open the firewall rules and create a new firewall rule.
  *  Network - ```default```
  *  Priority - ```65534```
  *  Direction - ```Ingress```
  *  Action on match - ```Allow```
  *  IP Ranges - ```0.0.0.0/0```
  *  Protocols and Ports - ```tcp:2200```

### 3.  Start the instance
  Open the google compute engine by clicking on the ssh button present under the 'connect' column.
### 4. Changing the default SSH port
  1.  Run this command-
  ```
  $ sudo nano /etc/ssh/sshd_config
  ```
  2.  Change the ```Port 22``` line to ```Port 2200```
  3.  Open a new instance with custom port 2200(but don't close this one). You can enter the custom port number 2200 by clicking on the dropdown button in the compute engine page under 'connect' column and selecting 'open in browser window on custom port'.

If it opens then the port is changed successfully.

### 5. Configuring the Uncomplicated Firewall
  1.  We have to only give access to http, ntp and our custom ssh port(i.e.2200)
  2.  Enter the following commands to configure the ufw
  ```
  $ sudo ufw allow www
  $ sudo ufw allow ntp
  $ sudo ufw allow 2200/tcp
  ```
  3.  Now activate the firewall
  ```
  $ sudo ufw enable
  ```

### 6. Creating grader user
```
$ sudo adduser grader
```

Enter the following inputs accordingly.

### 7. Giving sudo access to grader user
  1.  Adding a file to the ```sudoers.d``` directory
  ```
  $ sudo nano /etc/sudoer.d/grader
  ```
  2.  Enter the following line:
  ```
  grader ALL=(ALL) NOPASSWD:ALL
  ```
  3.  Save the file. Sudoers access is granted to the user 'grader'.

### 8. Login as grader and insert the public key
  1.  To login as the grader user enter ```$ sudo su grader```
  1.  Go to home directory: ```$ cd ~```
  1.  Make a new directory: ```$ mkdir .ssh```
  1.  Now on your local system create a ssh key by entering ```$ ssh-keygen``` in bash terminal. Enter the place where you want to save the key into the following prompt.
  1.  Copy the contents of the public key.
  1.  Now back in the linux instance create and paste the contents of the key to the file: ```$ sudo nano .ssh/authorized_keys```

### 9. Open grader user from the local machine
  1.  Enter the command:
  ```
  $ ssh grader@your-ip-address -p 2200 -i /location-to-the-rsa-key
  ```
  2.  Enter the passphrase of the ssh key that you generated.

  3.  _You have got access to the virtual machine locally_

### 12. Installing FLASK and other packages
  
  __Remember to install all the pip dependencies in home directory so they are accesible from HOME and doesn't cause any PATH related problems__

  ```
    $ sudo apt-get install python-pip
    $ sudo -H pip install Flask
    $ sudo -H pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
  ```

### 11. Install and configure Postgresql database
  1.  To install postgresql : 
  ```
    $ sudo apt-get install postgresql postgresql-contrib
  ```
  2.  To install python dependencies of postgresql: 
  ```
    $ sudo apt-get install libpq-dev python-dev
  ```
  3.  Login to postgres user and enter psql:
  ```
    $ sudo su - postgres
    $ psql
  ```
  4.  Configuring psql:
      -  Creating a new user named catalog: <br>
      ```# CREATE USER catalog WITH PASSWORD 'password';```
      -  Create a new DB named catalog: <br>
      ```# CREATE DATABASE catalog WITH OWNER catalog;```
      -  Connect to the database catalog: <br>
      ```# \c catalog```
      -  Revoke all rights: <br>
      ```# REVOKE ALL ON SCHEMA public FROM public;```
      -  Lock down the permissions only to user catalog: <br>
      ```# GRANT ALL ON SCHEMA public TO catalog;```
      -  Log out from PostgreSQL: <br>
      ```# \q```
      -  Then return to the grader user: <br>
      ```$ exit``` <br>
      
      ___Inside the Flask application, the database connection is now performed with:___ <br>
      ```engine = create_engine('postgresql://catalog:yourpassword@localhost/catalog')```
### 12. Install and configure Apache2 and mod-wsgi
  1. Installation
  ```
    $ sudo apt-get install apache2 libapache2-mod-wsgi
    $ sudo a2enmod wsgi
  ```
  2. Configuration
      1. Clone the catalog app from github
          - Make a directory named catalog in ```/var/www```:
          
          ```
          $ sudo mkdir /var/www/catalog
          ```
          - Clone the ___Science Digest___ project in this catalog directory:
	        
          ```
	        $ sudo git clone https://github.com/thakursaurabh1998/ScienceDigest.git /var/www/catalog
	      ```
	        
          - Making .wsgi file for serving on mod_wsgi
	        
          ```
	        $ sudo nano catalog.wsgi
          ```
          
          ___CONTENT OF THE FILE___
          ```
          import sys
          import logging
          logging.basicConfig(stream=sys.stderr)
          sys.path.insert(0, "/var/www/catalog/")
          
          from views import app as application
          ```
      2. Default virtual file
          - Open the default virtual file:

          ```
          $ sudo nano /etc/apache2/sites-available/000-default.conf
          ```
          - Replace the content of the file with the following content
          ```
          <VirtualHost *:80>
            ServerName 35.185.137.139
            ServerAdmin thakursaurabh1998@gmail.com
            WSGIProcessGroup views
            WSGIScriptAlias / /var/www/catalog/views.wsgi
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
		  ```	  
	  3. Restarting Apache
	    ```
	    sudo service apache2 restart
	    ```

---

### Server is succesfully running