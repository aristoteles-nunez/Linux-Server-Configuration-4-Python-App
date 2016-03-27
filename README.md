# Linux-Server-Configuration-4-Python-App
Configuration steps for production deploy of python3 web application


# Finished Application

The application can be seen in:
[http://52.36.132.142/][1]

# Project Goals

The main goal for this project is deploy a web application made with `Flask` using an `Apache` web server.

* **User access**.
	* Create user with `sudo` privilegies. In this case the user will be `grader`.
	* Use a key based `SSH` authentication with user `grader`.
	* Create user with database limited acces. This user will be `catalog`.
* **Firewall**.
	* The `SSH` service will be attending connections on port: `2200`
	* Only allow incoming connections for: 
		* `SSH`: `2200` 
		* `HTTP`: `80` 
		* `NTP`: `123` 
* **Software**.
	* Ubuntu 14.04
	* [Item-Catalog][2] python3 web application.
	* `Python 3.X` along with `pip3`.
	* `PostgreSQL 9.0`.
	* `Psycopg` adapter.

# Instructions

## 1. Amazon EC2 Instance creation
This `README` does not cover the creation of Virtual machines but you can follow the steps to create one in: [EC2 Launch instance][3]

The following steps are based in a `Ubuntu 14.04` Virtual Machine and you must have the `SSH` key to access it.

The SSH access key usually ends with `.pem` or `.rsa`, in this case we will call it `root_key.rsa` and for convenience store it inside `~/.ssh/` folder with `600` file permissions.

## 2. Create ssh pair key
We need a ssh pair key to use in the next steps for `grader` user, so we need to generate one pair with the following instructions: 

* On your local machine (Unix based) execute the command:

```
ssh-keygen
```

* It will prompt for a path to store the key, in this case we can use: 

```
~/.ssh/grader_key.rsa
```

* It generates two files `grader_key.rsa` and `grader_key.rsa.pub`

Note that these files are different to the one you already have to login in your virtual machine.

## 3. Fix hostname error

When a user is executing a command with `sudo` the following message appears:

```
sudo: unable to resolve host ip-10-20-5-22
```

To fix it, as `root`, edit `vi /etc/hosts` and after the line containing `127.0.0.1 localhost` add:

```
127.0.0.1 localhost
10.20.5.22 ip-10-20-5-22
```
Because the second line specifies the ip address used for `eth0` port locally in the Virtual environment

Save it, and now it is fixed.


## 4. Create `grader` user

The `grader` user will be in the `sudoers` group to perform priviligies operations.

* As `root` create user

```
adduser grader
```

* Add `grader` user to the sudoers group

Create file `/etc/sudoers.d/grader` with the following content:

```
grader ALL=(ALL) NOPASSWD:ALL
```

* Now you can login as user `grader` with:

```
su grader
```

## 5. Install public key to login as `grader` user


* As `grader` user create file `authorized_keys`

```
mkdir /home/grader/.ssh/
touch /home/grader/.ssh/authorized_keys
```

* On `authorized_keys` file, paste the content of the `grader_key.rsa.pub` key created before (view section "How to create ssh pair key"), and change the permissions with:

```
chmod 700 /home/grader/.ssh/
chmod 644 /home/grader/.ssh/authorized_keys
```

* Force key based authentication editing the file 

```
sudo vi /etc/ssh/sshd_config
``` 

* Set the option `PasswordAuthentication` to no

```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```

* Change the `SSH` port to 2200

```
# What ports, IPs and protocols we listen for
Port 2200
```

* Restart the `SSH` service to apply changes

```
sudo service ssh restart
```

* Now login directly with `grader` user using:

```
ssh -i ~/.ssh/grader_key.rsa  grader@52.36.132.142 -p 2200
```
From here we allways be using `grader` user to login.

## 6. Update software

```
sudo apt-get update
sudo apt-get upgrade
```

* Restart to apply changes

```
sudo shutdown -r now
```

## 7. Install software 

### Basic software

```
sudo apt-get install finger
sudo apt-get install build-essential
sudo apt-get install python3-pip python-dev
```
### Install Apache HTTP

```
sudo apt-get install apache2
```

### Install application handler `mod_wsgi` for python3

```
sudo apt-get install apache2-dev
sudo apt-get install libapache2-mod-wsgi-py3
sudo service apache2 restart
```

### Install Git 

```
sudo apt-get install git
```


### Install PostgreSQL

* Create the file:

```
sudo vi /etc/apt/sources.list.d/pgdg.list
```

* Add the following line:

```
deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main
```

* Execute:

```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
  sudo apt-key add -
sudo apt-get update
```

* Install PostgreSQL 9.4 and Psycopg2 plugin

```
sudo apt-get install postgresql-9.4
sudo pip3 install psycopg2
```

## 8. Configure local timezone to UTC

```
sudo timedatectl set-timezone UTC
```

## 9. Configure Firewall with Uncomplicated Firewall (`ufw`)

*  Configurate incoming requests

```
sudo ufw default deny incoming
```

* Configurate outgoing requests

```
sudo ufw default allow outgoing
```

* At this point we can see that the `ufw` is inactive with

```
sudo ufw status
```

* Open port for SSH (2200)

```
sudo ufw allow 2200/tcp
```

* Open port for HTTP (80)

```
sudo ufw allow www
```

* Open port for NTP (123)

```
sudo ufw allow 123/tcp
```

* Enable `ufw` with:

```
sudo ufw enable
```

* Now we can see that the `ufw` status is active with

```
grader@ip-10-20-5-22:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/tcp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/tcp (v6)               ALLOW       Anywhere (v6)
```

## 10. Create user with limited permissions for Database

* Create user `catalog`

```
sudo adduser catalog
```

* Create role `catalog` on database

```
sudo -u postgres createuser -dRSP catalog
```

It will prompt for a password,  take note of the one used to update app in future steps

* Create database `item_catalog`

```
sudo -u catalog createdb item_catalog
```

## 11. Install project Item-Catalog

### Create folder for project

```
sudo mkdir /project/
sudo chown catalog:catalog -R /project/
```


### Configure Apache to handle requests using de WSGI module

```
sudo vi /etc/apache2/sites-enabled/000-default.conf
```

Add the following lines before the `</VirtualHost>` tag

```
        WSGIScriptAlias / /project/myapp.wsgi
        <Directory "/project/">
                Options Indexes MultiViews FollowSymLinks
                AllowOverride None
                Require all granted
        </Directory>

        Alias "/static/" "/project/app/static/"
        <Directory "/project/app/static/">
        		  Options -Indexes
                Order allow,deny
		         Allow from all
        </Directory>
</VirtualHost>
```

* Restart Apache

```
sudo apache2ctl restart
```

### Clone project

* Login as user `catalog`

```
su catalog
```


* Inside `project` folder clone project 4 from GitHub

```
git clone https://github.com/aristoteles-nunez/Item-Catalog.git
```

### Install requirements

* Inside `Item-catalog` folder, and as `grader` user, Install requirements

```
sudo pip3 install -r requirements.txt
```

* Inside `Item-catalog` folder, and as `catalog` user populate with sample data

```
python3 insert_sample_data.py
```
### Make application changes
Some changes are required to make the application works.

* Create `__init__.py` file to identify the app as a module

```
touch /project/Item-Catalog/__init__.py
```

* Inside `project` folder, create the file `myapp.wsgi` with the following content

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/project/")
sys.path.insert(0,"/project/app/")

from app import app as application
application.secret_key = 'super_secret_key_for_catalog_item'
```

* Rename `Item-catalog` folder as `app`

```
mv Item-Catalog/ app
```

* Modify files `app/__init__.py` and `app/models.py` with the right postgresql role name and password.

```
postgresql+psycopg2://catalog:<password>g@localhost/item_catalog
```

* Modify files `app/app.py` with the full system path for `client_secret.json`, and the full path for static folder.

```
client_secret.json => /project/app/client_secret.json
static/ => /project/app/static
```

* Modify files `client_secret.json` with the correct content (updating ip in allowed origins).


* As `grader` user add `catalog` user  user to `www-data` group and change folder permissions to `/project`

```
sudo usermod -a -G  www-data catalog
sudo chgrp -R www-data /project/
sudo chmod -R 775 /project/
```

Restart Apache 2

Go to [http://52.36.132.142/][1] and enjoy it :) 



[1]: http://52.36.132.142/
[2]: https://github.com/aristoteles-nunez/Item-Catalog/
[3]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance_linux