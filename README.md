
# Linux-Server-Configuration-Project

This project is 5th project of Udacity's Full Stack Web Developer Nanodgree

this README walks through how to prepare a Linux server to host a web application. securing the server from number of attack factors, installing an configuring the database server, and deploy the  item catalog web application.


- ip : 35.159.41.242
- port :2200
- url : [http://35.159.41.242.xip.io](http://35.159.41.242.xip.io).

* [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS, Linux 
* [Amazon Lightsail](https://lightsail.aws.amazon.com). server 
* [PostgreSQL](https://www.postgresql.org/) database server.
* item catalog project.

##  Get server
#### 1- start new ubuntu linux server instance on Amazon Lightsail
	-  login into [Amazon Lightsail](https://lightsail.aws.amazon.com).
	- click on `create instance`.
	- choose `Linux/unix` platform, `OS Only`, and `Ubuntu 16.04 LST`.
	- choose an instance plan.
	- rename your instance or keep it as the default one.
	- click on `Create`
	- wait the instance to start up.
Reference:
 [Get started on Lightsail]( https://classroom.udacity.com/nanodegrees/nd004-connect/parts/226fb92a-d5dc-4d10-add0-c1dabff6ee69/modules/56cf3482-b006-455c-8acd-26b37b6458d2/lessons/046c35ef-5bd2-4b56-83ba-a8143876165e/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)
 
 #### 2- SSH into the server 
- from `account` - `ssh keys` download the default private key.
- move the downloaded file into the local folder `~/.ssh` and rename it `hatoon_key.rsa`
- in your terminal  change the file permission by this command:`chmod 600 ~/.ssh/hatoon_key.rsa`
- to connect to the instance via the terminal type: `ssh -i ~/.ssh/hatoon_key ubuntu@35.159.41.242`, 35.159.41.242 is the instance public IP.
	 
## Secure server
#### 3- Update all currently installed packages.
`sudo apt-get update
sudo apt-get upgrade`
#### 4- Change the SSH port from 22 to 2200
- edit sshd_config file using `sudo nano /etc/ssh/sshd_config`
- change the port number from 22 to 2200.
- save the file using `ctrl x then click y`
- restart SSH using: `sudo service ssh restart`.
#### 5- Configure the Uncomplicated Firewall (UFW) 
Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

- check status `sudo ufw status` it should be inactive.
- deny all incoming traffic `sudo ufw default deny incoming`
- Enable outgoing traffic `sudo ufw default allow outgoing`
- Allow incoming tcp packets on port 2200
`sudo ufw allow 2200/tcp` 
- Allow incoming udp packets on port 123 `sudo ufw allow 123/udp`
- Allow HTTP traffic in : `sudo ufw allow www`
- Deny tcp and udp packets on port 22 `sudo ufw deny 22`
-  Turn ufw on via: `sudo ufw enable`
- check the status 



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
- to close the connection: `exit`.
- to change the instance settings too click on the `manage` option of the Lightsail instance.
![image](https://user-images.githubusercontent.com/19895545/50542917-7594b700-0bd9-11e9-9c1c-c13aef9d6670.png)
- click on `Networking` tab and change the fire wall settings like before in the server, allow SSH (port 2200), HTTP (port 80), and NTP (port 123).
![image](https://user-images.githubusercontent.com/19895545/50542928-a70d8280-0bd9-11e9-8a58-0027eb133056.png)
- from the local terminal now you can connect to the server via : `ssh -i ~/.ssh/hatoon_key -p 2200 grader@35.159.41.242`.

References:
 Udacity's Full Stack Web Developer Nanodegree Program lessons.
 
##  Give `grader` access.
#### 6- Create a new user account named `grader`
-  while logged in as ubuntu via : `sudo adduser grader`.
- enter password and fill the information you can skip them by pressing enter, the only important thing is the password.
#### 7- Give `grader` the permission to `sudo`.
- edit the sudoers file with `sudo visudo`.
- look for the line that have 
``
root    ALL=(ALL:ALL) ALL
``
- copy and paste it with the name of the new user `grader` the line should look like this 
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```

- save the file using `ctrl x then click y`.
- to verify `grader` sudo permissions. login as grader: `su - grader` , type password, then type `sudo -l`, type password again the output should be like this.
```
Matching Defaults entries for grader on
    ip-172-26-7-180.eu-central-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User grader may run the following commands on
        ip-172-26-7-180.eu-central-1.compute.internal:
    (ALL : ALL) ALL
```
#### 8- Create an SSH key pair for `grader` using the `ssh-keygen` tool.
- On the local machine:
	- Run `ssh-keygen`.
	- enter the file name to save the key  i saved mine in the home directory. Two files will be generated ( `~/.ssh/hatoon_key` and `~/.ssh/hatoon_key.pub`)
	- run `cat ~/.ssh/hatoon_key.pub` then copy its contents.
	- login as grader in virtual machine.
	- run 
	```
	mkdir .ssh 
	 sudo nano ~/.ssh/authorized_keys
	`chmod 700 .ssh`
	 `chmod 644 .ssh/authorized_keys`
	 ```
	 - set `/etc/ssh/sshd_config` file for `PasswordAuthentication` and set it to no.
	 - Restart SSH: `sudo service ssh restart`
	 - connect now with your key from your local machine `ssh -i ~/.ssh/hatoon_key -p 2200 grader@35.159.41.242`

## Prepare to deploy your project.	 
#### 9- Configure the local timezone to UTC.
- While logged in as `grader`, configure the time zone to UTC: `sudo dpkg-reconfigure tzdata`. 
```
Current default time zone: 'Etc/UTC'
Local time is now:      Thu Dec 27 00:20:03 UTC 2018.
Universal Time is now:  Thu Dec 27 00:20:03 UTC 2018.
```
References
-   [How do I change my timezone to UTC/GMT? (Ask Ubuntu)](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)
#### 10- Install and configure Apache to serve a Python mod_wsgi application.
- while logged in as grader.
- install Apache via: `sudo apt-get install apache2`.
- Enter public IP of the Amazon Lightsail instance into browser. If Apache is working, you should find the `Apache Ubuntu default page`.
- my project is built with python3 so python3 mod_wsgi pakage need to be installed via : `sudo apt-get install libapache2-mod-wsgi-py3`. 
- Enable `mod_wsgi` via: `sudo a2enmod wsgi`.

#### 11- Install and configure PostgreSQL:
- install PostgreSQL: `sudo apt-get install postgresql`, while logged in as `grader`.
- make sure no remote connections are allowed, check `/etc/postgresql/9.5/main/pg_hba.conf` for:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5

```
-   Switch to the `postgres` user: `sudo su - postgres`.
    
-   Open PostgreSQL interactive terminal with `psql`.
    
-   Create the `catalog` user with a password and give them the ability to create databases:
    
    ```
    postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
    postgres=# ALTER ROLE catalog CREATEDB;
    
    ```
  - Exit psql: `\q`, then get back to grader user via `exit`.
  - -   Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
    
-   Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
    
-   look for the line that have 
    
    ```
    root    ALL=(ALL:ALL) ALL
    grader  ALL=(ALL:ALL) ALL
    
    ```
    
-   Below this line, add a new line to give sudo permissions to `catalog` user. the file should look like this:
    ```
    root    ALL=(ALL:ALL) ALL
    grader  ALL=(ALL:ALL) ALL
    catalog  ALL=(ALL:ALL) ALL
    ```
    - save the file using `ctrl x then click y`.
    - to verify `catalog` sudo permissions. login as grader: `su - catalog` , type password, then type `sudo -l`, type password again the output should be like this.
   
```
Matching Defaults entries for catalog on
    ip-172-26-7-180.eu-central-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User catalog may run the following commands on
        ip-172-26-7-180.eu-central-1.compute.internal:
    (ALL : ALL) ALL

```

-  while logged in as `catalog`.
- create database via : `createdb catalog`.
- Run `psql` then `\l` to check that the new database has been created successfully.
```

```
-  Exit psql: `\q`, then get back to grader user via `exit`.

#### 12- Install `git`.
- While logged in as `grader`.
- install `git` via :
 `sudo apt-get install git`.
## Deploy the Item Catalog project.

#### 13- Clone and setup **Item Catalog** project from Github repository created earlier in the Nanodegree program.
- While logged in as `grader`.
-  create `/var/www/catalog/` directory.
- cd to that directory and clone item catalog repository.
- `sudo git clone https://github.com/HatoonMo/ItemCatalogProject.git`
- From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` via: `sudo chown -R grader:grader catalog/`
- Cd to the `/var/www/catalog/catalog` directory.
- -   Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.
    
-   In `__init__.py`, replace line 233:
    
    ```
    # app.run(host='0.0.0.0', port=5000)
    app.run()
    
    ```
   - In  database_setup.py, replace line 58:
   
```
# engine = create_engine('sqlite:///catalogDB.db')
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
### authenticating login through google
-   From  [Google Cloud Plateform](https://console.cloud.google.com/).
-   Click `APIs & services` on left menu then `Credentials`.
-   Create an OAuth Client ID, add  and add  [http://35.159.41.242.xip.io](http://35.159.41.242.xip.io)  as authorized JavaScript origins.
-   Add [http://35.159.41.242.xip.io/oauth2callback](http://35.159.41.242.xip.io/oauth2callback)
- [http://35.159.41.242.xip.io/login](http://35.159.41.242.xip.io/login)
- [http://35.159.41.242.xip.io/gconnect](http://35.159.41.242.xip.io/gconnect)
 as authorized redirect URI.
-   Download the corresponding JSON file, open it then copy the contents and paste them into `/var/www/catalog/catalog/c.json`.
-   Replace the client ID to line 12 of `templates/login.html` file in the project directory.
-  Add the path of `c.json` to line 18 and 67 of `__init__.py`

#### 14 -  Install virtual environment and all the project dependencies
-  While logged in as `grader`.
-  install pip: `sudo apt-get install python3-pip`.
    
-   Install the virtual environment: `sudo apt-get install python-virtualenv`
    
-   Change to the `/var/www/catalog/catalog/` directory.
    
-   Create the virtual environment: `sudo virtualenv -p python3 venv3`.
    
-   Change the ownership to `grader` with: `sudo chown -R grader:grader venv3/`.
    
-   Activate the new environment: `. venv3/bin/activate`.
    
-   Install the following dependencies:
 ```

pip install flask
pip install sqlalchemy
pip install requests
pip install psycopg2
pip install --upgrade oauth2client
```
-   Run `python3 __init__.py` to check everything is okay: 
    ```
    * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
    
    ```
    
-   Deactivate the virtual environment: `deactivate`.
 #### 15- set up virtual host
 - Create `/etc/apache2/sites-available/catalog.conf` and add the following lines to configure the virtual host:
 ```
 <VirtualHost *:80>
    ServerName 35.159.41.242
    ServerAlias 35.159.41.242.xip.io
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
 - Add the following line in `/etc/apache2/mods-enabled/wsgi.conf` file to use Python 3.
  ```      #WSGIPythonPath directory|directory-1:directory-2:...
         WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
 ```
- `sudo a2ensite catalog` to enable virtual host.
- `sudo service apache2 reload` to reload Apache server.
#### 16- Setting up flask application
- Create `/var/www/catalog/catalog.wsgi`file then add these lines 
 ```
activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "---"

 ```
 `sudo service apache2 restart` to restart the Apache server.
 #### 17- set up the database and fill it with data
 - cd to `/var/www/catalog/catalog/` directory. 
 - activate the virtual environment via :`. venv3/bin/activate`.
 - Run `database_setup.py` **first** 
 - Run `seeder.py` **second**
 - deactivate virtual environment. 
 #### 18- start the web application
 - `sudo a2dissite 000-default.conf` to disable the default Apache site.
 - `sudo service apache2 reload`
 - `sudo service apache2 restart`.
 - open in browser [http://35.159.41.242.xip.io](http://35.159.41.242.xip.io).
 
 ## edit after first review 
 - prohibiting log in as `root` remotely.
 - use this command `sudo nano /etc/ssh/sshd_config` to change these lines to no
``    
 PermitRootLogin no 
  `` 
    `` 
 ProhibitRootLogin no
 `` 
 - check everything is up to date 
	 - fixed the problem of some packages not updated and the MOTD (Message Of The Day) stating that there are some packages that need to be installed even after running `sudu apt-get update `and `sudo apt-get upgrade`
	 - running these commands fixed it 
```
sudo apt-get aptitude 
sudo aptitude update
sudo aptitude safe-upgrade
```

 ## Helpful Resources
- [boisalai](https://github.com/boisalai)/[udacity-linux-server-configuration](https://github.com/boisalai/udacity-linux-server-configuration)
- [iliketomatoes/linux_server_configuration](https://github.com/iliketomatoes/linux_server_configuration)
- [rrjoson](https://github.com/rrjoson)/[udacity-linux-server-configuration](https://github.com/rrjoson/udacity-linux-server-configuration)
 
