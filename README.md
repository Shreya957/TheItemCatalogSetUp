## Linux Server Configuration for The Item Catalog Application
This repository includes the configuration setup for The Item Catalog Application


## 1. Amazon LightSail Ubuntu Instance Creation

1. Lon In / Sign Up  to AWS
![Screeshot](https://github.com/Shreya957/TheItemCatalogSetUp/blob/master/images/AWS_SignIn.PNG)

2.Selecting the type of OS, in our case we have used Ubuntu 16.04(OS Only)
![Screeshot](https://github.com/Shreya957/TheItemCatalogSetUp/blob/master/images/select_ubuntu.PNG)

3. Download the SSH Key
![Screeshot](https://github.com/Shreya957/TheItemCatalogSetUp/blob/master/images/key_download_stepone.PNG)

4. Download the default key after clicking on the above.

5. select the monthly plan
![Screeshot](https://github.com/Shreya957/TheItemCatalogSetUp/blob/master/images/monthlyplan.PNG)

6. Name your Instance
![Screeshot](https://github.com/Shreya957/TheItemCatalogSetUp/blob/master/images/nameInstance.PNG)

7. Finally click on the create instance at the botton of the page.

8. Connect using sshconnect 
This page also displays your public IP which will be required in configurations
![Screeshot](https://github.com/Shreya957/TheItemCatalogSetUp/blob/master/images/sshconnect.PNG)



## 2. Server update
post SSH connect , run below command to update and upgrade the server.

```
sudo apt-get update
sudo apt-get upgarde

```

## 3. User Creation

##### run below command to create a user
```
sudo adduser grader

```
## 4. Providing sudo privileges to the grader user

```
usermod -aG sudo grader
```

## 5. Configuring SSH to 2200

A non default port can be enabled in Lightsail via from the Networking Tab of the instance page.
Add a custom application with TCP protocol on port 2200
![Screeshot](https://github.com/Shreya957/TheItemCatalogSetUp/blob/master/images/AddingCustom2200.PNG)

Edit the sdhd config file: sudo nano /etc/ssh/sshd_config Change the following options.
```
# What ports, IPs and protocols we listen for
# Port 22
Port 2200

# Authentication:
LoginGraceTime 120
#PermitRootLogin prohibit-password
PermitRootLogin no
StrictModes yes

# Change to no to disable tunnelled clear text passwords
PasswordAuthentication No
```

Then restart SSH 

```
restart ssh: sudo service restart ssh
```

### 1. Application Installation and setup

#### 1. Apache

Install apache server using below

```
sudo apt-get install apache2 

```
If apache has been set properly, Apache web page will appear at http://13.234.48.213(public IP of the server)


#### 2. WSGI

Install mod wsgi, python setup tools, and python-dev by running below commands

```
sudo apt-get install libapache2-mod-wsgi python-dev

```
enable wsgi if not already enabled by using

```
sudo a2enmod wsgi
```
WSGI can be set up as follows

-  Create the WSGI file  /var/www/TheCatalogApp/TheCatalogApp.wsgi with following details

```
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/TheCatalogApp")

from project import app as application
application.secret_key='super_secret_key'

```
Now set up a virtual host file /etc/apache2/sites-available/TheCatalogApp.conf with following details

```
<virtualHost *:80>
                ServerName publicIP
                ServerAdmin username@servername
                WSGIScriptAlias / /var/www/TheCatalogApp/TheCatalogApp.wsgi
                <Directory /var/www/TheCatalogApp>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/TheCatalogApp/static
                <Directory /var/www/TheCatalogApp/static>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

run the below command to enable the MyCatalogApp site

```
sudo a2dissite 000-default.conf
sudo a2ensite TheCatalogApp.conf
sudo service apache2 reload

```
#### 3. GIT

```
sudo apt-get install git

```
for git account set up run following commands

```
git config --global user.name "Your Name"
git config --global user.email "youremail@domain.com"

```

create a directory TheCatalogApp and clone the git project repositoory in it

Clone the project repository to /var/www/

```
git clone https://github.com/Shreya957/TheCatalogApp.git

```

#### 4. Flask

Run below commands to install 
```
sudo apt-get install python-pip python-flask python-sqlalchemy python-psycopg2
sudo pip install oauth2client requests httplib2

```


#### 5. PostgreSQL

Install postgreSQL by running following commands
```
sudo apt-get install postgresql

``` 


###### Databse creation

1. run psql commands as a postgres user 

```
sudo su - postgres
psql

```
run below commands to create catalog user as the DB owner

```
postgres=# CREATE USER catalog WITH PASSWORD 'password';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
```

connect to the catalog DB using command ``` \c catalog```

```
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
```
 Exit postgres and postgres user
 
 ```
 postgres=# \q
postgres@PublicIP~$ exit

```

Now you have a Databse created with catalog user as its owner.
Now configure this DB in the project.py file.

replace the DB details in project.py , databaseSetup.py and CatalogData.py

``` engine = create_engine('postgresql://catalog:catalog@localhost/catalog') ```

run ``` sudo python databaseSetup.p ``` and ``` sudo python CatalogData.py``` 


#### 6 Oauth config and set up

1. add "xip.io" in "authorized domains" available in the "oauth consent screen" tab
2. Add the Authorised javascript Origin as ``` 	http://13.234.48.213.xip.io ```
3. Add the authorised redirect URl as ```http://13.234.48.213.xip.io/oauth2callback	 and http://13.234.48.213.xip.io/signInCallback```
4. Update the /var/www/TheCatalogApp/client_secrets.json with the lates Json data update after adding the authorosed JS origin
5. Update the oauth value in project.py

```oauth_flow = flow_from_clientsecrets('/var/www/TheCatalogApp/client_secrets.json', scope='')```

### Finally  Restart apache and load the application at http://13.234.48.213.xio.ip

[Instructions to run the Application](https://github.com/Shreya957/TheCatalogApp/blob/master/README.md)
