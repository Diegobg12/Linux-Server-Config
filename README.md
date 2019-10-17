# Linux-Server-Config
Udacity final project for [Full Stack Web Developer
](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd0044)


Website on:

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
