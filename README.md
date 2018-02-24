# Linux Server Configuration (Lightsail Amazon Server)

This is the project for Udacity - Full Stack Web Developer Nanodegree. A Linux virtual machine will be configurated to support a web application. I will secure the server from a number of attack vectors, install and configure a database server, and deploy one of my existing web applications onto it.

## IP address and SSH port
- public IP: 13.59.54.0
- Server Alias Name: ec2-13-59-54-0.us-east-2.compute.amazonaws.com
- ssh port: 2200

## URL to deployed web application
http://ec2-13-59-54-0.us-east-2.compute.amazonaws.com/

## Getting started with Amazon Lightsail
1. Log in to Lightsail from here https://amazonlightsail.com/. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
2. Create a Ubuntu instance (OS only).
3. Choose a plan (choosing the lowest plan with get you free-tier access for a month).
4. Give your instance a hostname.
5. You now have an instance running (It may take a few minutes for your instance to start up).
6. Once your instance has started up, you can log into it with SSH from your browser.

## Instructions for SSH access to the instance
1. Download Private Key from the SSH keys section in the Account section on Amazon Lightsail.
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal. `mv ~/Downloads/Lightsail-key.pem ~/.ssh/`
3. Open your terminal and type in `chmod 400 ~/.ssh/Lightsail-key.pem`
4. In your terminal, type in `ssh -i {.pem file} ubunut@{ip_address}`

## Update all currently installed packages
1. Update available packages: `sudo apt-get update`
2. Upgrade installed packages: `sudo apt-get upgrade`

## Change the SSH port from 22 to 2200
1. Use `sudo vi /etc/ssh/sshd_config` and change the `PORT 22` to `PORT 2200`, save and quit.
2. Restart SSH service: `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)
1. Allow SSH on port 2200: `sudo ufw allow 2200/tcp`
2. Allow HTTP on port 80: `sudo ufw allow 80/tcp`
3. Allow NTP on port 123: `sudo ufw allow 123/udp`
4. Enable firewall: `sudo ufw enable`
5. Check the status: `sudo ufw status`
Note: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used.

## Create a new user grader and give grader sudo
1. Create a new user named grader: `sudo adduser grader`
2. Create a password (We will be disabling this feature). Then, we will be asked for addition information which we can just press enter (optional).
3. Use the usermod command to add the user to the sudo group:
`sudo usermod -aG sudo grader`

## As grader, force Key-Based Authentication
1. As ubuntu, use `sudo su - grader` to switch the user to grader.
2. Generate keys on local machine using `ssh-keygen` ; then save the private key in `/home/bcko/.ssh/id_rsa` on local machine
3. `sudo mkdir /home/grader/.ssh
    sudo chown grader:grader /home/grader/.ssh
    sudo chmod 700 /home/grader/.ssh
    sudo cp /home/ubuntu/.ssh/authorized_key
    sudo chmod 644 /home/grader/.ssh/authorized_keys`
4. Reload SSH using `service ssh restart`
5. Now you must use key pair to login. In local machine, in the directory with the .pem file you can login to the lightsail server:
  `ssh -i {.pem file} grader@{ip_address} -p 2200`

## Disable root (ubuntu user)
1. Change `PermitRootLogin yes` to `PermitRootLogin no` with
  `sudo vi /etc/ssh/sshd_config`
2. Restart ssh service `sudo service ssh restart`

## Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## Install and configure Apache to serve a Python mod_wsgi application
1. To get apache2: `sudo apt-get install apache2`
2. To get mod_wsgi: `sudo apt-get install libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install git, clone and setup your Catalog App project
1. To get git: `sudo apt-get install git`
2. Go into apache2 directory and create a FlaskApp directory
   `cd /var/www`
   `sudo mkdir FlaskApp`
   `cd FlaskApp`
   `git clone https://github.com/junyan59/item-catalog.git`
   `sudo mv ./item-catalog ./FlaskApp`
   `cd FlaskApp`
3. Rename `application.py` to `__init__.py` using `sudo mv application.py __init__.py`, if `__init__.py` not present
4. Edit `database_setup.py` and `example_items.py` to change `engine = create_engine('sqlite:///item-catalog.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`, if not already done.
5. Get Flask and set up virtual env
  `sudo apt-get install python-pip`
   You should also run the command to update pip
  `sudo pip install virtualenv`
6. In the FlaskApp/FlaskApp directory: create the environment in the directory
  `sudo virtualenv venv`
7. Activate the virtual environment:
  `source venv/bin/activate`
8. Install Flask:
  `sudo pip install Flask`
9. The app should work with:
  `sudo python __init__.py`
10. Deactivate the env with:
  `deactivate`

## Set up the config file for apache
1. Create FlaskApp.conf to edit:
   `sudo vi /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host.
```
<VirtualHost *:80>
    ServerName example_items.py
    ServerAdmin stephanieyan59@gmail.com
    WSGIScriptAlias /var/www/FlaskApp/flaskapp.wsgi
    <Directory /var/www/FlaskApp/FlaskApp/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/FlaskApp/FlaskApp/static
    <Directory /var/www/FlaskApp/FlaskApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable virtual host:
  `sudo a2ensite FlaskApp`

## Set up wsgi file
1. Create the .wsgi File under /var/www/FlaskApp:
  `cd /var/www/FlaskApp`
  `sudo vi flaskapp.wsgi`
2. Add the following lines of code to the flaskapp.wsgi file:
```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'super_secret_key'
```
3. Restart Apache: `sudo service apache2 restart`

## Install modules
1. Go inside `source venv/bin/activate`
2. Use pip to install all modules:
  `pip install httplib2` - For httplib2 module
  `pip install requests` - For requests module
  `pip install --upgrade oauth2client` - To use oauth authentication
  `pip install sqlalchemy` - To use the python sqlalchemy
  `sudo apt-get install python-psycopg2` - To use python Postgresql psycopg

## Install and configure PostgreSQL
1. Install PostgreSQL:
  `sudo apt-get install postgresql`
2. Create a user for psql:
  `sudo adduser catalog {Password}`
3. Change user to postgres:`sudo su - postgres`
4. Connect to psql:`psql`
5. Create a new database named catalog and create a new user named catalog in postgreSQL shell:
  `postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER catalog;`
6. Set a password for user catalog:
  `postgres=# ALTER ROLE catalog WITH PASSWORD 'password';`
7. Give user "catalog" permission to "catalog" application database:
  `postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
8. Quit postgreSQL `postgres=# \q`
9. Exit from user "postgres" `exit`

## Set up OAuth
1. Google oauth:
  - https://console.developers.google.com/project
  Go to your application credentials tab and add your public ip and hostname to Authorized JS origins
2. Facebook oauth:
  - https://developers.facebook.com/apps/
  Go to your application and the settings tab
  - Add your public ip to the site URL

## Restart Apache and Run the application
1. Restart Apache `sudo service apache2 restart`
2. Use the URL below to visit web application
http://ec2-13-59-54-0.us-east-2.compute.amazonaws.com/

## References
- [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/docs/getting-started/article/getting-started-with-amazon-lightsail)
- [Set up SSH for your Linux/Unix-based Lightsail instances](https://lightsail.aws.amazon.com/ls/docs/how-to/article/lightsail-how-to-set-up-ssh)
- [Create a Sudo User on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart)
- [mod_wsgi set up](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
