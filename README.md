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
  sudo nano /etc/ssh/sshd_config
  ```
  2.  Change the ```Port 22``` line to ```Port 2200```
  3.  Open a new instance with custom port 2200(but don't close this one). You can enter the custom port number 2200 by clicking on the dropdown button in the compute engine page under 'connect' column and selecting 'open in browser window on custom port'.

If it opens then the port is changed successfully.

### 5. Configuring the Uncomplicated Firewall
  1.  We have to only give access to http, ntp and our custom ssh port(i.e.2200)
  2.  Enter the following commands to configure the ufw
  ```
  sudo ufw allow www
  sudo ufw allow ntp
  sudo ufw allow 2200/tcp
  ```
  3.  Now activate the firewall
  ```
  sudo ufw enable
  ```

### 6. Creating grader user
```
sudo adduser grader
```

Enter the following inputs accordingly.

### 7. Giving sudo access to grader user
  1.  Adding a file to the ```sudoers.d``` directory
  ```
  sudo nano /etc/sudoer.d/grader
  ```
  2.  Enter the following line:
  ```
  grader ALL=(ALL) NOPASSWD:ALL
  ```
  3.  Save the file. Sudoers access is granted to the user 'grader'.

### 8. Login as grader and insert the public key
  1.  To login as the grader user enter ```sudo su grader```
  1.  Go to home directory: ```cd ~```
  1.  Make a new directory: ```mkdir .ssh```
  1.  Now on your local system create a ssh key by entering ```ssh-keygen``` in bash terminal. Enter the place where you want to save the key into the following prompt.
  1.  Copy the contents of the public key.
  1.  Now back in the linux instance create and paste the contents of the key to the file: ```sudo nano .ssh/authorized_keys```

### 9. Open grader user from the local machine
  1.  Enter the command:
  ```
  ssh grader@your-ip-address -p 2200 -i /location-to-the-rsa-key
  ```
