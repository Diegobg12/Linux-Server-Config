# Linux-Server-Config
This document explain how to deploy an app on [AWS](https://lightsail.aws.amazon.com/ls/webapp/home/instances)
Udacity final project for [Full Stack Web Developer
](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd0044)


## Website
 + http://34.212.92.249/
 + http://ec2-34-212-92-249.us-west-2.compute.amazonaws.com

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


 ## 5. Set up Uncomplicated FireWall (UFW)
 
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

 ## 6. Configure TIMEZONE:
 
 + `sudo dpkg-reconfigure tzdata`

 ## 7. Create GRADER user:
 
 + Istall finger to manage user by `sudo apt-get install finger`
 + to create `sudo adduser grader
 + Set password and details for user.
 + See ig user was created correctly `finger grader`
 
 ## 8. Give GRADER sudo permits:
 
 + Edit `sudo visudo`
 + Below `root    ALL=(ALL:ALL) ALL` add `grader  ALL=(ALL:ALL) ALL`
 + Log in as grader `su - grader`.
 + Verify sudo permissions `sudo -l`, the output should be:
 ```
 Matching Defaults entries for grader on ip-18.236.101.92.us-west-2.compute.internal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on ip-18.236.101.92.us-west-2.compute.internal:
    (ALL : ALL) ALL
 ```
 +  
  
 ## 9. Give GRADER Keys:
 
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
  
  ## 10. Configure firewall to monitor unsuccesful login attemps
  
  + Istall the package `sudo apt-get install fail2ban`
  + Recive alerts toto the admin user`sudo apt-get install sendmail`
  + File to costomize funtionality `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
  + Set configuration `sudo nano /etc/fail2ban/jail.local`
       * destmail: user's email address
       * `bantime = 600`
       * `action = %(action_mwl)s`
       
       
 ## 11. Install Apache
 
  + Install apache2 `sudo apt-get install apache2`
  + Istall mos_wsgi `sudo apt-get install libapache2-mod-wsgi-py3`
  + Enable mod_wsgi: `sudo a2enmod wsgi`
  + `sudo service apache2 start`
  + Verify apache2 Ubuntu Default Page.
  
  ## 12. Install, conf and create DB with PostgresSQL
  
  + Install Python packages for PostgreSQL `sudo apt-get install libpq-dev python-dev`.
 + Install `sudo apt-get install postgresql postgresql-contrib`
 + Connect with postgres user `sudo -u postgres psql`
 + `CREATE ROLE catalog WITH LOGIN PASSWORD 'password'`;
 + `ALTER ROLE catalog CREATEDB;`
 + Connect to db `\c catalog`
 + `# REVOKE ALL ON SCHEMA public FROM public;`
 + ` GRANT ALL ON SCHEMA public TO catalog``
 +  `\q`to log out and `exit`to return
 + Edit
 ```
 Change engine = create_engine('sqlite:///category.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
 ```
 in `populateDB.py.py` and `database_setup.py` files before clone.

 + Verify remote conenections are blocked to PostgreSQL `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`.
 + Create a catalog user `sudo adduser catalog`.
 + Give it sudo permissions by adding `catalog  ALL=(ALL:ALL) ALL` in `sudo visudo`.
 + Verify permissions, logg in `su - catalog` and `sudo -l`.
 + While log in in `catalog`in user `createdb catalog`.
 + Switch back to Grader user with `exit`.


  ## 13. Clone project
  
  + Install GIT `sudo apt-get install git`
  + Create directory to save project `cd /var/www`  `sudo mkdir catalog`
  + `sudo chown -R grader:grader catalog`
  + `cd catalog`
  + `git clone https://github.com/Diegobg12/Catalog.git`
  + rename the app.py file by `mv app.py __init()__.py

 
   ## 14. Install virtual env. and dependencies
   
   + While logged in in `grader`
   + Install pip `sudo apt-get install python3-pip`
   + In /var/www/catalog/catalog/ directory.
   + Create the V.E. `sudo virtualenv -p python3 venv3`.
   + `Grader` as the ownership `sudo chown -R grader:grader venv3/`.
   + Activate the V.E. by `. venv3/bin/activate`
   + Flask `sudo pip install Flask`
   + sqlalchemy `sudo pip install sqlalchemy`
   + requests `sudo pip install requests`
   + oauth2client `sudo pip install oauth2client`
   + psycopg2 `sudo apt-get install python-psycopg2`
   + Run the `python3 __init__.py`.
   + Desactivate the V.E. `deactivate`
  
  
  ## 15. Config app

   + `sudo nano /etc/apache2/mods-enabled/wsgi.conf file to use Python 3` and add the following line:
   ```
   #WSGIPythonPath directory|directory-1:directory-2:...
   WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
   ```
   + Configure an enable Virtual Host `sudo nano /etc/apache2/sites-available/catalog.conf`.
   + Add the following content: 
   ```
<VirtualHost *:80>
    ServerName 34.212.92.249
  ServerAlias ec2-34-212-92-249.us-west-2.compute.amazonaws.com
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
          Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

   ```
   + Enable the virtual Host `sudo a2ensite catalog
   + Reload Apache `sudo service apache2 reload`.
   + Create and config the .wsgi file `sudo nano /var/www/catalog/catalog.wsgi`

   + Add the following content
   ```import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/Catalog")

  from catalog import app as application
  application.secret_key = 'secret'
  ```
   + Reload Apache `sudo service apache2 reload`.


 ## 16. Update OAuth

  + Configurate credentials in Google API to new address.
  + Config `client_secrets.json`to new host address.


 ## 17. Launch the app.

  + Activate the V.E. by `. venv3/bin/activate`
  + Run `python database_setup.py`
  + Populate DataBase running `python populateDB.py`
  + Desactivate the V.E. `deactivate`
  + Reload Apache `sudo service apache2 reload`.
  + Open http://ec2-18-236-101-92.us-west-2.compute.amazonaws.com/catalog/Baseball/3/items


 ## 18. Sources. 

  + [Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances)
  + [How to Create a Server on Amazon Lightsail](https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail)
  + [postgresSQL](https://www.postgresql.org/)
  + [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW)
  + [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
  + [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
  + [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
  + [Google Cloud Plateform](https://console.cloud.google.com/home/dashboard?project=cedar-league-253513)
  + [Create a Python 3 virtual environment](https://superuser.com/questions/1039369/create-a-python-3-virtual-environment)
  + [Getting Flask to use Python3 (Apache/mod_wsgi)](https://stackoverflow.com/questions/30642894/getting-flask-to-use-python3-apache-mod-wsgi)
  + [Working with Virtual Environments](https://flask.palletsprojects.com/en/0.12.x/deploying/mod_wsgi/#working-with-virtual-environments)







 
 
 

