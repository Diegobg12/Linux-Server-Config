# Linux-Server-Config
Udacity final project for [Full Stack Web Developer
](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd0044)


Website on: http://35.165.20.33/

## 1. CREATE A SERVER

+ Create  AWS Lightsail and singn
+ Create a new instance by chosing `Linux/Unix` - `OS only` - `Ubuntu 16.04LTS`
+ Choose the cheapest plan and create the instance.


## 2. GET YOUR SSH KEYS

+ Go to your AWS account and select `SSH keys`
+ Create .ssh folder in your terminal by `mkdir .ssh`
+ Download the default file, it woul be name like this `LightsailDefaultPrivateKey-*.pem`
+ Save the file in your .ssh folder.
+ Restric file permission by running chmod 600 `LightsailDefaultPrivateKey-*.pem`
+ Change the file name to `lightsail_key.rsa`.

+ To connect run in your terminal `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@54.188.22.32`.


## 3. SECURE THE SERVER

+ Update and upgrade packages running 
 ```sudo apt-get update```
 ```sudo apt-get upgrade```.
+ Configure automatically update packages with unattended-upgrades packages
```
 sudo apt-get install unattended-upgrades
 sudo dpkg-reconfigure --priority=low unattended-upgrades
 
 ```
 
 ## 4. CHANGE THE SERVER PORT
 
 + Run in your terminal `sudo nano /etc/ssh/sshd_config`to edit `etc/ssh/sshd_config`file.
 + Edit the file changing `22` to `2200` in the 5th line.
 + Save by `ctrl+X` and exirt from nano with `Y`.
 + Restart SSH with `sudo service ssh restart`


 ## 4. Set up Uncomplicated FireWall (UFW)
 
+ `sudo ufw status`  # The UFW should be inactive.

+ `sudo ufw default deny incoming`   # Deny any incoming traffic.

+ `sudo ufw default deny outgoing`   Enable outgoing traffic.

+ `sudo ufw allow 2200/tcp`   # Allow incoming tcp packets on port 2200.

+ `sudo ufw allow 80/tcp`   # Allow HTTP traffic in.

+ `sudo ufw allow 123/udp`  # Allow incoming udp packets on port 123.

+ `sudo ufw deny 22` # Deny tcp and udp packets on port 53.

+ `sudo ufw enable` The output should be like this `Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup`

+ `sudo ufw status`  Status should look like this
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```

### Go to Lightsail

+ Account, Networking.
+ Select Edit rules.
+ Change 22 to 2200 deleting the ssh rule.
+ Add 123 for NTP port and save changes.

 ## 5. Create GRADER user:
 
 + Istall finger to manage user by `sudo apt-get install finger`
 + to create `sudo adduser grader`
 + Set password and details for user.
 + See ig user was created correctly `finger grader`
 
 ## 6. Configure TIMEZONE:
 
 + `sudo dpkg-reconfigure tzdata`
 
 
 ## 7. Give GRADER sudo permits:
 
 + Became the root user `sudo -i`
 + Go to sudores `cd /etc/sudoers.d`
 + Create file `nano grader`
 + Paste this content `grader ALL=(ALL) NOPASSWD:ALL`
 + Change permissions on file `chmod 440 grader`.
 + Exit from root user `exit`
 
  
 ## 8. Give GRADER Keys:
 
 ### On .shh in your local machine:
 
 + Create keys `ssh-keygen`
 + Choose a location for the tow files `~/.ssh/authorized_keys`
 + Open the file and copy the content `nano .ssh/authorized_keys`
 
 ### On grader user in VM:
 
 + Create new folder `mkdir .ssh`
 + Create file and paste content `touch .ssh/authorized_keys`
 + set permissions 
  ```
     chmod 700 .ssh 
     chmod 644 .ssh/authorized_keys
  
  ```
  
  + Turn Off password authentication in the file `sudo nano /etc/ssh/sshd_config` change `PasswordAuthentication no`
  + Restart Ssh `sudo service ssh restart`
  + Turn Off password authentication in the file `sudo nano /etc/ssh/sshd_config` change `PasswordAuthentication no`
  + Turn Off password authentication in the file `sudo nano /etc/ssh/sshd_config` change `PasswordAuthentication no`
  ### Login as GRADER user:
  `ssh grader@54.188.22.32 -p 2200 -i ~/.ssh/authorized_keys`
  
  ## 9. Configure firewall to moitos unsuccesful login attemps
  
  + Istall the package `sudo apt-get install fail2ban`
  + Recive alerts toto the admin user`sudo apt-get install sendmail`
  + File to costomize funtionality `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
  + Set configuration `sudo nano /etc/fail2ban/jail.local`
       * destmail: user's email address
       * `bantime = 600`
       * `action = %(action_mwl)s`
       
       
 ## 10. Install packages
 
  + Install apache2 `sudo apt-get install apache2`
  + Istall mos_wsgi `sudo apt-get install libapache2-mod-wsgi python-dev`
  + Enable mod_wsgi: `sudo a2enmod wsgi`
  + `sudo service apache2 start`
  + Install GIT `sudo apt-get install git`
  +Install pip and virtualenv
  ```
   sudo apt-get install python-pip
   sudo pip install virtualenv
   sudo virtualenv venv
   source venv/bin/activate
   sudo chmod -R 777 venv
  
 ```
  + Flask `sudo pip install Flask`
  + sqlalchemy `sudo pip install sqlalchemy`
  + requests `sudo pip install requests`
  + oauth2client `sudo pip install oauth2client`
  + psycopg2 `sudo apt-get install python-psycopg2`

## 11. Clone project

  + Create directory to save project `cd /var/www`  `sudo mkdir catalog`
  + `sudo chown -R grader:grader catalog`
  + `cd catalog`
  + `git clone https://github.com/Diegobg12/Catalog.git`
  
  
## 12. Config app

 + Configure an enable Virtual Host `sudo nano /etc/apache2/sites-available/catalog.conf`.
 + Add the following content: 
 ```
 <VirtualHost *:80>
   ServerName 52.91.21.75
   ServerAlias ec2-52-91-21-75.compute-1.amazonaws.com
   ServerAdmin grader@52.91.21.75
   WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/Catalog/venv/lib/python2.7/site-packages
   WSGIProcessGroup catalog
   WSGIScriptAlias / /var/www/catalog/Catalog/catalog.wsgi
   <Directory /var/www/catalog/catalog/>
       Order allow,deny
       Allow from all
   </Directory>
   Alias /static /var/www/catalog/catalog/static
   <Directory /var/www/catalog/catalog/Catalog/static/>
       Order allow,deny
       Allow from all
   </Directory>
   ErrorLog ${APACHE_LOG_DIR}/error.log
   LogLevel warn
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
 
 ```
 + Enable the virtual Host `sudo a2ensite catalog
 + Create and config the .wsgi file `cd /var/www/catalog/Catalog`
`sudo nano catalog.wsgi`
 + Add the following content
 ```import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/Catalog")

from catalog import app as application
application.secret_key = 'secret'
```
+ Rename project.py to __init__.py
+ Update the absolute path of client_secrets.json in __init__.py
+ Add app.secret_key for the Flask app in __init__.py


## 13. Config PostgreSQL

+ Install Python packages for PostgreSQL `sudo apt-get install libpq-dev python-dev`.
+ Install `sudo apt-get install postgresql postgresql-contrib`
+ Connect with postgres user `sudo -u postgres psql`
+ `CREATE USER catalog WITH PASSWORD 'catalog'`
+ Connect to db `\c catalog`
+ `# REVOKE ALL ON SCHEMA public FROM public;`
+ ` GRANT ALL ON SCHEMA public TO catalog``
+  `\q`to log out and `exit`to return
+ Edit
```
Change engine = create_engine('sqlite:///category.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
in `populateDB.py.py` and `database_setup.py` files.

+ Block remote conenections to PostgreSQL `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`.


## 14. Update OAuth

+ Configurate credentials in Google API to new address.
+ Config `client_secrets.json`to new host address.
+ Populate DataBase running `python populateDB.py`






 
 
 

